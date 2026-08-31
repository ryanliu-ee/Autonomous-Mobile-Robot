# Autonomous Obstacle-Avoidance Robot

A differential-drive robot built to navigate point-to-point across a room while avoiding obstacles. Personal project — hardware, firmware, and (in progress) a custom carrier PCB.

## Status
🚧 In progress — subsystems developed and bench-tested individually; integration underway.

- [x] Motor control (bench-tested)
- [x] IR-based obstacle detection (bench-tested)
- [ ] IMU integration (in progress)
- [ ] Encoder odometry
- [ ] Point-to-point navigation
- [ ] Custom carrier PCB (KiCad)

## Hardware
- **MCU:** Arduino Nano 33 BLE Rev2 (onboard IMU)
- **Drive:** dual encoder motors, differential drive
- **Sensing:** 4–6 IR sensors for obstacle detection
- **Chassis:** ~150 × 120 mm, 3D-printed

## Software
Written in C/C++ (Arduino). Subsystems developed and validated independently before integration.

## Repo structure
- `/motor-control` — motor driver code
- `/ir-detection` — IR obstacle-detection code
- `/docs` — build photos, wiring notes, test documentation

## Notes
Built to practice the full hardware loop: embedded firmware, sensor integration, soldering/assembly, and PCB design.
