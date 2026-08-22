# RoboSub 2026/2027 - Development Task Breakdown (Final)

## Development Philosophy

**Phase 0: ROV First** → Get a manually-controlled vehicle in the water within 8 weeks. This validates mechanical/electrical systems before adding autonomy.

**Phase 1: AUV Second** → Layer autonomy on top of proven hardware. Autonomy is useless if the vehicle can't hold depth or respond to joystick commands.

---

## Phase 0: ROV Foundation (Weeks 1-8)

### 0.1 Requirements & Task Analysis
- [ ] **RT-01** Document competition tasks (gate, buoys, torpedoes, etc.) with success criteria and point values
- [ ] **RT-02** Define vehicle operational envelope: max depth (5m), max speed (1.5 m/s), mission duration (15 min)
- [ ] **RT-03** Create interface control document (ICD) between all compute nodes: UART baud rates, ROS 2 topics, MAVLink message types
- [ ] **RT-04** Document failure modes and recovery actions for each system state 
- [ ] **RT-05** **[DEFER]** Autonomy-specific requirements (waypoint accuracy, object detection thresholds) → defer until Phase 1
- [ ] **RT-06** **[DEFER]** Detailed behavior tree design → defer until Phase 1, use simple state machine for ROV

### 0.2 Repository Setup (Simplified)
- [ ] **REPO-01** Create monorepo with three subdirectories: `firmware/` (Cube Orange), `companion/` (Raspberry Pi), `autonomy/` (Jetson)
- [ ] **REPO-02** Set up Git with `main` branch (stable) and `dev` branch (active development)
- [ ] **REPO-03** Create `README.md` with setup instructions, wiring diagram, and quick-start guide
- [ ] **REPO-04** Add `requirements.txt` and `setup.sh` for dependency installation on each platform
- [ ] **REPO-05** **[DEFER]** CI/CD (GitHub Actions) → add after ROV is working
- [ ] **REPO-06** **[DEFER]** Docker containers → defer until Phase 1
- [ ] **REPO-07** **[DEFER]** Doxygen/Sphinx documentation → defer until Phase 1

### 0.3 Development Environment (Native)
- [ ] **ENV-01** Set up ROS 2 Jazzy on Jetson Orin Nano (Ubuntu 24.04) with `ros-base` installation
- [ ] **ENV-02** Install MAVROS and MAVLink packages on Jetson: `sudo apt install ros-jazzy-mavros ros-jazzy-mavros-extras`
- [ ] **ENV-03** Set up ROS 2 Jazzy on development workstation (Ubuntu 24.04) for simulation/testing
- [ ] **ENV-04** Install Python dependencies: `opencv-python`, `numpy`, `scipy`, `pyyaml`, `pyserial`
- [ ] **ENV-05** Create `setup.bash` script to source ROS 2 and workspace environment variables

### 0.4 Simulation Environment
- [ ] **SIM-01** Install and configure Blue Robotics `ardusub-gazebo` simulator with vehicle model
- [ ] **SIM-02** Create ROS 2 Jazzy bridge (`ardusub_gazebo` ↔ ROS 2 Jazzy) using `ros_gz_bridge`
- [ ] **SIM-03** Implement joystick teleoperation in simulation (manual ROV mode)
- [ ] **SIM-04** Add pool environment model: 15m × 10m × 5m with visual landmarks (AprilTags)
- [ ] **SIM-05** **[DEFER]** Add currents/waves to simulation → defer until Phase 1
- [ ] **SIM-06** **[DEFER]** Mission replay capability → defer until Phase 1

### 0.5 Hardware Abstraction Layer (Cube Orange)
- [ ] **HAL-01** Flash ArduSub 4.5+ firmware to Cube Orange with default Blue Robotics parameters
- [ ] **HAL-02** Configure actuator mixer for 8-thruster vectored configuration (from vehicle geometry)
- [ ] **HAL-03** Calibrate ESCs and verify thruster spin direction in water (or bucket test)
- [ ] **HAL-04** Configure Bar30 depth sensor on I2C bus with correct addressing
- [ ] **HAL-05** Set up MAVLink 2.0 on UART (TELEM1 port) for communication with Raspberry Pi
- [ ] **HAL-06** Verify PID tuning in simulation, export parameters for hardware
- [ ] **HAL-07** **[DEFER]** Tune PID gains on actual vehicle → defer after ROV sea trials
- [ ] **HAL-08** **[DEFER]** EKF3 external vision configuration → defer until Phase 1
- [ ] **HAL-09** **[NEW]** Conduct benchtop thrust-vs-PWM characterization for all 8 thrusters (critical for dead reckoning model)
- [ ] **HAL-10** **[NEW]** Perform and document pre-mission IMU bias calibration routine

