# ROS Recording Workflow - Drone LiDAR Data

This document explains how live 2D LiDAR data from a drone's companion computer is captured and saved as a file using ROS. Each section pairs a short explanation with a diagram to make the process clear for readers new to ROS.

---

## 1. Overview

A LiDAR sensor does not save data on its own — it only transmits a continuous stream of readings. ROS, running on the companion computer, listens to this stream and records it to disk when instructed. The diagram below outlines the complete process, from sensor output to a file that can be replayed later.

```mermaid
flowchart TD
    A["2D LiDAR<br/>Transmits live scan data"] --> B["ROS Driver<br/>Publishes data on topic '/scan'"]
    B --> C["Recording Command<br/>Initiates data capture"]
    C --> D["ROS Writes to Disk<br/>Each scan stored with a timestamp"]
    D --> E["Bag File Created<br/>.bag / .db3 / .mcap"]
    E --> F["Playback & Review<br/>Foxglove Studio / Rviz"]

    classDef live fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef process fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef saved fill:#dcfce7,stroke:#16a34a,color:#14532d
    class A,B live
    class C,D process
    class E,F saved
```

---

## 2. Data Flow Sequence

The sequence diagram below shows the same process viewed as a timeline of interactions. Data remains transient — visible only in memory — until a recording process actively subscribes to the topic and begins writing it to storage.

```mermaid
sequenceDiagram
    participant L as 2D LiDAR
    participant D as ROS Driver
    participant T as Topic "/scan"
    participant R as Recorder
    participant F as Bag File

    L->>D: Raw scan data
    D->>T: Publish LaserScan message
    Note over T: Data exists only in memory<br/>until recording begins
    R->>T: Subscribe to topic
    T->>R: Scan data + timestamp
    R->>F: Write to disk
    Note over F: File grows incrementally,<br/>one scan at a time
    F-->>R: Available for playback
```

---

## 3. Selecting a File Format

The resulting file format depends on the ROS version in use and, for ROS2, the storage plugin selected at recording time.

```mermaid
flowchart TD
    Start(["ROS Version"]) --> Q1{"ROS1 or ROS2?"}
    Q1 -->|"ROS1"| A1["rosbag record /scan"]
    A1 --> R1["Output: single .bag file"]

    Q1 -->|"ROS2"| Q2{"Storage Plugin"}
    Q2 -->|"Default (SQLite3)"| A2["ros2 bag record /scan"]
    A2 --> R2["Output: folder containing<br/>.db3 file + metadata.yaml"]

    Q2 -->|"MCAP"| A3["ros2 bag record /scan -s mcap"]
    A3 --> R3["Output: single .mcap file<br/>Compatible with Foxglove Studio"]

    classDef q fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef cmd fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef out fill:#dcfce7,stroke:#16a34a,color:#14532d
    class Start,Q1,Q2 q
    class A1,A2,A3 cmd
    class R1,R2,R3 out
```

| ROS Version | Command | Output |
|:---|:---|:---|
| ROS1 | `rosbag record /scan` | Single `.bag` file |
| ROS2 (default) | `ros2 bag record /scan` | Folder with `.db3` + `metadata.yaml` |
| ROS2 (MCAP) | `ros2 bag record /scan -s mcap` | Single `.mcap` file |

---

## 4. Bag File Contents

Each recorded scan stores four pieces of information: the topic it originated from, its message type, the sensor readings themselves, and the exact time it was captured.

```mermaid
flowchart LR
    Bag["Bag File Entry"] --> T["Topic Name<br/>/scan"]
    Bag --> Type["Message Type<br/>sensor_msgs/LaserScan"]
    Bag --> Data["Scan Data<br/>ranges · angles · intensities"]
    Bag --> Time["Timestamp<br/>Time of capture"]

    classDef bag fill:#f3e8ff,stroke:#9333ea,color:#581c87
    classDef content fill:#fce7f3,stroke:#db2777,color:#831843
    class Bag bag
    class T,Type,Data,Time content
```

---

## 5. Recording Multiple Topics (SLAM Use Case)

A LiDAR scan alone does not indicate the position of the drone at the time of capture. For SLAM (mapping), the LiDAR topic must be recorded alongside position and motion data so that all sources remain synchronized during playback.

```mermaid
flowchart LR
    subgraph Sources["ROS Topics"]
        S1["/scan<br/>LiDAR data"]
        S2["/tf<br/>Position transforms"]
        S3["/odom<br/>Motion data"]
    end

    Cmd["ros2 bag record /scan /tf /odom<br/>-o my_slam_run"]

    S1 --> Cmd
    S2 --> Cmd
    S3 --> Cmd
    Cmd --> Out["my_slam_run<br/>All topics synchronized"]

    classDef src fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef cmd fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef out fill:#dcfce7,stroke:#16a34a,color:#14532d
    class S1,S2,S3 src
    class Cmd cmd
    class Out out
```

---

## 6. Command Reference

Once a bag file exists, four operations cover most use cases: recording, playback, inspection, and format conversion.

```mermaid
flowchart TD
    Root["Bag File Operations"] --> Rec["Record"]
    Root --> Play["Playback"]
    Root --> Info["Inspect"]
    Root --> Conv["Convert"]

    Rec --> Rec1["ros2 bag record /scan"]
    Play --> Play1["ros2 bag play my_slam_run"]
    Info --> Info1["ros2 bag info my_slam_run"]
    Conv --> Conv1["ros2 bag convert -i my_slam_run -o output.yaml"]

    classDef root fill:#e0e7ff,stroke:#4f46e5,color:#312e81
    classDef action fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef cmd fill:#dcfce7,stroke:#16a34a,color:#14532d
    class Root root
    class Rec,Play,Info,Conv action
    class Rec1,Play1,Info1,Conv1 cmd
```

| Task | Command |
|:---|:---|
| Start recording | `ros2 bag record /scan` |
| Replay recording | `ros2 bag play my_slam_run` |
| Inspect contents | `ros2 bag info my_slam_run` |
| Convert format | `ros2 bag convert -i my_slam_run -o output.yaml` |

---

## Summary

- A LiDAR only transmits live data — it does not store anything independently.
- ROS captures this data through a **topic** and writes it to a **bag file** when recording is active.
- The output format (`.bag`, `.db3`, `.mcap`) depends on the ROS version and storage plugin used.
- For SLAM applications, record the LiDAR topic together with position data (`/tf`, `/odom`) to keep them synchronized.
- Bag files can be replayed, inspected, or converted to other formats at any time after recording.

---

*Reference documentation compiled from project notes.*
