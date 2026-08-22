# RoboSub 2026/2027 - Development Task Breakdown (Final)

## Development Philosophy

**Phase 0: ROV First** → Get a manually-controlled vehicle in the water within 8 weeks. This validates mechanical/electrical systems before adding autonomy.

**Phase 1: AUV Second** → Layer autonomy on top of proven hardware. Autonomy is useless if the vehicle can't hold depth or respond to joystick commands.

This is a 24-week total plan at breakneck speed (and assumes 6 devs); it ends by February 2027, but realistically, we'll be finished by like May.

---


## Phase 0: ROV Foundation (Weeks 1-8)

### 0.0 Essential Design Documents (Weeks 1-2)

**Goal:** Create minimum viable design documents to guide development.

- [ ] **DOC-01** Create System Architecture Document: block diagram, hardware-software mapping, communication protocols, power budget
- [ ] **DOC-02** Create Project Management Document: team roles, meeting cadence, decision-making process
- [ ] **DOC-03** Create Safety & Risk Document: emergency procedures, kill switch, battery safety

**Acceptance Criteria:**
- [ ] System Architecture diagram is complete and team has reviewed it
- [ ] Team roles and meeting schedule are documented
- [ ] Safety procedures are documented and understood by all team members

---

### 0.4 Wiring & Power Diagram (Weeks 3-4)

**Goal:** Document all physical connections before assembly.

- [ ] **WIR-01** Create wiring diagram: all UART, I2C, USB, PWM, power connections
- [ ] **WIR-02** Create power distribution diagram: battery → regulators → components
- [ ] **WIR-03** Document pinout assignments for Raspberry Pi, Cube Orange, PCB

**Acceptance Criteria:**
- [ ] Wiring diagram is complete and matches the physical build
- [ ] Power budget is validated (total draw < battery capacity)
- [ ] Pinout assignments are documented and version-controlled

---

### 0.6 Test Plan (Weeks 5-6)

**Goal:** Define testing strategy before pool deployment.

- [ ] **TEST-01** Define test progression: bench → simulation → pool → competition
- [ ] **TEST-02** Define success criteria for each test phase
- [ ] **TEST-03** Document data collection requirements for each test

**Acceptance Criteria:**
- [ ] Test progression is documented and team has reviewed it
- [ ] Success criteria are quantified (e.g., depth error < 0.3m)
- [ ] Data collection plan is documented

---

### 0.1 Requirements & Task Analysis
**Goal:** Document what the vehicle must do and how components will communicate.

- [ ] **RT-01** Document competition tasks (gate, buoys, torpedoes, etc.) with success criteria and point values
- [ ] **RT-02** Define vehicle operational envelope: max depth (5m), max speed (1.5 m/s), mission duration (15 min)
- [ ] **RT-03** Create interface control document (ICD) between all compute nodes: UART baud rates, ROS 2 topics, MAVLink message types
- [ ] **RT-04** Document failure modes and recovery actions for each system state (see ESS ENN Guide - Fault Detection)
- [ ] **RT-05** **[DEFER]** Autonomy-specific requirements (waypoint accuracy, object detection thresholds) → defer until Phase 1
- [ ] **RT-06** **[DEFER]** Detailed behavior tree design → defer until Phase 1, use simple state machine for ROV

**Acceptance Criteria:**
- [ ] All competition tasks are listed with success criteria and point values
- [ ] Operational envelope (depth, speed, duration) is documented and signed off by team
- [ ] ICD specifies: UART baud rate (115200), ROS 2 topics with message types, MAVLink message IDs
- [ ] Failure modes document covers at least: Jetson crash, Pi crash, thruster failure, leak detection, depth sensor failure

---

### 0.2 Repository Setup
**Goal:** Establish version control and basic project structure.

- [ ] **REPO-01** Create monorepo with three subdirectories: `firmware/` (Cube Orange), `companion/` (Raspberry Pi), `autonomy/` (Jetson)
- [ ] **REPO-02** Set up Git with `main` branch (stable) and `dev` branch (active development)
- [ ] **REPO-03** Create `README.md` with setup instructions, wiring diagram, and quick-start guide
- [ ] **REPO-04** Add `requirements.txt` and `setup.sh` for dependency installation on each platform
- [ ] **REPO-05** **[DEFER]** CI/CD (GitHub Actions) → add after ROV is working
- [ ] **REPO-06** **[DEFER]** Docker containers → defer until Phase 1
- [ ] **REPO-07** **[DEFER]** Doxygen/Sphinx documentation → defer until Phase 1

**Acceptance Criteria:**
- [ ] All team members can clone the repository and build on their workstations
- [ ] `README.md` contains: wiring diagram, quick-start commands, and troubleshooting section
- [ ] `setup.sh` installs all dependencies on a clean Jetson/Raspberry Pi installation without errors