### 0.6 Control Layer (Raspberry Pi - ROV Mode)
- [ ] **CTRL-01** Install BlueOS on Raspberry Pi 4 with minimal extensions (MAVLink Router, Web UI)
- [ ] **CTRL-02** Implement MAVLink router: Jetson (UART) ↔ Cube Orange (UART) with heartbeat monitoring
- [ ] **CTRL-03** Write I/O Manager node (Python): poll leak/temp/humidity sensors via GPIO/I2C
- [ ] **CTRL-04** Implement joystick passthrough: USB joystick → MAVLink `MANUAL_CONTROL` messages
- [ ] **CTRL-05** Add basic fail-safe: if Raspberry Pi loses Jetson heartbeat for >3s, surface command
- [ ] **CTRL-06** **[DEFER]** Current Disturbance Observer → defer until Phase 1
- [ ] **CTRL-07** **[DEFER]** Data logging with timestamps → defer until Phase 1

### 0.7 Operator Interface (ROV Dashboard)
- [ ] **UI-01** **[NEW]** Set up QGroundControl or BlueOS web interface on surface laptop
- [ ] **UI-02** **[NEW]** Configure dashboard to display: primary camera feed (OAK-D), depth, heading, thruster status, leak detection
- [ ] **UI-03** **[NEW]** Add telemetry overlays on video feed (depth, heading, target waypoints)
- [ ] **UI-04** **[NEW]** Test interface in simulation before pool deployment

### 0.8 Hardware Bring-Up (Physical Vehicle)
- [ ] **HW-01** Assemble pressure vessel: acrylic dome, O-rings, penetrators, bulkheads
- [ ] **HW-02** Test waterproofing: dunk test in freshwater pool for 1 hour, check for leaks
- [ ] **HW-03** Mount thrusters and servos to frame; verify clearance and wiring routing
- [ ] **HW-04** Assemble custom PCB and verify power distribution: 5V, 3.3V, servo PWM, I2C pull-ups
- [ ] **HW-05** Install batteries and verify voltage/current monitoring (if available)
- [ ] **HW-06** Dry test: power on all electronics (thrusters not submerged), verify no shorts
- [ ] **HW-07** Wet test: submerge vehicle, verify buoyancy (slightly positive), trim using weights
- [ ] **HW-08** **[DEFER]** Install cameras and lenses → defer until Phase 1 (ROV uses only depth + IMU)
- [ ] **HW-09** **[NEW]** Perform depth sensor calibration: compare Bar30 readings against known depths

### 0.9 ROV Integration & Sea Trials
- [ ] **INT-01** Connect all layers: joystick → Raspberry Pi → Cube Orange → thrusters
- [ ] **INT-02** Perform pool test: manual control with joystick, verify 6-DOF motion (surge, sway, heave, yaw)
- [ ] **INT-03** Test depth hold in manual mode: engage depth PID via RC channel
- [ ] **INT-04** Verify fail-safe: disconnect Jetson (or simulate crash), vehicle surfaces
- [ ] **INT-05** Leak test: submerge vehicle to 3m for 30 minutes, monitor leak sensors
- [ ] **INT-06** **[DEFER]** Endurance test: 20-minute continuous operation → defer after ROV validated
- [ ] **INT-07** **[DEFER]** Add telemetry dashboard (BlueOS web interface) → defer until Phase 1

### 0.10 Jetson Integration (During ROV Trials)

- [ ] **JET-01** Install ROS 2 Jazzy on Jetson with MAVROS and DepthAI drivers
- [ ] **JET-02** Implement heartbeat publisher node: sends `jetson_heartbeat` to Raspberry Pi at 1Hz
- [ ] **JET-03** Implement passive data logger: subscribes to `/imu`, `/depth`, `/thruster_telemetry`, logs to CSV/ROS bag
- [ ] **JET-04** Verify OAK-D and ArduCam streams over USB 3.0; check for dropped frames
- [ ] **JET-05** (Optional) Run visual odometry pipeline in passive mode — no control outputs
- [ ] **JET-06** Test fail-safe: disconnect Jetson USB/UART → Raspberry Pi commands surface
- [ ] **JET-07** Verify Jetson power consumption and thermal performance during 30-minute pool test

---

## Phase 1: AUV Development (Weeks 9-20)

