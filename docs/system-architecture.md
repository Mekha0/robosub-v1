# RoboSub Software Architecture

## Overview

This repository contains the high-level software architecture for an autonomous underwater vehicle (AUV) designed to compete in RoboSub. The system is built around a layered architecture that separates concerns between perception, planning, control, and actuation. This design prioritizes modularity, real-time performance, and fault tolerance—with specific emphasis on robust operation **without a DVL**.

The architecture runs across three primary compute platforms:

| Hardware | Role | Operating System |
|----------|------|------------------|
| NVIDIA Jetson Orin Nano | Main computer (perception & planning) | Ubuntu 24.04 (JetPack 7.2) |
| Raspberry Pi 4 | Companion computer (I/O, fail-safe, disturbance observer) | BlueOS (Docker-based) |
| Cube Orange+ | Flight controller (real-time control) | ArduSub (ChibiOS/RT) |

---

## System Architecture Diagram
![System Architecture](diagrams/system-architecture.svg)

The diagram above illustrates the six layers of the architecture and their interconnections. Each layer is described in detail below.

---

## Layer Descriptions

### 1. Power Layer

**Purpose:** Provides isolated power distribution to all system components.

**Components:**
- **Thruster Batteries:** Two 7000mAh 4S LiPo batteries dedicated exclusively to propulsion
- **Electronics Batteries:** Two 4500mAh 4S LiPo batteries for all logic and sensing components
- **Voltage Regulators:** Step down voltage to stable 5V and 3.3V rails for sensitive electronics
- **Electronic Speed Controllers (ESCs):** Convert PWM signals to motor power for thrusters

**Key Design Decision:** Thrusters and electronics are on separate power buses. This prevents motor noise and voltage sag from affecting the flight controller IMU, cameras, or processing units.

**Flow:** Power flows from batteries through regulators/ESCs to all downstream components. No data flows through this layer.

---

### 2. Perception Layer

**Purpose:** Converts raw sensor data into usable information about the robot's environment and state. This layer is responsible for generating the highest-quality state estimate possible given the **absence of a DVL**.

**Components:**
- **OAK-D Lite Stereo Camera:** Provides RGB images, depth maps, and onboard neural inference for object detection at 30Hz
- **ArduCam Global Shutter Camera:** Captures high-speed images at 60Hz without motion blur for optical flow and feature tracking
- **Blue Robotics Bar30 Depth Sensor:** Provides accurate depth measurements via I2C to the flight controller
- **IMU (Internal to Cube Orange):** Provides acceleration and angular rate data at 400Hz
- **Visual Odometry (VO) Pipeline:** Two-stage system for robust velocity estimation without DVL:
  - **Primary:** OAK-D stereo + depth → ORB-SLAM3 (stereo mode) → 6-DOF pose at 30Hz
  - **Fallback:** ArduCam global shutter → feature tracking (KLT) + optical flow → velocity estimate at 60Hz (scale recovered from Bar30 depth)
- **VO Quality Monitor:** Continuously evaluates feature count, disparity variance, and tracking stability. Publishes a `vo_quality` metric (0-1) to the Autonomy Layer.
- **Sensor Fusion Module:** Merges VO estimates, IMU, depth, and compass data into a coherent state estimate using an Extended Kalman Filter (EKF) running on the Jetson

**Key Design Decisions:**
- The ArduCam Global Shutter is **critical** for fallback odometry. Rolling shutter would make velocity estimation impossible during motion.
- The VO Quality Monitor is **essential**—without it, the autonomy layer cannot know when to trust visual data versus fallback dead reckoning.
- Fogging prevention is handled at this layer: temperature/humidity sensors trigger a "surface-and-wipe" behavior if internal humidity exceeds 70%.

**Flow:** Raw sensor data enters this layer. Processed state estimates (position, velocity, attitude) and `vo_quality` metadata exit to the Autonomy Layer via ROS 2 topics.

---

### 3. Autonomy Layer

**Purpose:** High-level decision making, path planning, and motion control. This layer adapts its behavior based on the quality of visual perception.

**Components:**
- **Mission Planner:** Implements a behavior tree state machine that determines task execution order with retry/skip logic for failed tasks
- **Path Planner:** Generates waypoints between current position and target, accounting for obstacles and uncertainty bounds
- **Motion Controller:** Converts desired paths into velocity commands (surge, sway, heave, yaw) for the flight controller
- **Dead Reckoning Fallback Module:** Activates when `vo_quality < 0.3`. Uses:
  - Integrated IMU acceleration (with bias correction)
  - Thruster telemetry (RPM → estimated force → acceleration model)
  - Depth sensor for heave constraint
  - Position uncertainty grows linearly with time; triggers re-acquisition behavior when uncertainty > 0.5m