---

### 0.3 Development Environment
**Goal:** Set up native development environments on all platforms.

- [ ] **ENV-01** Set up ROS 2 Jazzy on Jetson Orin Nano (Ubuntu 24.04) with `ros-base` installation
- [ ] **ENV-02** Install MAVROS and MAVLink packages on Jetson: `sudo apt install ros-jazzy-mavros ros-jazzy-mavros-extras`
- [ ] **ENV-03** Set up ROS 2 Jazzy on development workstation (Ubuntu 24.04) for simulation/testing
- [ ] **ENV-04** Install Python dependencies: `opencv-python`, `numpy`, `scipy`, `pyyaml`, `pyserial`
- [ ] **ENV-05** Create `setup.bash` script to source ROS 2 and workspace environment variables

**Acceptance Criteria:**
- [ ] `ros2 --version` returns "Jazzy" on Jetson and workstation
- [ ] MAVROS nodes can communicate over simulated UART
- [ ] `setup.bash` sources correctly and all Python imports succeed

---

### 0.4 Simulation Environment
**Goal:** Validate software stack in a virtual environment before hardware deployment.

- [ ] **SIM-01** Install and configure Blue Robotics `ardusub-gazebo` simulator with vehicle model
- [ ] **SIM-02** Create ROS 2 Jazzy bridge (`ardusub_gazebo` ↔ ROS 2 Jazzy) using `ros_gz_bridge`
- [ ] **SIM-03** Implement joystick teleoperation in simulation (manual ROV mode)
- [ ] **SIM-04** Add pool environment model: 15m × 10m × 5m with visual landmarks (AprilTags)
- [ ] **SIM-05** **[DEFER]** Add currents/waves to simulation → defer until Phase 1
- [ ] **SIM-06** **[DEFER]** Mission replay capability → defer until Phase 1

**Acceptance Criteria:**
- [ ] Simulated vehicle moves in all 6 DOF under joystick control
- [ ] Depth hold PID works in simulation (depth error < 0.1m within 5s)
- [ ] AprilTags are visible in simulated camera stream

---

### 0.5 Hardware Abstraction Layer (Cube Orange)
**Goal:** Configure the flight controller firmware and validate actuator/sensor integration.

- [ ] **HAL-01** Flash ArduSub 4.5+ firmware to Cube Orange with default Blue Robotics parameters
- [ ] **HAL-02** Configure actuator mixer for 8-thruster vectored configuration (from vehicle geometry)
- [ ] **HAL-03** Calibrate ESCs and verify thruster spin direction in water (or bucket test)
- [ ] **HAL-04** Configure Bar30 depth sensor on I2C bus with correct addressing
- [ ] **HAL-05** Set up MAVLink 2.0 on UART (TELEM1 port) for communication with Raspberry Pi
- [ ] **HAL-06** Verify PID tuning in simulation, export parameters for hardware
- [ ] **HAL-07** **[DEFER]** Tune PID gains on actual vehicle → defer after ROV sea trials
- [ ] **HAL-08** **[DEFER]** EKF3 external vision configuration → defer until Phase 1
- [ ] **HAL-09** Conduct benchtop thrust-vs-PWM characterization for all 8 thrusters (critical for dead reckoning model)
- [ ] **HAL-10** Perform and document pre-mission IMU bias calibration routine

**Acceptance Criteria:**
- [ ] ArduSub firmware boots successfully; Mission Planner/QGroundControl connects via USB
- [ ] All 8 thrusters spin in correct direction (matching mixer configuration)
- [ ] Bar30 depth sensor reads within 0.1m of actual depth in a bucket test
- [ ] MAVLink heartbeat visible on UART (115200 baud)
- [ ] Thrust-vs-PWM curve documented for all 8 thrusters (min 10 data points each)
- [ ] IMU bias values recorded for accelerometer and gyroscope (stationary for 60s)

---

### 0.6 Control Layer (Raspberry Pi - ROV Mode)
**Goal:** Set up the companion computer for MAVLink routing, I/O, and manual control.

- [ ] **CTRL-01** Install BlueOS on Raspberry Pi 4 with minimal extensions (MAVLink Router, Web UI)
- [ ] **CTRL-02** Implement MAVLink router: Jetson (UART) ↔ Cube Orange (UART) with heartbeat monitoring
- [ ] **CTRL-03** Write I/O Manager node (Python): poll leak/temp/humidity sensors via GPIO/I2C
- [ ] **CTRL-04** Implement joystick passthrough: USB joystick → MAVLink `MANUAL_CONTROL` messages
- [ ] **CTRL-05** Add basic fail-safe: if Raspberry Pi loses Jetson heartbeat for >3s, surface command
- [ ] **CTRL-06** **[DEFER]** Current Disturbance Observer → defer until Phase 1
- [ ] **CTRL-07** **[DEFER]** Data logging with timestamps → defer until Phase 1