### 1.1 Development Environment (Phase 1 Additions)
- [ ] **ENV-AUV-01** Create Dockerfiles for reproducible builds on Jetson (ROS 2 Jazzy + CUDA/OpenCV)
- [ ] **ENV-AUV-02** Set up `docker-compose.yml` for Jetson with camera passthrough and volume mounts
- [ ] **ENV-AUV-03** Set up CI/CD: GitHub Actions for ROS 2 build verification on push to `main`/`dev`
- [ ] **ENV-AUV-04** Add pre-commit hooks for linting (Python `black`, `ruff`, C++ `clang-format`)
- [ ] **ENV-AUV-05** Create documentation pipeline: Doxygen for C++, Sphinx for Python, auto-build on PR merge
- [ ] **ENV-AUV-06** Write `Dockerfile.ros` with ROS 2 Jazzy + DepthAI + OpenCV + CUDA dependencies
- [ ] **ENV-AUV-07** Write `Dockerfile.pi` with BlueOS extension development environment

### 1.2 Perception Layer
- [ ] **PER-01** Install OAK-D Lite drivers (`depthai-ros`) on Jetson; verify camera streaming at 30Hz
- [ ] **PER-02** Install ArduCam drivers; verify global shutter streaming at 60Hz
- [ ] **PER-03** Implement ORB-SLAM3 (stereo mode) with ROS 2 Jazzy wrapper; validate against simulation
- [ ] **PER-04** Implement KLT optical flow on ArduCam as fallback odometry
- [ ] **PER-05** Create VO Quality Monitor node: feature count, disparity variance, tracking stability → publish `vo_quality`
- [ ] **PER-06** Implement sensor fusion: EKF on Jetson fusing VO + IMU + depth + compass
- [ ] **PER-07** Add fogging prevention: humidity sensor triggers heaters; publish status
- [ ] **PER-08** **[DEFER]** YOLO/neural network for object detection (gate/buoys/torpedoes) → defer until Phase 2
- [ ] **PER-09** **[DEFER]** Depth map generation (OAK-D disparity) → defer until Phase 2
- [ ] **PER-10** **[NEW]** Document camera lens port optical properties under pressure (calibration reference)

### 1.3 Autonomy Layer (Core)
- [ ] **AUTO-01** Implement ROS 2 node for Mission Planner: simple state machine with 5 states (IDLE, NAVIGATE, TASK, RECOVER, ABORT)
- [ ] **AUTO-02** Implement Path Planner: A* or RRT in 2D (horizontal plane) with depth waypoints
- [ ] **AUTO-03** Implement Motion Controller: PID on position error → velocity commands (surge, sway, heave, yaw)
- [ ] **AUTO-04** Implement Dead Reckoning Fallback: integrate IMU acceleration + thruster telemetry (from HAL-09) when `vo_quality < 0.3`
- [ ] **AUTO-05** Implement Re-Acquisition Behavior: spiral search when position uncertainty > 0.5m
- [ ] **AUTO-06** Add EKF adaptive covariance: inflate when `vo_quality` drops
- [ ] **AUTO-07** Add real-time logging: state estimates, commands, `vo_quality`, uncertainty
- [ ] **AUTO-08** **[DEFER]** Behavior tree implementation (state machine is sufficient for Phase 1) → defer until Phase 2
- [ ] **AUTO-09** **[DEFER]** Obstacle avoidance → defer until Phase 2
- [ ] **AUTO-10** **[NEW]** Design lightweight acoustic status protocol (if modem acquired) for emergency communications

### 1.4 Control Layer (AUV Enhancements)
- [ ] **CTRL-AUV-01** Implement Current Disturbance Observer on Raspberry Pi: compare desired vs. achieved velocity
- [ ] **CTRL-AUV-02** Add feedforward compensation to velocity commands before MAVLink forwarding
- [ ] **CTRL-AUV-03** Extend fail-safe: if `vo_quality < 0.1` for >10s, trigger re-acquisition (not surfacing)
- [ ] **CTRL-AUV-04** Add thruster telemetry logging: RPM, current draw, PWM values → for dead reckoning
- [ ] **CTRL-AUV-05** **[DEFER]** Web dashboard for real-time state visualization → defer until Phase 2
- [ ] **CTRL-AUV-06** **[DEFER]** Data logging to onboard SSD (Jetson) → defer until Phase 2

### 1.5 HAL Enhancements (Cube Orange)
- [ ] **HAL-AUV-01** Configure EKF3 to accept external vision via `VISION_POSITION_ESTIMATE` MAVLink messages
- [ ] **HAL-AUV-02** Implement conditional fusion: only accept external vision if `vo_quality > 0.5`
- [ ] **HAL-AUV-03** Enable `VISION_ESTIMATOR` in ArduSub parameters
- [ ] **HAL-AUV-04** Test vision fusion in simulation (ArduSub Gazebo + ROS 2 bridge)
- [ ] **HAL-AUV-05** **[DEFER]** Tune EKF3 covariance parameters on real vehicle → defer after pool tests

