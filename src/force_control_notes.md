# UR10e Active Compliance and Force Control

This project highlights a key advantage of collaborative robots (UR10e) over traditional rigid industrial arms: the ability to decouple rigid positional control from compliant force control along specific Cartesian axes.

## 1. The Polishing Challenge
When an industrial robot polishes a car hood, the surface is never perfectly flat—it curves in 3D space. 
- If you simply program stiff X, Y, Z waypoints across the hood, any slight deviation in the physical car's placement or the hood's curvature will either cause the polishing tool to lose contact with the surface (leaving unpolished spots) or crash into the metal (damaging the payload).

## 2. Dynamic Surface Detection
Before polishing begins, the system must find the exact height of the car hood without crashing:
- The script commands a slow `movel` in the negative Z-direction (`[0.0, 0.0, -1.0]`).
- It polls `tool_contact()` continuously. Upon physical collision, the arm halts.
- Using `get_actual_joint_positions_history()`, the system calculates exactly where contact occurred and retracts +5mm to prepare for initialization.

## 3. Tare and `zero_ftsensor()`
Before activating compliance, the script calls `zero_ftsensor()`. This is critical. It tares the 6-axis Force/Torque sensor at the tool flange, mathematically nullifying the gravitational weight of the physical polishing wheel end-effector. Without this, the weight of the tool would skew the subsequent 10N force command.

## 4. `force_mode()` Mechanics
The core of the application lies in this single URScript command:
```python
force_mode(tool_pose(), [0, 0, 1, 0, 0, 0], [0.0, 0.0, 10.0, 0.0, 0.0, 0.0], 2, [0.1, 0.1, 0.15, 0.35, 0.35, 0.35])
```
Here is the engineering breakdown of what this accomplishes:
1. **Task Frame (`tool_pose()`)**: The forces will be calculated relative to the Tool Center Point (TCP), not the base of the robot. This ensures the force is always applied perpendicular to the tool face.
2. **Selection Vector (`[0, 0, 1, 0, 0, 0]`)**: This matrix tells the UR10e controller to remain *perfectly rigid* in the X, Y, Rx, Ry, and Rz axes. However, it hands the Z-axis over to compliance control.
3. **Wrench (`[0.0, 0.0, 10.0, ...]`)**: The robot is commanded to exert a constant 10 Newtons of force in that compliant Z-axis.
4. **Velocity Limits**: The final array sets maximum drift speeds (e.g., 0.15 m/s in Z). If the car hood was suddenly removed, the robot wouldn't plunge violently toward the floor; it would only drop at 0.15 m/s before erroring out.

## 5. Result
Because the Z-axis is now compliant, the robot ignores the strict Z-heights of `Waypoint_2` through `Waypoint_10`. It strictly follows the X and Y coordinates of the path while its internal PID loops dynamically adjust its real-time Z-height to maintain exactly 10N of pressure against the curving car hood.