**Acceptance Criteria:**
- [ ] BlueOS web interface accessible from surface laptop
- [ ] MAVLink messages routed correctly between Jetson UART and Cube Orange UART
- [ ] Leak/temp/humidity sensors return valid readings via I2C/GPIO
- [ ] USB joystick controls simulated vehicle in `ardusub-gazebo`
- [ ] Fail-safe triggers surface command when Jetson heartbeat stops (tested on bench)

---

### 0.7 Operator Interface (ROV Dashboard)
**Goal:** Provide the pilot with real-time telemetry and video feedback.

- [ ] **UI-01** Set up QGroundControl or BlueOS web interface on surface laptop
- [ ] **UI-02** Configure dashboard to display: primary camera feed (OAK-D), depth, heading, thruster status, leak detection
- [ ] **UI-03** Add telemetry overlays on video feed (depth, heading, target waypoints)
- [ ] **UI-04** Test interface in simulation before pool deployment

**Acceptance Criteria:**
- [ ] Dashboard displays depth, heading, and thruster status in real-time (update rate > 10Hz)
- [ ] Video feed from simulated camera displays with < 500ms latency
- [ ] Overlay shows depth and heading on video feed
- [ ] Interface is usable on a laptop with a single screen (no switching between windows)

---

### 0.8 Hardware Bring-Up (Physical Vehicle)
**Goal:** Assemble and validate the physical vehicle structure, waterproofing, and power distribution.

- [ ] **HW-01** Assemble pressure vessel: acrylic dome, O-rings, penetrators, bulkheads
- [ ] **HW-02** Test waterproofing: dunk test in freshwater pool for 1 hour, check for leaks
- [ ] **HW-03** Mount thrusters and servos to frame; verify clearance and wiring routing
- [ ] **HW-04** Assemble custom PCB and verify power distribution: 5V, 3.3V, servo PWM, I2C pull-ups
- [ ] **HW-05** Install batteries and verify voltage/current monitoring (if available)
- [ ] **HW-06** Dry test: power on all electronics (thrusters not submerged), verify no shorts
- [ ] **HW-07** Wet test: submerge vehicle, verify buoyancy (slightly positive), trim using weights
- [ ] **HW-08** **[DEFER]** Install cameras and lenses → defer until Phase 1 (ROV uses only depth + IMU)
- [ ] **HW-09** Perform depth sensor calibration: compare Bar30 readings against known depths

**Acceptance Criteria:**
- [ ] Pressure vessel survives 1-hour submerged dunk test with no visible water ingress
- [ ] All 8 thrusters and 3 servos receive power and respond to PWM signals
- [ ] PCB voltage rails: 5V ± 0.25V, 3.3V ± 0.1V under load
- [ ] Vehicle floats with slightly positive buoyancy (slowly rises when released from 1m depth)
- [ ] Bar30 depth readings match known depths within 0.05m at 0.5m, 1.0m, and 2.0m

---

### 0.9 ROV Integration & Sea Trials (Without Jetson)
**Goal:** Validate manual control with Raspberry Pi + Cube Orange only.

- [ ] **INT-01** Connect all layers: joystick → Raspberry Pi → Cube Orange → thrusters
- [ ] **INT-02** Perform pool test: manual control with joystick, verify 6-DOF motion (surge, sway, heave, yaw)

**Acceptance Criteria (Pre-Jetson Integration):**
- [ ] Vehicle responds to joystick commands within 500ms
- [ ] All 6 DOF (surge, sway, heave, roll, pitch, yaw) are controllable
- [ ] Vehicle holds depth within ±0.3m when depth hold engaged
- [ ] No unexpected behavior (thruster oscillations, heading drift, etc.)

---

### 0.10 Jetson Integration (Insert After INT-02)
**Goal:** Add the Jetson in a passive role for data logging and fail-safe validation.

- [ ] **JET-01** Install ROS 2 Jazzy on Jetson with MAVROS and DepthAI drivers
- [ ] **JET-02** Implement heartbeat publisher node: sends `jetson_heartbeat` to Raspberry Pi at 1Hz
- [ ] **JET-03** Implement passive data logger: subscribes to `/imu`, `/depth`, `/thruster_telemetry`, logs to CSV/ROS bag
- [ ] **JET-04** Verify OAK-D and ArduCam streams over USB 3.0; check for dropped frames
- [ ] **JET-05** (Optional) Run visual odometry pipeline in passive mode — no control outputs
- [ ] **JET-06** Test fail-safe on bench: disconnect Jetson USB/UART → Raspberry Pi commands surface
- [ ] **JET-07** Verify Jetson power consumption and thermal performance during 30-minute bench test

