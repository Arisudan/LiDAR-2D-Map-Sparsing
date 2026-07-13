# Autonomous Drone Pipeline — SLAM, LiDAR & Networking Reference

---

## 1. Sensor Stack

```mermaid
flowchart LR
    A["ORB-SLAM3"] <--> B["Intel RealSense D435i"]
    C["2D LiDAR"] --> D["Time of Flight (ToF) Principle"]

    classDef slam fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef lidar fill:#dcfce7,stroke:#16a34a,color:#14532d
    class A,B slam
    class C,D lidar
```

---

## 2. LiDAR → SBC → ROS → Visualization Pipeline

```mermaid
flowchart TD
    subgraph HW["Hardware"]
        L["2D LiDAR"]
    end

    subgraph CONN["Physical Connection"]
        C1["USB-to-Serial"]
        C2["Direct UART TTL Pins"]
    end

    subgraph DRV["Driver Layer"]
        D1["YDLiDAR SDK"]
        D2["RPLiDAR SDK"]
    end

    subgraph COMP["SBC + ROS"]
        SBC["SBC"]
        ROS["ROS<br/>sensor_msgs/LaserScan"]
    end

    subgraph VIS["Visualization — WiFi / TCP-IP"]
        V1["Rviz"]
        V2["Foxglove Studio"]
    end

    L --> C1
    L --> C2
    L --> D1
    L --> D2
    C1 --> SBC
    C2 --> SBC
    D1 --> SBC
    D2 --> SBC
    SBC <--> ROS
    ROS --> V1
    ROS --> V2

    classDef hw fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef drv fill:#e0e7ff,stroke:#4f46e5,color:#312e81
    classDef comp fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef vis fill:#dcfce7,stroke:#16a34a,color:#14532d
    class L,C1,C2 hw
    class D1,D2 drv
    class SBC,ROS comp
    class V1,V2 vis
```

---

## 3. Data Format Cheat Sheet

```mermaid
flowchart LR
    subgraph Formats["Map / Sensor Data Formats"]
        F1["ROS / ROS2 Bag<br/>.bag · .db3 · .mcap"]
        F2["CSV<br/>.csv · .txt"]
        F3["JSON<br/>.json"]
    end

    F1 --> U1["Industry Standard"]
    F2 --> U2["Custom Python Code"]
    F3 --> U3["Web-based Robotic Apps"]

    classDef fmt fill:#f3e8ff,stroke:#9333ea,color:#581c87
    classDef use fill:#fce7f3,stroke:#db2777,color:#831843
    class F1,F2,F3 fmt
    class U1,U2,U3 use
```

| Format | Extension(s) | Best For |
|:---|:---:|:---|
| ROS / ROS2 Bag | `.bag` `.db3` `.mcap` | Industry standard |
| CSV | `.csv` `.txt` | Custom Python code |
| JSON | `.json` | Web-based robotic apps |

**Custom Python setup:**
```bash
pip install pyserial rplidar-roboticia
```

---

## 4. TCP vs UDP

```mermaid
flowchart TD
    subgraph TCP["TCP — Transmission Control Protocol"]
        T0["3-Way Handshake<br/>(before sending data)"]
        T0 --> T1["Tracks"]
        T0 --> T2["Sequences"]
        T0 --> T3["Acknowledgement"]
        T3 --> TU["✅ Use when ACCURACY is critical"]
    end

    subgraph UDP["UDP — User Datagram Protocol"]
        U0["Connectionless<br/>'fire-and-forget'"]
        U0 --> U1["No Handshake"]
        U0 --> U2["No Persistent Connection"]
        U0 --> U3["No Retransmissions"]
        U3 --> UU["⚡ Use when LOW LATENCY matters"]
    end

    classDef tcp fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef udp fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    class T0,T1,T2,T3,TU tcp
    class U0,U1,U2,U3,UU udp
```

| | **TCP** | **UDP** |
|---|:---:|:---:|
| Connection | Handshake-based | Connectionless |
| Reliability | Guaranteed / ordered | Best-effort |
| Retransmission | ✅ Yes | ❌ No |
| Overhead | High | Low |
| Ideal Use | Accuracy-critical data | Low-latency streaming |

---

*Digitized from handwritten lab notebook for GitHub reference.*