- **Re-Acquisition Behavior:** When position uncertainty exceeds threshold, executes a spiral search pattern until visual landmarks are re-detected or a timeout occurs
- **Extended Kalman Filter (EKF):** Fuses visual odometry, IMU, depth, and compass data into a single state estimate. **Adaptive covariance:** when `vo_quality` drops, EKF covariance inflates to reflect reduced confidence.
- **ROS 2 Middleware:** Provides publish-subscribe communication between all nodes

**Key Design Decisions:**
- The Autonomy Layer runs entirely on the Jetson Orin Nano, leveraging GPU acceleration for path planning and state estimation.
- The Dead Reckoning Fallback is **not** a separate mode—it's a continuous fallback that interpolates between visual and non-visual estimates based on `vo_quality`.
- Re-acquisition behavior is triggered by uncertainty, not time, making it robust to variable visual conditions.

**Flow:** Receives processed perception data (including `vo_quality`) as input. Outputs desired velocity commands to the Control Layer via ROS 2 topics.

---

### 4. Control Layer

**Purpose:** Manages I/O, provides fail-safe functionality, bridges ROS 2 to MAVLink, and implements current disturbance rejection.

**Components:**
- **BlueOS:** Companion computer operating system built on Docker, provides web-based configuration and extension management
- **MAVLink Router:** Routes MAVLink packets between the Jetson (via ROS 2) and the Cube Orange (via UART)
- **I/O Manager:** Polls leak sensors, temperature sensors, humidity sensors, and controls servos via GPIO/I2C on the Raspberry Pi
- **Current Disturbance Observer:** Runs on the Raspberry Pi at 50Hz. Estimates water current direction and magnitude by comparing:
  - Desired velocity (from Autonomy Layer)
  - Achieved velocity (from IMU + thruster telemetry)
  - Applies feedforward compensation to velocity commands before forwarding to MAVLink
- **Fail-Safe Logic:** Monitors heartbeats from the Jetson and flight controller. If communication is lost, commands the vehicle to surface or hold position
- **Data Logger:** Records estimated state, actual commands, sensor data, and `vo_quality` for post-mission analysis

**Key Design Decisions:**
- The Raspberry Pi acts as an intermediary between the high-level Jetson and the low-level flight controller, providing hardware isolation.
- The Current Disturbance Observer runs on the Pi (not the Jetson) to offload computation and ensure it continues running even if the Jetson crashes.
- Thruster telemetry (RPM, current draw) is logged here and used both for dead reckoning and current estimation.

**Flow:** Receives ROS 2 velocity commands from the Autonomy Layer. Applies current compensation, translates to MAVLink, and forwards to the Hardware Abstraction Layer. Also monitors system health and can override commands if necessary.

---

### 5. Hardware Abstraction Layer

**Purpose:** Converts desired motion into actuator commands using real-time control loops.

**Components:**
- **ArduSub Firmware:** Open-source autopilot firmware specifically designed for underwater vehicles
- **EKF3:** Onboard Extended Kalman Filter. **Configured to accept external vision input** from the Jetson, but only fuses it when `vo_quality > 0.5` (received via MAVLink `VISION_POSITION_ESTIMATE` messages)
- **PID Controllers:** Run at 400Hz for depth hold, heading hold, pitch/roll stabilization. Includes anti-windup and derivative kick protection
- **Actuator Mixer:** Maps desired forces to individual thruster commands based on vehicle geometry (vectored 8-thruster configuration)
- **ChibiOS/RT:** Real-time operating system that guarantees deterministic execution of control loops

**Key Design Decisions:**
- The flight controller runs bare-metal firmware (not Linux) to ensure microsecond-precision control loops.
- External vision is fused **conditionally**—if VO quality is poor, the onboard EKF3 relies solely on internal IMU + depth + compass.
- This layer does **not** perform current estimation—that is handled upstream in the Control Layer.

**Flow:** Receives MAVLink commands from the Control Layer. Executes PID loops and outputs PWM signals to the Hardware Layer. Also sends IMU/depth data back up to the Perception Layer for sensor fusion.

---

### 6. Hardware Layer

**Purpose:** Physical actuation and signal distribution.

**Components:**
- **Custom PCB:** Routes PWM signals from the flight controller to thrusters and servos, provides I2C bus expansion for sensors
- **8x Bi-Directional Thrusters:** Provide 6-DOF motion (surge, sway, heave, roll, pitch, yaw) in a vectored configuration
- **High-Torque Waterproof Servos:** Actuate task mechanisms (torpedo launchers, marker droppers, grabber arms)
- **Leak/Temperature/Humidity Sensors:** Provide health monitoring data to the Control Layer via GPIO/I2C