**Acceptance Criteria (Before Returning to Pool):**
- [ ] Heartbeat publisher runs continuously and is received by Raspberry Pi
- [ ] Data logger writes valid CSV files with timestamps for IMU, depth, thruster telemetry
- [ ] OAK-D and ArduCam streams are visible in `rviz2` or `rqt_image_view` with < 5% dropped frames
- [ ] Fail-safe triggers surface command within 3s of Jetson heartbeat loss on bench
- [ ] Jetson temperature remains < 65°C during 30-minute bench test (with cooling fan)

---

### 0.11 ROV Integration & Sea Trials (With Jetson)
**Goal:** Validate manual control with Jetson integrated (passive mode).

- [ ] **INT-03** Repeat INT-02 with Jetson powered on → verify no change in manual control behavior
- [ ] **INT-04** Test fail-safe: disconnect Jetson (or simulate crash) → vehicle surfaces
- [ ] **INT-05** Leak test: submerge vehicle to 3m for 30 minutes, monitor leak sensors (with Jetson logging)
- [ ] **INT-06** Endurance test: 20-minute continuous manual operation with Jetson logging data
- [ ] **INT-07** Add telemetry dashboard (BlueOS web interface) with Jetson data integration

**Acceptance Criteria (Final Phase 0 Validation):**
- [ ] Manual control behavior is identical with and without Jetson powered on
- [ ] Fail-safe surfaces vehicle within 5s of Jetson heartbeat loss (pool test, not just bench)
- [ ] Leak sensors detect water ingress within 5s of simulated leak
- [ ] Vehicle operates continuously for 20 minutes with no thermal shutdowns or voltage drops
- [ ] Jetson logs complete dataset: IMU, depth, thruster telemetry, camera streams for entire run
- [ ] Dashboard displays all telemetry with < 1s latency

---

## Phase 0 Completion Milestone

### ✅ Phase 0 Acceptance Criteria (All Must Be Met)

| # | Criterion | Verification Method |
|---|-----------|---------------------|
| 1 | Vehicle moves reliably in all 6 DOF under manual control | Pool test with joystick; video evidence |
| 2 | Depth hold maintains depth within ±0.3m | Pool test; log review |
| 3 | Fail-safe surfaces vehicle within 5s of Jetson heartbeat loss | Pool test with intentional disconnect |
| 4 | All sensors (Bar30, IMU, leak, temp, humidity) return valid data | Log review |
| 5 | All 8 thrusters characterized (PWM → thrust curve) | Benchtop test documentation |
| 6 | OAK-D and ArduCam stream without frame drops > 5% | Log review |
| 7 | Vehicle passes 30-minute submerged leak test | Visual inspection; leak sensor logs |
| 8 | Jetson logs complete dataset for a 20-minute run | Log file verification |
| 9 | All documentation (ICD, wiring diagram, failure modes) is complete | Document review |
| 10 | Team can transition to Phase 1 (AUV development) with confidence | Team consensus |

**Phase 0 Duration:** Weeks 1-8  
**Phase 0 Go/No-Go Decision:** All 10 criteria must be marked ✅ before proceeding to Phase 1.

---

## Phase 1: AUV Development (Weeks 8-20)

### 1.0 Pre-Phase 1 Planning (Week 8-9)

**Goal:** Create design documents for autonomy and perception before implementation.

- [ ] **PLAN-01** Create State Machine Design Document: states, transitions, failure modes
- [ ] **PLAN-02** Create CV Pipeline Design Document: VO, quality monitor, sensor fusion
- [ ] **PLAN-03** Create Autonomy Design Document: path planning, motion control, dead reckoning

**Acceptance Criteria:**
- [ ] State machine diagram is complete and reviewed
- [ ] CV pipeline design is documented and reviewed
- [ ] Autonomy design is consistent with state machine and CV pipeline
- [ ] Team can begin Phase 1 implementation with clear direction

---

### 1.1 Development Environment (Phase 1 Additions)
**Goal:** Add containerization and CI/CD for reproducible builds.

- [ ] **ENV-AUV-01** Create Dockerfiles for reproducible builds on Jetson (ROS 2 Jazzy + CUDA/OpenCV)
- [ ] **ENV-AUV-02** Set up `docker-compose.yml` for Jetson with camera passthrough and volume mounts
- [ ] **ENV-AUV-03** Set up CI/CD: GitHub Actions for ROS 2 build verification on push to `main`/`dev`
- [ ] **ENV-AUV-04** Add pre-commit hooks for linting (Python `black`, `ruff`, C++ `clang-format`)
- [ ] **ENV-AUV-05** Create documentation pipeline: Doxygen for C++, Sphinx for Python, auto-build on PR merge
- [ ] **ENV-AUV-06** Write `Dockerfile.ros` with ROS 2 Jazzy + DepthAI + OpenCV + CUDA dependencies
- [ ] **ENV-AUV-07** Write `Dockerfile.pi` with BlueOS extension development environment

