# HoneyMine Rescue Swarm
### AI-Powered Underground Mine Safety, Monitoring and Rescue System

> A fault-tolerant swarm of rugged rescue rovers that maps a mine, detects toxic gas and trapped workers, and keeps working when tunnels cut the radio links or a robot is lost, while a human rescue commander stays in control of every critical decision.

**Smart India Hackathon | Problem Theme: Underground Mine Safety, Monitoring and Rescue (Jharkhand coal mines)**

| | |
|---|---|
| **Team / Lead** | _[Team name]_ / _[Lead name]_ |
| **Domain** | Robotics · AI/ML · Swarm Coordination · IoT · Mine Safety |
| **Core IP** | Honey Network Protocol (fault-tolerant swarm coordination, originally built for underwater acoustic swarms) |
| **Status** | Coordination core simulation-validated; mine prototype in development |

---

## Table of Contents
1. [Problem Statement](#1-problem-statement)
2. [Solution Overview](#2-solution-overview)
3. [Innovation and Creativity](#3-innovation-and-creativity)
4. [System Architecture](#4-system-architecture)
5. [System Components and Roles](#5-system-components-and-roles)
6. [End-to-End Workflow](#6-end-to-end-workflow)
7. [Honey Network Protocol: Adapted for Mines](#7-honey-network-protocol-adapted-for-mines)
8. [AI / ML Pipeline](#8-ai--ml-pipeline)
9. [Hardware and Resources](#9-hardware-and-resources)
10. [Software Stack](#10-software-stack)
11. [Simulation, Testing and Validation](#11-simulation-testing-and-validation)
12. [Safety and Compliance](#12-safety-and-compliance)
13. [Benefits and Impact](#13-benefits-and-impact)
14. [Feasibility and Roadmap](#14-feasibility-and-roadmap)
15. [Risks and Mitigations](#15-risks-and-mitigations)
16. [Future Scope](#16-future-scope)
17. [Presentation (PPT) Slide Map](#17-presentation-ppt-slide-map)

---

## 1. Problem Statement

Underground coal mines in Jharkhand face recurring life-threatening hazards:

| Hazard | Why it is dangerous |
|---|---|
| **Toxic and explosive gases** (methane, CO, H₂S, low O₂) | Cause poisoning, suffocation and explosions |
| **Tunnel collapse / roof fall** | Traps workers and blocks access |
| **Flooding** | Cuts off galleries; rescuers cannot enter |
| **Poor visibility** (dust, smoke, darkness) | Slows search and increases risk |
| **Communication loss** | Rock, bends and debris block radio, so nobody knows what is happening underground |

**The gap:** during an emergency, rescue teams often enter blind. They lack real-time information about gas levels, tunnel condition and worker location, which delays response and puts rescuers at risk.

**Need:** an intelligent robotic system that explores, senses, detects hazards, locates trapped workers and streams the picture to a surface control station without exposing rescuers to danger.

---

## 2. Solution Overview

**HoneyMine Rescue Swarm** is a coordinated team of robots, not a single rover.

- A **Queen rover** leads exploration, fuses data and proposes rescue plans.
- Multiple low-cost **Worker rovers** explore side galleries in parallel.
- Small **relay beacons**, dropped like breadcrumbs, extend the communication mesh deeper into the mine.
- A **Surface Control Station** shows the live map, gas heatmap, thermal/video feeds and a **risk map**, and requires **human approval** before any critical action.
- The **Honey Network Protocol** keeps the swarm coordinated when links drop, the network splits or a robot is lost.

**One-line pitch:** *Autonomy that heals itself, but asks before it acts.*

### What the system does

| Capability | How |
|---|---|
| Monitor mine conditions | Gas, temperature, humidity sensors on every robot |
| Detect hazards | Threshold + AI anomaly detection on gas, heat and structure |
| See in the dark | Thermal + night-vision cameras |
| Locate trapped workers | AI person detection on thermal + camera fusion |
| Map the mine | LiDAR/IMU SLAM building a live tunnel map with hazard overlay |
| Stay connected | Self-healing multi-hop mesh + dropped relay beacons |
| Guide rescuers | Colour-coded safe / caution / no-go risk map |

---

## 3. Innovation and Creativity

| # | Innovation | Why it matters |
|---|---|---|
| 1 | **Swarm instead of single rover** | Parallel search, no single point of failure |
| 2 | **Honey Network Protocol** (quorum election, exactly-once delivery, self-healing) | Swarm survives robot loss and network partition |
| 3 | **Human-approval gate built into the architecture** | Auditable, safe autonomy; rescuers stay in command |
| 4 | **Breadcrumb relay beacons** | Robots build their own comms infrastructure as they go deeper |
| 5 | **Multi-sensor fusion** (thermal + camera + gas + LiDAR) | Fewer false alarms; reliable in smoke, dust and darkness |
| 6 | **Rescue risk map** | Converts raw sensor data into decisions: where it is safe to send people |
| 7 | **Digital twin first** | Missions rehearsed with fault injection before any real deployment |
| 8 | **Cross-domain proven core** | The coordination core comes from an underwater swarm project, where the channel is far harsher than a mine, so it is a demanding foundation to adapt |
| 9 | **Flood extension** | Underwater swarm design becomes submersible scouts for flooded galleries (Phase 2) |

### Design principle
Two loops are deliberately separated:

- **Autonomous loop:** sense → map → plan (runs freely)
- **Human-gated loop:** approve → act (nothing crosses over without commander approval)

---

## 4. System Architecture

### 4.1 Layered architecture

```
┌──────────────────────────────────────────────────────────────┐
│ L5  MISSION / COMMAND   risk map · plan approval · digital twin│
├──────────────────────────────────────────────────────────────┤
│ L4  COORDINATION        Honey Network Protocol: election ·     │
│                         ARQ · relay routing · heartbeat ·      │
│                         task allocation                        │
├──────────────────────────────────────────────────────────────┤
│ L3  AUTONOMY            SLAM · localisation · path planning ·  │
│                         obstacle avoidance                     │
├──────────────────────────────────────────────────────────────┤
│ L2  PERCEPTION          person detection · gas analytics ·     │
│                         hazard detection · sensor fusion       │
├──────────────────────────────────────────────────────────────┤
│ L1  PLATFORM / SENSORS  drive · IMU · LiDAR · thermal · camera │
│                         · gas · temp/humidity · radio          │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Deployment topology

```
                ┌──────────────────────────────────┐
                │    SURFACE CONTROL STATION       │
                │  live map · gas heatmap · video  │
                │  risk map · HUMAN APPROVAL GATE  │
                └───────────────┬──────────────────┘
                                │  mine backbone / tether / surface link
                     ┌──────────┴──────────┐
                     │   BASE RELAY NODE   │  (at shaft / portal)
                     └──────────┬──────────┘
                                │  self-healing mesh (Honey Protocol)
        ┌───────────────────────┼─────────────────────────┐
        │                       │                         │
   ┌────┴─────┐   ┌─────────────┴───┐   ┌────────────┐   ┌┴───────────┐
   │  QUEEN   │◄─►│  RELAY BEACON   │◄─►│  WORKER 1  │   │  WORKER 2  │
   │  rover   │   │  (dropped)      │   │  side gal. │   │  side gal. │
   └────┬─────┘   └─────────────────┘   └────────────┘   └────────────┘
        └──► drops beacons whenever link quality falls below threshold
```

### 4.3 Data flow

```
[Sensors] → [Perception] → [Fusion] → HAZARD / CONTACT report
                                          │   (Honey: seq + ACK + relay)
                                          ▼
                        [Queen: shared map + risk map + planner]
                                          │   proposed rescue plan
                                          ▼
                        [Surface Station: HUMAN APPROVAL GATE]
                                          │   approve / revise
                                          ▼
                        [Queen assigns tasks → Workers execute]
                                          │
                                          ▼
                        [Results aggregated → live map + report]
```

---

## 5. System Components and Roles

| Component | Role | Key hardware / tech |
|---|---|---|
| **Surface Control Station** | Live dashboard, video/thermal viewing, risk map, human approval gate | Laptop/PC, web dashboard |
| **Base Relay Node** | Bridge between the surface link and the underground mesh | LoRa/Wi-Fi mesh gateway |
| **Queen Rover** | Elected leader: fuses data, builds map, plans, coordinates | Onboard GPU/AI compute, LiDAR, thermal + RGB, full gas suite |
| **Worker Rovers** | Explore assigned galleries, sense, relay | Low-cost chassis, thermal/camera, gas, basic IMU |
| **Relay Beacons** | Extend mesh range; act as localisation anchors | Small battery radio nodes (optionally UWB) |
| **Honey Network Protocol stack** | Coordination fabric | Election, ARQ, relay, heartbeat, registry replication |
| **AI Perception Module** | Detect people, hazards, anomalies | CNN detectors, fusion, anomaly detection |
| **Digital Twin** | Simulate mines and failures; rehearse missions | ROS 2, Gazebo |

### Sensor suite per robot

| Sensor | Purpose |
|---|---|
| Gas: CH₄, CO, H₂S, O₂ | Toxic / explosive atmosphere monitoring |
| Temperature + humidity | Fire risk, heat stress, environment monitoring |
| Thermal camera | Detect body heat, hot spots, fire precursors |
| RGB / night-vision camera | Visual confirmation, structure inspection |
| LiDAR | Tunnel mapping (SLAM), obstacle detection, deformation cues |
| IMU + wheel encoders | Odometry and pose estimation |
| Radio module | Mesh communication |

---

## 6. End-to-End Workflow

### 6.1 Mission phases

| Phase | What happens | Human role |
|---|---|---|
| **0. Pre-deployment** | Load mine layout (if available), check sensors, deploy swarm at the entry | Commander launches mission |
| **1. Election and formation** | Robots elect a Queen by quorum; base relay node registers the swarm | Monitor |
| **2. Exploration** | Queen and workers split the gallery network, map tunnels, sample gas/heat/humidity | Monitor live map |
| **3. Hazard and person detection** | Perception flags gas spikes, heat anomalies, blockages, and possible trapped workers | Review alerts |
| **4. Fusion and risk mapping** | Queen fuses reports into a shared map with safe / caution / no-go zones | Review risk map |
| **5. Plan proposal** | Queen proposes a rescue plan (routes, target areas, sensor sweeps) | **Approves or revises** |
| **6. Execution** | Workers carry out the approved tasks and report results | Approve any critical step |
| **7. Continuous healing** | Heartbeats, relay repair, task reassignment, beacon drops | Automatic |
| **8. Handover** | Final map, gas history, victim location report, video logs | Guides rescue team |

### 6.2 Communication workflow

1. A robot's perception detects a hazard or possible person.
2. The robot sends a compact **report** to the Queen with a sequence number (ACK + bounded retransmission; relayed if out of range).
3. The Queen fuses reports into the shared map and updates the risk map.
4. The Queen sends the proposed plan to the Surface Station via the base relay.
5. **The commander approves or revises.**
6. On approval, the Queen decomposes the mission and assigns galleries or tasks to workers.
7. Workers execute and report; the Queen aggregates the results.
8. Throughout: heartbeats, topology updates, relay re-discovery, task reassignment.

### 6.3 Failure scenarios and system response

| Failure | Detection | Response |
|---|---|---|
| **Worker lost** (collapse, fault) | Heartbeat SUSPECTED → DEAD | Its gallery is reassigned to another worker |
| **Queen lost** | Heartbeat timeout | Quorum election of a new Queen; registry replicated to a standby, so mission state is preserved |
| **Link broken** (tunnel bend, debris) | ACK timeouts, route quality drop | Multi-hop rerouting; a relay beacon is dropped |
| **Network partition** | Quorum check | Only one Queen may be active (no split-brain); the minority side holds safe behaviour |
| **Duplicate or replayed command** | Sequence numbers | Exactly-once execution |
| **Gas above threshold** | Sensor alert | Zone marked no-go; robots avoid it; alert raised at the station |

### 6.4 Emergency response sequence

```
Incident reported
      │
      ▼
Deploy swarm at entry ──► Elect Queen ──► Explore + map in parallel
      │
      ▼
Detect gas / heat / blockage / possible person
      │
      ▼
Fuse → risk map → proposed rescue plan
      │
      ▼
   COMMANDER APPROVAL
      │
      ▼
Targeted search → victim location + safe route → rescue team goes in informed
```

---

## 7. Honey Network Protocol: Adapted for Mines

The Honey Network Protocol was designed for the **underwater acoustic channel**: extremely low bandwidth, high latency, packet loss and frequent partition. Underground tunnels share the same class of problems (no GPS, intermittent links, partitions, unpredictable loss), so the same safety guarantees carry over.

### 7.1 Guarantees

| Mechanism | Guarantee |
|---|---|
| Quorum election with monotonic terms | At most one Queen at a time (no split-brain), even under partition |
| Sequence numbers + ACK + bounded retransmission | Every command is delivered and applied exactly once |
| Multi-hop relay with route repair | Robots beyond direct range remain reachable |
| Heartbeat failure ladder (SUSPECTED → DEAD) | Failures are detected within a bounded time |
| Registry replication to a standby | A new Queen inherits worker registry and task state |
| Human approval gate | No critical action without explicit authorisation |

### 7.2 Adaptation from underwater to underground

| Parameter | Underwater original | Mine version |
|---|---|---|
| Medium | Acoustic | Radio (LoRa / mesh Wi-Fi / UWB) plus wired/tether at the base |
| Latency | Seconds (slow propagation) | Milliseconds to low seconds |
| Timers (heartbeat, ACK timeout, suspicion thresholds) | Tuned for seconds-scale delay | Re-tuned for the tunnel radio channel |
| Bandwidth | Extremely low | Low to moderate; still transmit compact data first, video only on demand |
| Localisation | Inertial + acoustic | IMU + wheel odometry + LiDAR SLAM + beacon ranging |
| Relay | Worker vehicles | Worker rovers + **dropped relay beacons** |

### 7.3 Validation status (stated honestly)
- The coordination core was validated in a simulation harness across roughly **1,000 randomised fault-injection trials**, with **zero split-brain events and zero duplicate mission actions**, using underwater acoustic-channel parameters.
- **Not yet validated for the mine channel.** Re-validating with mine-channel parameters and a tunnel digital twin is part of the plan (Section 11).
- The result is simulation-validated, not hardware-proven.

---

## 8. AI / ML Pipeline

### 8.1 Overview

```
thermal ─► preprocess ─► person/heat detector ─┐
camera  ─► enhance ────► object detector ──────┼─► FUSION ─► confidence-scored CONTACT
LiDAR   ─► SLAM / structure features ──────────┤
gas + temp + humidity ─► anomaly + threshold ──┘
                                     │
                                     ▼
                           RISK MAP (safe / caution / no-go)
```

### 8.2 Modules

| Module | Function | Approach |
|---|---|---|
| **Person detection** | Find trapped workers | CNN detector (YOLO-family) on thermal + RGB, with confidence-weighted decision-level fusion |
| **Hazard detection** | Gas leaks, heat spots, smoke, blockages, water | Threshold rules + time-series anomaly detection |
| **Structural risk indicator** | Flag possible roof/wall deformation or debris | LiDAR/vision change detection between passes (a **risk indicator**, not a guarantee of stability) |
| **Sensor fusion** | Combine sources into one confidence score | Bayesian / confidence-weighted fusion; uncertainty gating to reduce false alarms |
| **SLAM and mapping** | Live tunnel map with hazard overlay | LiDAR + IMU SLAM |
| **Task allocation** | Assign galleries to robots | Consensus-based allocation (CBBA-style) |
| **Path planning** | Exploration and coverage | Frontier-based exploration, coverage planning |
| **Risk map generation** | Decision support for the commander | Rules + learned scoring over fused data |

### 8.3 Handling challenges

| Challenge | Mitigation |
|---|---|
| Little labelled mine data | Pretrain on public thermal/person datasets; synthetic data from simulation; fine-tune on staged tests |
| Smoke, dust, heat clutter | Multi-sensor fusion; uncertainty gating; human review before action |
| Limited onboard compute | Model compression / lightweight models on the Queen; simple models on workers |
| Bandwidth | Send compact detections first; stream video only when requested |

---

## 9. Hardware and Resources

> Prices are **indicative estimates for a prototype** and should be verified against current vendor listings. The prototype is a demonstrator, not a certified explosion-proof product (see Section 12).

### 9.1 Prototype hardware (per robot type)

| Subsystem | Queen rover | Worker rover |
|---|---|---|
| **Chassis** | Rugged 4WD/tracked platform | Compact 4WD chassis |
| **Compute** | Jetson-class module or Raspberry Pi 5 | ESP32 / Raspberry Pi |
| **Mapping sensor** | 2D LiDAR (e.g., RPLidar class) | Optional ultrasonic / small LiDAR |
| **Thermal camera** | Thermal array/camera (e.g., MLX90640 class or better) | Thermal array |
| **Vision** | Low-light / night-vision camera | Low-light camera |
| **Gas** | CH₄, CO, H₂S, O₂ sensors | CH₄, CO (minimum) |
| **Env.** | Temperature + humidity | Temperature + humidity |
| **Navigation** | IMU + wheel encoders | IMU + encoders |
| **Comms** | LoRa + Wi-Fi mesh | LoRa / ESP-NOW mesh |
| **Power** | Li-ion battery pack + regulation | Li-ion pack |

### 9.2 Additional hardware
- **Relay beacons:** small radio nodes with battery (3 to 5 for the demo)
- **Base relay node:** gateway at the entry
- **Surface station:** laptop running the dashboard

### 9.3 Sensor note
Low-cost hobby gas sensors are fine for a **demonstration** but are not reliable for real mine deployment. A production version needs calibrated, certified methane (NDIR) and electrochemical sensors.

### 9.4 Human and time resources

| Resource | Need |
|---|---|
| Robotics / embedded | Chassis, sensors, firmware |
| AI / ML | Detection, fusion, risk map |
| Software / dashboard | Web UI, backend, mesh integration |
| Simulation | Gazebo mine twin, fault injection |
| Documentation / pitch | PPT, demo video |

---

## 10. Software Stack

| Layer | Technology |
|---|---|
| Robot middleware | ROS 2 |
| Simulation / twin | Gazebo (optionally Isaac Sim) |
| Robot code | C++ (real-time), Python (perception, coordination) |
| AI / ML | PyTorch / TensorFlow, OpenCV, ONNX / TensorRT for edge inference |
| Coordination | Honey Network Protocol (custom, Python/C++) |
| SLAM | ROS 2 SLAM packages (LiDAR + IMU) |
| Backend | Python (FastAPI) / Node.js, WebSocket streaming |
| Dashboard | React / web UI with map, gas heatmap, video/thermal panels, approve/reject controls |
| Data | Time-series store for gas history; logs for after-action review |
| Messaging | MQTT / ROS 2 topics with Honey reliability layer |

---

## 11. Simulation, Testing and Validation

### 11.1 Methodology: digital-twin first
1. **Protocol validation:** re-run fault-injection Monte Carlo with mine-channel parameters
2. **Perception development:** person and hazard detection on thermal/RGB datasets
3. **Integrated simulation:** Gazebo mine tunnel with Queen + workers + beacons
4. **Hardware-in-the-loop:** real compute, simulated physics
5. **Staged physical trials:** corridor, then basement/tunnel-like test environment

### 11.2 Test scenarios

| Scenario | What is checked |
|---|---|
| Queen loss mid-mission | Re-election, no state loss |
| Worker loss | Task reassignment |
| Tunnel link cut | Relay repair / beacon drop |
| Network partition | No split-brain |
| Gas spike | Zone marked no-go, robots reroute, alert raised |
| Person in smoke or dark | Detection with fusion |
| Duplicate / replayed command | Exactly-once execution |

### 11.3 Metrics

| Metric | Type |
|---|---|
| Split-brain events, duplicate mission actions | Safety (target: zero) |
| Person detection precision / recall, false alarms | Perception |
| Fault-detection latency, election recovery time | Coordination |
| Map coverage rate, mission completion under faults | Mission |
| Alert latency (hazard → station) | Operational |

*All figures above are targets to be measured, not results already achieved for the mine setting.*

---

## 12. Safety and Compliance

| Topic | Approach |
|---|---|
| **Explosive atmospheres** | Coal mines contain methane. A field product must be **intrinsically safe / flameproof** and meet applicable Indian mining safety rules (DGMS guidance) and explosive-atmosphere standards. The hackathon prototype is a demonstrator; a certification path is part of the roadmap. |
| **Human authority** | Every critical action requires commander approval; approvals are logged |
| **Fail-safe behaviour** | On lost link, robots hold safe behaviour instead of taking risky actions |
| **Security** | Signed messages, anti-replay via sequence numbers |
| **Ground-first design** | Rovers are the primary platform; aerial scouts are optional given dust, methane and ignition concerns |
| **Auditability** | Full logs of detections, decisions and approvals |

---

## 13. Benefits and Impact

| Stakeholder | Benefit |
|---|---|
| **Trapped workers** | Faster location and better chance of timely rescue |
| **Rescue teams** | Enter with real-time information; reduced exposure to gas, collapse and fire |
| **Mine operators** | Continuous monitoring, earlier hazard warnings, fewer incidents and downtime |
| **Regulators / safety officers** | Auditable logs and decisions |
| **Society / economy** | Fewer casualties; lower accident cost; safer mining regions |

### Technical benefits
- No single point of failure
- Works through comms degradation
- Modular: perception, coordination and planning can be upgraded independently
- Cost-attritable workers protect expensive sensing assets
- Reusable across other hazardous environments

### Commercial and scaling potential
- Mine safety monitoring services and rescue readiness kits
- Extension to tunnels, metro construction, industrial inspection and disaster response
- Coordination core reusable across domains

---

## 14. Feasibility and Roadmap

| Phase | Scope | Deliverable |
|---|---|---|
| **Phase 1 (Hackathon MVP)** | 1 to 2 rovers, gas + temp/humidity + thermal + camera, live dashboard, relay beacon, basic Honey mesh handoff | Working demo + dashboard |
| **Phase 1 (Simulation)** | Gazebo mine twin: Queen + 3 to 5 workers, fault injection | Simulation demo video + metrics |
| **Phase 2** | Full swarm on hardware, LiDAR SLAM, risk map, field-like trials | Validated prototype |
| **Phase 3** | Intrinsically safe redesign, certified sensors, pilot with a mine operator | Pilot deployment |
| **Phase 4** | Flooded-gallery submersible scouts; aerial scouts; federated learning | Extended platform |

---

## 15. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Methane ignition from electronics | Critical | Intrinsic-safety design path; demonstrator disclaimer |
| Comms dropout | High | Mesh, relay beacons, Honey partition-safety |
| False alarms / missed detections | High | Fusion, uncertainty gating, human review |
| Localisation drift | Medium | SLAM, beacon ranging, loop closure |
| Sim-to-real gap | Medium | Staged trials, domain adaptation |
| Dust / heat damage | Medium | Sealed enclosures, rugged chassis |
| Battery endurance | Medium | Energy-aware planning, swappable packs |
| Cyber risk | Medium | Signed messages, anti-replay |
| Over-scoping for the deadline | High | MVP-first plan (Phase 1) |

---

## 16. Future Scope

- **Flooded-gallery scouts:** adapt the underwater swarm vehicles as submersible workers
- **Aerial scouts:** compact drones for large open voids, subject to safety review
- **Federated learning:** robots improve shared models without sending raw data
- **Wearable worker tags:** improved locating of trapped personnel
- **Structural health modelling:** long-term deformation trends
- **Cross-domain use:** tunnels, construction, industrial plants, disaster zones

---

## 17. Presentation (PPT) Slide Map

| Slide | Title | Content source |
|---|---|---|
| 1 | Title | Project name, tagline, team |
| 2 | The Problem | Section 1 |
| 3 | Our Solution | Section 2 |
| 4 | Innovation | Section 3 |
| 5 | System Architecture | Section 4 |
| 6 | Components and Roles | Section 5 |
| 7 | Workflow | Section 6 |
| 8 | Failure Handling | Section 6.3 |
| 9 | Honey Network Protocol | Section 7 |
| 10 | AI Pipeline | Section 8 |
| 11 | Hardware and Resources | Section 9 |
| 12 | Tech Stack | Section 10 |
| 13 | Validation and Metrics | Section 11 |
| 14 | Safety and Compliance | Section 12 |
| 15 | Benefits and Impact | Section 13 |
| 16 | Roadmap | Section 14 |
| 17 | Risks | Section 15 |
| 18 | Future Scope and Close | Section 16 |

---

*Scope note: this document describes a research and prototype-level rescue and monitoring system. Performance figures are targets to be validated. The coordination protocol is simulation-validated, not hardware-proven.*
