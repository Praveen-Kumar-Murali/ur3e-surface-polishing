# UR10e Active Compliance Surface Polishing

![Language](https://img.shields.io/badge/Language-URScript-blue) ![Robot](https://img.shields.io/badge/Robot-UR10e-orange) ![Topic](https://img.shields.io/badge/Topic-Force_Control-brightgreen) ![Status](https://img.shields.io/badge/Status-Active-success)

## Overview
Implementation of an automated continuous-surface polishing routine utilizing a Universal Robots UR10e. This collaborative robotics project bypasses rigid positional tracking by implementing Active Compliance Control (`force_mode`), dynamically applying a constant tool-pressure vector across an unpredictable 3D contour (a vehicle hood).

## System Architecture
* **Manipulator**: Universal Robots UR10e (Collaborative)
* **Controller Environment**: UR Polyscope / Native URScript
* **End-Effector**: Simulated Rotary Polishing Pad
* **Sensing Modality**: In-built 6-axis Force/Torque (F/T) kinematics

## Operational Workflow

The isolated core logic (`src/polishing_routine.script`) handles compliant trajectory execution through three sophisticated kinematic phases:

### 1. Dynamic Surface Searching
The UR10e executes a concurrent background descent thread. The primary loop polls the `tool_contact()` function, physically feeling for the car hood without prior positional knowledge. Upon contact, the arm halts (`stopl`), calculates the exact collision coordinate using position history buffers (`get_actual_joint_positions_history`), and retracts cleanly to a 5mm hover altitude.

### 2. Gravitational Taring
Prior to polishing, `zero_ftsensor()` nullifies the baseline gravitational strain placed on the tool flange by the end-effector's own weight, calibrating the baseline for precision pressure application.

### 3. Compliant Execution (`force_mode`)
The system engages a bespoke `force_mode` matrix:
- **Axis Selection**: `[0, 0, 1, 0, 0, 0]` decouples the Tool Z-axis, placing it into force-control while locking X, Y, and all angular orientations rigidly into position-control.
- **Wrench Command**: `10.0` Newtons of continuous positive Z-axis pressure is requested.
- **Dynamic Routing**: The robot executes eight subsequent linear Cartesian waypoints (`movel`) outlining the polishing pattern. Because Z is compliant, the robot ignores the hard-coded heights and physically rides the varying curvature of the car hood while preserving uniform 10N abrasion intensity.

## Demonstration

### Demo Video
[![Watch Demo](https://img.youtube.com/vi/1sj0m-wEcds/0.jpg)](https://www.youtube.com/shorts/1sj0m-wEcds)

[Watch on YouTube](https://www.youtube.com/shorts/1sj0m-wEcds)