**Acceptance Criteria:**
- [ ] Docker images build successfully on Jetson and workstation
- [ ] CI/CD pipeline passes on pull requests (build + lint)
- [ ] Documentation auto-generates and is hosted (e.g., GitHub Pages)
- [ ] All team members can use the same Docker environment for development

---


**Acceptance Criteria:**
- [ ] State diagram is complete and reviewed
- [ ] All transitions have clear conditions (thresholds, events)
- [ ] ROS 2 interface definitions are in version control
- [ ] Team has approved the design
- [ ] Simulation test cases cover all states and transitions

---

### 1.2 Perception Layer
**Goal:** Implement visual odometry, sensor fusion, and quality monitoring.

- [ ] **PER-01** Install OAK-D Lite drivers (`depthai-ros`) on Jetson; verify camera streaming at 30Hz
- [ ] **PER-02** Install ArduCam drivers; verify global shutter streaming at 60Hz
- [ ] **PER-03** Implement ORB-SLAM3 (stereo mode) with ROS 2 Jazzy wrapper; validate against simulation
- [ ] **PER-04** Implement KLT optical flow on ArduCam as fallback odometry
- [ ] **PER-05** Create VO Quality Monitor node: feature count, disparity variance, tracking stability → publish `vo_quality`
- [ ] **PER-06** Implement sensor fusion: EKF on Jetson fusing VO + IMU + depth + compass
- [ ] **PER-07** Add fogging prevention: humidity sensor triggers heaters; publish status
- [ ] **PER-08** **[DEFER]** YOLO/neural network for object detection (gate/buoys/torpedoes) → defer until Phase 2
- [ ] **PER-09** **[DEFER]** Depth map generation (OAK-D disparity) → defer until Phase 2
- [ ] **PER-10** Document camera lens port optical properties under pressure (calibration reference)

**Acceptance Criteria:**
- [ ] OAK-D publishes stereo images at 30Hz; ArduCam publishes at 60Hz
- [ ] ORB-SLAM3 produces position estimates with drift < 5% of distance traveled (simulation)
- [ ] KLT optical flow produces velocity estimates when ORB-SLAM3 fails (feature count < 100)
- [ ] `vo_quality` metric correlates with position error (verified in simulation)
- [ ] EKF fuses all sensors and produces a smooth state estimate (< 10ms latency)
- [ ] Humidity > 70% triggers heater activation (bench test)

---

### 1.3 Autonomy Layer (Core)
**Goal:** Implement mission planning, path planning, motion control, and dead reckoning.

- [ ] **AUTO-01** Implement ROS 2 node for Mission Planner: simple state machine with 5 states (IDLE, NAVIGATE, TASK, RECOVER, ABORT)
- [ ] **AUTO-02** Implement Path Planner: A* or RRT in 2D (horizontal plane) with depth waypoints
- [ ] **AUTO-03** Implement Motion Controller: PID on position error → velocity commands (surge, sway, heave, yaw)
- [ ] **AUTO-04** Implement Dead Reckoning Fallback: integrate IMU acceleration + thruster telemetry (from HAL-09) when `vo_quality < 0.3`
- [ ] **AUTO-05** Implement Re-Acquisition Behavior: spiral search when position uncertainty > 0.5m
- [ ] **AUTO-06** Add EKF adaptive covariance: inflate when `vo_quality` drops
- [ ] **AUTO-07** Add real-time logging: state estimates, commands, `vo_quality`, uncertainty
- [ ] **AUTO-08** **[DEFER]** Behavior tree implementation (state machine is sufficient for Phase 1) → defer until Phase 2
- [ ] **AUTO-09** **[DEFER]** Obstacle avoidance → defer until Phase 2
- [ ] **AUTO-10** Design lightweight acoustic status protocol (if modem acquired) for emergency communications

**Acceptance Criteria:**
- [ ] State machine transitions correctly through IDLE → NAVIGATE → TASK → IDLE (simulation)
- [ ] Path Planner finds a path from start to goal within 1s (grid size: 20x20m)
- [ ] Motion Controller tracks waypoints with position error < 0.3m (simulation, no currents)
- [ ] Dead Reckoning Fallback holds position within 0.5m for 10s during visual dropout (pool test)
- [ ] Re-Acquisition Behavior recovers visual tracking within 30s of dropout
- [ ] EKF covariance adapts correctly based on `vo_quality` (verified in simulation)

---

### 1.4 Control Layer (AUV Enhancements)
**Goal:** Add current disturbance observer and enhanced fail-safe logic.