### 1.6 Perception Hardware Installation
- [ ] **HW-AUV-01** Mount OAK-D Lite in acrylic dome with anti-fog coating; verify waterproofing
- [ ] **HW-AUV-02** Mount ArduCam Global Shutter in separate housing; verify waterproofing
- [ ] **HW-AUV-03** Install humidity sensor inside dome; route I2C to Raspberry Pi
- [ ] **HW-AUV-04** Configure camera exposures: OAK-D (auto-exposure), ArduCam (fixed exposure for optical flow)
- [ ] **HW-AUV-05** Verify camera streaming over USB 3.0 to Jetson (no dropped frames)

### 1.7 AUV Integration & Pool Tests
- [ ] **INT-AUV-01** Integration test: Perception → Autonomy → Control → HAL (all layers active)
- [ ] **INT-AUV-02** Pool test: navigate to a visual target (AprilTag) using visual odometry
- [ ] **INT-AUV-03** Pool test: visual dropout for 10s → verify dead reckoning hold within 0.5m
- [ ] **INT-AUV-04** Pool test: re-acquisition behavior (cover camera, then uncover → vehicle resumes)
- [ ] **INT-AUV-05** Pool test: station-keeping in artificial current (use trolling motor) → disturbance observer convergence <5s
- [ ] **INT-AUV-06** Pool test: complete a simple mission (gate → buoy) in simulation, then on real vehicle
- [ ] **INT-AUV-07** **[DEFER]** Full competition run (all tasks) → defer until Phase 2

### 1.8 Pre-Competition Validation
- [ ] **VAL-01** Document all parameters: PID gains, EKF covariances, VO thresholds, disturbance observer gains
- [ ] **VAL-02** Create mission script for competition: define waypoints and task sequence
- [ ] **VAL-03** Run 5 complete mission simulations with different initial conditions
- [ ] **VAL-04** Perform 3 full pool test runs (with judges watching) to practice troubleshooting
- [ ] **VAL-05** **[DEFER]** Endurance test: 15-minute continuous operation → defer until after competition qualification
- [ ] **VAL-06** **[DEFER]** Night/testing conditions (low-light, turbid water) → defer until Phase 2

---

## Phase 2: Competition-Ready (Weeks 21-24)

### 2.1 ML & Perception Upgrades
- [ ] **ML-01** Train/YOLO model for gate detection (red/green buoys, torpedo launchers, etc.)
- [ ] **ML-02** Implement object detection inference on OAK-D's VPU (onboard neural accelerator)
- [ ] **ML-03** Fuse object detections with visual odometry → target-relative positioning
- [ ] **ML-04** **[DEFER]** Underwater image enhancement (auto white balance, contrast correction) → if time permits
- [ ] **ML-05** **[NEW]** Investigate loop closure detection for visual SLAM (ORB-SLAM3 map persistence or Bag-of-Words)

### 2.2 Advanced Autonomy
- [ ] **AUTO-ADV-01** Convert state machine to Behavior Tree (using `behaviortree_cpp_v3`)
- [ ] **AUTO-ADV-02** Implement task-specific behaviors: gate navigation, buoy traversal, torpedo firing, marker drop
- [ ] **AUTO-ADV-03** Add sequential task planning: complete all tasks in optimal order
- [ ] **AUTO-ADV-04** Add obstacle avoidance (static: pool walls; dynamic: other AUVs)
- [ ] **AUTO-ADV-05** **[DEFER]** Multi-vehicle coordination → not applicable (only one AUV)

### 2.3 Competition-Specific Tasks
- [ ] **TASK-01** Implement gate detection: identify gate center from OAK-D RGB + depth
- [ ] **TASK-02** Implement buoy traversal: detect colored buoys, navigate through them in sequence
- [ ] **TASK-03** Implement torpedo launcher: actuate servo based on object detection
- [ ] **TASK-04** Implement marker drop: actuate servo to release marker at GPS waypoint
- [ ] **TASK-05** Implement acoustic pinger tracking (if required) → needs external hardware

### 2.4 Finals Preparation
- [ ] **FINAL-01** Create competition-specific mission plan (customize for venue)
- [ ] **FINAL-02** Dry-run at competition pool (if allowed) or equivalent local pool
- [ ] **FINAL-03** Create troubleshooting checklist: common failures and recovery procedures
- [ ] **FINAL-04** Create recovery team protocol: physical retrieval, data download, battery swap
- [ ] **FINAL-05** **[DEFER]** Post-competition data analysis → after competition

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
