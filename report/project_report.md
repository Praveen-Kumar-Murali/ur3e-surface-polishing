# Collaborative Surface Polishing (UR10e)

## 1. Introduction
This documentation outlines the architecture, logic, and force-control mathematics behind an automated surface polishing (e.g., sanding, wiping, buffing) application utilizing a Universal Robots UR10e. Using native URScript, the system demonstrates Active Compliance Control (`force_mode`) to maintain constant tool pressure across a complex, unpredictable contoured surface (such as a vehicle hood).

## 2. System Overview
Unlike traditional industrial robotics where rigid positioning is paramount, collaborative polishing requires the end-effector to physically yield to the environment. If the pre-programmed Z-axis trajectory deviates even slightly from the physical curvature of the car hood, a rigid arm will either lose surface contact or crash forcefully into the workpiece. 

To solve this, the UR10e leverages its built-in six-axis Force/Torque (F/T) sensors to:
1. Dynamically search for the hood surface (`tool_contact`),
2. Tare the tool payload gravity vector (`zero_ftsensor`),
3. De-couple the spatial axes, applying a constant 10-Newton downward pressure along the Tool Z-axis while remaining perfectly position-rigid along the Tool X and Y axes.

## 3. Robot and Controller Specifications
- **Manipulator**: Universal Robots UR10e (6-Axis Collaborative Robot)
- **Programming Environment**: UR Polyscope / Native URScript
- **End-Effector**: Simulated Polishing Pad
- **Key Sensor Integration**: Native `force_mode()` and `tool_contact()` physics polling

## 4. Application Architecture
The URScript logic executes the polishing pathway in three distinct control phases:

### Phase 1: Dynamic Surface Identification
- **Concurrent Linear descent**: Before polishing, the robot hovers safely over the hood. It then initiates a background thread commanding a deep linear plunge (`movel`).
- **Force Feedback Interruption**: The primary thread continuously polls `tool_contact()`. A payload collision against the hood triggers an immediate deceleration (`stopl(3.0)`). 
- **Backtrack Algorithm**: The script accesses the joint history buffer (`get_actual_joint_positions_history`) to determine the exact collision coordinate minus a 5mm offset, moving to hover perfectly just above the surface without touching it.

### Phase 2: Tool Taring
- **Gravitational Nullification**: The `zero_ftsensor()` command is mathematically critical. It calibrates the tool flange F/T reading to zero immediately prior to contact. This guarantees the subsequent 10N command isn't corrupted by the inherent weight of the polishing end-effector pulling down on the flange.

### Phase 3: Active Compliance Execution
The script activates the UR10e `force_mode` matrix:
`force_mode(tool_pose(), [0, 0, 1, 0, 0, 0], [0.0, 0.0, 10.0, 0.0, 0.0, 0.0], 2, [0.1, 0.1, 0.15, ...])`

- **Task Frame (`tool_pose`)**: All forces are calculated orthogonal to the end-effector face, not the robot base.
- **Selection Vector**: The script sets `[0,0,1,0,0,0]`. The `1` in the Z-index toggles the Z-axis from position-control to force-control. All other axes (X, Y, and orientations) remain locked in position-control.
- **Wrench Profile**: The Z-axis is commanded `10.0` Newtons of compressive continuous force.

Because the Z-axis is now compliant, the robot actively ignores the strict programmed Z-heights of the subsequent 8 polishing waypoints. It strictly follows the X and Y coordinates of the path while its internal PID loops dynamically adjust its real-time height to trace the physical curvature of the car hood maintaining precisely 10N.

## 5. Technical Highlights
By leveraging `force_mode`:
- The polishing path requires zero vision systems or complex 3D CAD modeling of the car hood's topological curves.
- The system is inherently immune to slight variations in part fixturing or suspension sag.
- Surface abrasion is perfectly uniform across the entire trajectory regardless of contour depth.