- [ ] **CTRL-AUV-01** Implement Current Disturbance Observer on Raspberry Pi: compare desired vs. achieved velocity
- [ ] **CTRL-AUV-02** Add feedforward compensation to velocity commands before MAVLink forwarding
- [ ] **CTRL-AUV-03** Extend fail-safe: if `vo_quality < 0.1` for >10s, trigger re-acquisition (not surfacing)
- [ ] **CTRL-AUV-04** Add thruster telemetry logging: RPM, current draw, PWM values → for dead reckoning
- [ ] **CTRL-AUV-05** **[DEFER]** Web dashboard for real-time state visualization → defer until Phase 2
- [ ] **CTRL-AUV-06** **[DEFER]** Data logging to onboard SSD (Jetson) → defer until Phase 2

**Acceptance Criteria:**
- [ ] Current Disturbance Observer converges within 5s of artificial current introduction (pool test)
- [ ] Feedforward compensation reduces position error by > 50% under current (pool test)
- [ ] Fail-safe triggers re-acquisition (not surfacing) when `vo_quality < 0.1` for >10s
- [ ] Thruster telemetry logs match expected values (RPM vs. PWM from HAL-09)

---

### 1.5 HAL Enhancements (Cube Orange)
**Goal:** Configure EKF3 to accept external vision data conditionally.

- [ ] **HAL-AUV-01** Configure EKF3 to accept external vision via `VISION_POSITION_ESTIMATE` MAVLink messages
- [ ] **HAL-AUV-02** Implement conditional fusion: only accept external vision if `vo_quality > 0.5`
- [ ] **HAL-AUV-03** Enable `VISION_ESTIMATOR` in ArduSub parameters
- [ ] **HAL-AUV-04** Test vision fusion in simulation (ArduSub Gazebo + ROS 2 bridge)
- [ ] **HAL-AUV-05** **[DEFER]** Tune EKF3 covariance parameters on real vehicle → defer after pool tests

**Acceptance Criteria:**
- [ ] EKF3 accepts `VISION_POSITION_ESTIMATE` messages and fuses them with IMU data (simulation)
- [ ] Conditional fusion drops external vision when `vo_quality < 0.5` (simulation)
- [ ] Vehicle tracks position with vision fusion enabled (position error < 0.5m, simulation)

---

### 1.6 Perception Hardware Installation
**Goal:** Mount and verify cameras on the physical vehicle.

- [ ] **HW-AUV-01** Mount OAK-D Lite in acrylic dome with anti-fog coating; verify waterproofing
- [ ] **HW-AUV-02** Mount ArduCam Global Shutter in separate housing; verify waterproofing
- [ ] **HW-AUV-03** Install humidity sensor inside dome; route I2C to Raspberry Pi
- [ ] **HW-AUV-04** Configure camera exposures: OAK-D (auto-exposure), ArduCam (fixed exposure for optical flow)
- [ ] **HW-AUV-05** Verify camera streaming over USB 3.0 to Jetson (no dropped frames)

**Acceptance Criteria:**
- [ ] Both cameras are waterproof after 30-minute submerged test
- [ ] OAK-D streams at 30Hz, ArduCam at 60Hz with < 5% dropped frames
- [ ] Humidity sensor reads correctly inside dome
- [ ] Camera exposures produce usable images in pool lighting conditions (manual check)

---

### 1.7 AUV Integration & Pool Tests
**Goal:** Validate full autonomy stack in the pool.

- [ ] **INT-AUV-01** Integration test: Perception → Autonomy → Control → HAL (all layers active)
- [ ] **INT-AUV-02** Pool test: navigate to a visual target (AprilTag) using visual odometry
- [ ] **INT-AUV-03** Pool test: visual dropout for 10s → verify dead reckoning hold within 0.5m
- [ ] **INT-AUV-04** Pool test: re-acquisition behavior (cover camera, then uncover → vehicle resumes)
- [ ] **INT-AUV-05** Pool test: station-keeping in artificial current (use trolling motor) → disturbance observer convergence <5s
- [ ] **INT-AUV-06** Pool test: complete a simple mission (gate → buoy) in simulation, then on real vehicle
- [ ] **INT-AUV-07** **[DEFER]** Full competition run (all tasks) → defer until Phase 2

**Acceptance Criteria:**
- [ ] Vehicle navigates to AprilTag within 0.5m (pool test)
- [ ] Dead reckoning holds position within 0.5m for 10s during visual dropout (pool test)
- [ ] Re-acquisition recovers visual tracking within 30s (pool test)
- [ ] Station-keeping maintains position within 0.3m in artificial current (pool test)
- [ ] Simple mission (gate → buoy) completes successfully in 3/5 attempts (pool test)

---

### 1.8 Pre-Competition Validation
**Goal:** Document parameters and validate mission performance.