**Key Design Decision:** The custom PCB centralizes all signal routing and power distribution for actuators, reducing wiring complexity and providing a single point for debugging.

**Flow:** Receives PWM signals from the Hardware Abstraction Layer. Converts electrical signals to mechanical motion. Returns sensor data (leak detection, temperature, humidity) back up to the Control Layer.

---

## Communication Flow

The following sequence describes how data flows through the system during a typical task:

1. **Perception:** OAK-D Lite detects a gate and estimates its 3D position via ORB-SLAM3. ArduCam runs parallel feature tracking. VO Quality Monitor publishes a confidence score (e.g., 0.85).

2. **Autonomy:** Mission Planner identifies "Navigate to Gate" as the current task. Path Planner generates waypoints. Motion Controller calculates desired velocity commands. `vo_quality = 0.85` → EKF uses visual odometry with high confidence.

3. **Control:** Raspberry Pi receives velocity commands via ROS 2. Current Disturbance Observer applies feedforward compensation. MAVLink Router translates to MAVLink packets. Fail-Safe Logic confirms Jetson heartbeat is valid.

4. **Hardware Abstraction:** Cube Orange receives MAVLink commands. EKF3 fuses external vision (since `vo_quality > 0.5`) with internal IMU. PID controllers calculate required thruster forces.

5. **Hardware:** Custom PCB routes PWM signals to ESCs. Thrusters spin to achieve desired motion. IMU and depth data are sent back to the Cube Orange for the next control loop.

### Failure Scenario (Visual Dropout):
1. **Perception:** OAK-D loses visual tracking. VO Quality Monitor drops to 0.1 and publishes warning.
2. **Autonomy:** Dead Reckoning Fallback Module activates. EKF covariance inflates. Re-acquisition behavior initiates a spiral search.
3. **Control:** Current compensation continues. Fail-Safe remains inactive (heartbeat valid).
4. **HAL:** Cube Orange EKF3 ignores external vision (`vo_quality < 0.5`) and relies on internal sensors.
5. **Recovery:** When visual features are re-acquired, `vo_quality` rises above 0.5. EKF covariance contracts. Re-acquisition behavior exits. Normal operation resumes.

### Loop Frequencies:
- Vision processing (primary): 30Hz
- Vision processing (fallback): 60Hz
- VO Quality Monitor: 30Hz
- ROS 2 communication: 50-100Hz
- MAVLink communication: 50Hz
- Current Disturbance Observer: 50Hz
- PID control loops: 400Hz
- PWM output: 400Hz
- Dead Reckoning Fallback: 50Hz

---

## Pre-Mission Calibration

Before each competition run, the following automated calibration routine executes:

1. **Visual Odometry Scale Calibration:** Vehicle moves forward 1m (measured by depth sensor + thruster RPM), VO scale factor is adjusted to match
2. **IMU Bias Calibration:** Stationary for 5 seconds, accelerometer and gyroscope biases are recorded
3. **Thruster Mapping:** A brief burst test verifies PWM → thrust mapping and identifies any failed thrusters
4. **Fogging Check:** Humidity sensor reading is verified. If >60%, anti-fog heaters activate until <50%

This calibration takes <30 seconds and runs automatically when the vehicle is powered on in the water.

---

## Redundancy & Fault Tolerance Summary

| Failure Mode | Mitigation |
|--------------|------------|
| OAK-D failure | ArduCam fallback odometry continues |
| Visual odometry dropout | Dead Reckoning Fallback + re-acquisition behavior |
| Jetson crash | Raspberry Pi fail-safe triggers surfacing |
| Raspberry Pi crash | Cube Orange continues last command, then holds position |
| Single thruster failure | Actuator mixer re-allocates forces to remaining 7 thrusters |
| Camera fogging | Humidity sensor triggers heaters; surface-and-wipe behavior if severe |
| Current disturbance | Disturbance observer provides feedforward compensation |

---

## Validation Tests

The following tests are performed at each stage of integration:

| Test | Pass Criteria |
|------|--------------|
| Visual dropout (10s) | Position error < 0.5m after recovery |
| Current rejection (artificial current) | Disturbance observer converges within 5s |
| Long run (5min, square path) | End-position error < 1.0m |
| Fogging simulation (cold water) | Heaters activate within 10s of humidity spike |
| Jetson hard reset | Vehicle surfaces within 5s |
| Single thruster kill | Vehicle maintains heading within ±5° |

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 2026-08-22 | 2.0 | Major revision: Added Dead Reckoning Fallback, VO Quality Monitor, Current Disturbance Observer, fogging prevention, pre-mission calibration, and re-acquisition behavior. Removed dependency on DVL. |