- [ ] **VAL-01** Document all parameters: PID gains, EKF covariances, VO thresholds, disturbance observer gains
- [ ] **VAL-02** Create mission script for competition: define waypoints and task sequence
- [ ] **VAL-03** Run 5 complete mission simulations with different initial conditions
- [ ] **VAL-04** Perform 3 full pool test runs (with judges watching) to practice troubleshooting
- [ ] **VAL-05** **[DEFER]** Endurance test: 15-minute continuous operation → defer until after competition qualification
- [ ] **VAL-06** **[DEFER]** Night/testing conditions (low-light, turbid water) → defer until Phase 2

**Acceptance Criteria:**
- [ ] All parameters documented in a single YAML file per platform
- [ ] Mission script runs without errors in simulation
- [ ] 5/5 simulation runs complete successfully
- [ ] 3/3 pool test runs complete successfully (no aborts)

---

## Phase 1 Completion Milestone

### ✅ Phase 1 Acceptance Criteria (All Must Be Met)

| # | Criterion | Verification Method |
|---|-----------|---------------------|
| 1 | Visual odometry works reliably in pool conditions (min 30 minutes continuous) | Log review; position drift < 5% of distance traveled |
| 2 | Dead reckoning holds position within 0.5m for 10s during visual dropout | Pool test with intentional camera cover |
| 3 | Re-acquisition behavior recovers within 30s of visual dropout | Pool test |
| 4 | Current disturbance observer converges within 5s under artificial current | Pool test with trolling motor |
| 5 | Simple mission (gate → buoy) succeeds in 3/5 pool attempts | Pool test log |
| 6 | All parameters are documented and version-controlled | File review |
| 7 | Docker/CI/CD pipeline passes on `main` branch | CI/CD status check |
| 8 | Team can transition to Phase 2 (competition-specific tasks) with confidence | Team consensus |

**Phase 1 Duration:** Weeks 9-20  
**Phase 1 Go/No-Go Decision:** All 8 criteria must be marked ✅ before proceeding to Phase 2.

---

## Phase 2: Competition-Ready (Weeks 21-24)

### 2.1 ML & Perception Upgrades
**Goal:** Add object detection and loop closure for robust competition performance.

- [ ] **ML-01** Train/YOLO model for gate detection (red/green buoys, torpedo launchers, etc.)
- [ ] **ML-02** Implement object detection inference on OAK-D's VPU (onboard neural accelerator)
- [ ] **ML-03** Fuse object detections with visual odometry → target-relative positioning
- [ ] **ML-04** **[DEFER]** Underwater image enhancement (auto white balance, contrast correction) → if time permits
- [ ] **ML-05** Investigate loop closure detection for visual SLAM (ORB-SLAM3 map persistence or Bag-of-Words)

**Acceptance Criteria:**
- [ ] YOLO model detects gates/buoys/torpedoes with > 90% precision in pool test
- [ ] Object detection runs at > 10Hz on OAK-D VPU
- [ ] Fused target-relative positioning has error < 0.3m at 5m range
- [ ] Loop closure improves position estimate (drift reduction > 30% in 20m mission)

---

### 2.2 Advanced Autonomy
**Goal:** Implement behavior trees and task-specific behaviors.

- [ ] **AUTO-ADV-01** Convert state machine to Behavior Tree (using `behaviortree_cpp_v3`)
- [ ] **AUTO-ADV-02** Implement task-specific behaviors: gate navigation, buoy traversal, torpedo firing, marker drop
- [ ] **AUTO-ADV-03** Add sequential task planning: complete all tasks in optimal order
- [ ] **AUTO-ADV-04** Add obstacle avoidance (static: pool walls; dynamic: other AUVs)
- [ ] **AUTO-ADV-05** **[DEFER]** Multi-vehicle coordination → not applicable (only one AUV)

**Acceptance Criteria:**
- [ ] Behavior Tree executes all tasks without human intervention
- [ ] Task-specific behaviors complete successfully (gate: navigate center; buoy: traverse in sequence; torpedo: fire within 0.5m; marker: drop at waypoint)
- [ ] Obstacle avoidance prevents collisions with pool walls (simulation + pool test)
- [ ] Sequential planning completes all tasks in optimal order (highest point value first)

---

### 2.3 Competition-Specific Tasks
**Goal:** Implement and validate each competition task.

- [ ] **TASK-01** Implement gate detection: identify gate center from OAK-D RGB + depth
- [ ] **TASK-02** Implement buoy traversal: detect colored buoys, navigate through them in sequence
- [ ] **TASK-03** Implement torpedo launcher: actuate servo based on object detection
- [ ] **TASK-04** Implement marker drop: actuate servo to release marker at GPS waypoint
- [ ] **TASK-05** Implement acoustic pinger tracking (if required) → needs external hardware

**Acceptance Criteria:**
- [ ] Gate detection works in 3/5 attempts (pool test)
- [ ] Buoy traversal works in 3/5 attempts (pool test)
- [ ] Torpedo fires within 0.5m of target (pool test)
- [ ] Marker drops within 0.5m of waypoint (pool test)

---

### 2.4 Finals Preparation
**Goal:** Prepare for competition day.

- [ ] **FINAL-01** Create competition-specific mission plan (customize for venue)
- [ ] **FINAL-02** Dry-run at competition pool (if allowed) or equivalent local pool
- [ ] **FINAL-03** Create troubleshooting checklist: common failures and recovery procedures
- [ ] **FINAL-04** Create recovery team protocol: physical retrieval, data download, battery swap
- [ ] **FINAL-05** **[DEFER]** Post-competition data analysis → after competition

**Acceptance Criteria:**
- [ ] Mission plan is documented and reviewed by all team members
- [ ] Dry-run completes all tasks in competition order (pool test)
- [ ] Troubleshooting checklist covers: Jetson crash, Pi crash, thruster failure, leak, camera failure
- [ ] Recovery protocol is rehearsed and timed (< 5 minutes for full vehicle swap)

---

## Phase 2 Completion Milestone

### ✅ Phase 2 Acceptance Criteria (All Must Be Met)

| # | Criterion | Verification Method |
|---|-----------|---------------------|
| 1 | All competition tasks (gate, buoy, torpedo, marker) work in 3/5 pool attempts | Pool test log |
| 2 | Behavior Tree executes full mission without human intervention | Pool test |
| 3 | Object detection precision > 90% | Pool test log review |
| 4 | Loop closure improves position estimate (drift reduction > 30%) | Pool test log review |
| 5 | Obstacle avoidance prevents collisions (simulation + pool test) | Log review; no collisions observed |
| 6 | Mission plan is documented and rehearsed | Document review; dry-run success |
| 7 | Troubleshooting checklist and recovery protocol are rehearsed | Team rehearsal |
| 8 | Vehicle is fully charged, calibrated, and ready for competition | Pre-competition checklist |

**Phase 2 Duration:** Weeks 21-24  
**Competition Readiness:** All 8 criteria must be marked ✅ before departing for competition.

---

## Overall Project Milestones

| Milestone | Week | Acceptance Criteria |
|-----------|------|---------------------|
| **Phase 0 Complete** | 8 | All 10 Phase 0 criteria met → Reliable manual ROV |
| **Phase 1 Complete** | 20 | All 8 Phase 1 criteria met → AUV with core autonomy |
| **Phase 2 Complete** | 24 | All 8 Phase 2 criteria met → Competition-ready AUV |
| **Competition** | 24-25 | All tasks executed successfully; vehicle retrieved safely |

---

## Hardware Limitations & Workarounds

| Hardware Gap | Impact | Workaround / Mitigation |
|--------------|--------|------------------------|
| **No DVL** | No direct velocity measurement; dead reckoning drifts rapidly | Visual odometry (primary) + IMU/thruster telemetry fallback (AUTO-04); VO Quality Monitor (PER-05) |
| **No acoustic positioning** | No absolute position fixes underwater | Use visual landmarks (AprilTags) in pool; design mission to stay in visual range |
| **No sonar** | Cannot perform SLAM in turbid/low-light conditions | Rely on visual SLAM; add loop closure (ML-05); document visibility limitations |
| **Navigation-grade IMU not available** | Higher drift than dedicated navigation IMU | Calibrate IMU bias (HAL-10); fuse with visual odometry aggressively; limit mission duration |
| **No high-bandwidth tether** | Cannot stream high-res video to surface | Use onboard recording; stream compressed/low-res for pilot feedback; rely on BlueOS web interface |
| **No acoustic modem** | No long-range communication | Not required for pool competition; design missions to stay within visual/operator range |

---

## Notes for Team

1. **Industry Alignment:** This plan follows industry best practices: ROV-first validation, distributed architecture, behavior-based autonomy, robust fault detection, and comprehensive data logging.
2. **Hardware Realities:** Acknowledge the limitations (no DVL, no sonar, no acoustic modem). Design missions that play to the vehicle's strengths (visual odometry in clear water, short-range operations in controlled pool environments).
3. **ROV is the Foundation:** If manual control fails, autonomy cannot save you. Invest time in getting the ROV phase right.
4. **Calibration is Critical:** The new tasks (HAL-09, HAL-10, HW-09) are essential for dead reckoning and sensor accuracy. Don't skip them.
5. **Simulation is Not Reality:** Get the vehicle in the pool as early as possible (Week 6 target).
6. **Document Everything:** Parameters, wiring diagrams, calibration steps, failure modes. You will thank yourself during competition troubleshooting.
7. **Keep It Simple:** Resist feature creep. Your competition score depends on reliable execution of basic tasks, not fancy features.
8. **Phase Gates Are Mandatory:** Do not proceed to the next phase until all acceptance criteria for the current phase are met. This prevents compounding issues.