# gap_follow

See the [system overview](overview.md) for how this node interacts with the rest of Arcus.

`gap_follow` drives using only the nearby lidar scan; it does not need a map or a waypoint path. It looks for open space in front of the car and steers toward the farthest safe direction. This makes it useful as a fallback when map-based driving is unsafe.

A *disparity* is a sudden jump between two neighboring lidar distances, which usually marks the edge of an obstacle. The node expands these edges by the car's clearance radius so that the selected opening is wide enough for the whole car.

## Node behavior

- Executable: `gap_follow`; launched as `/arcus/gap_follow`.
- Publishes an `arcus_msgs/msg/ErrorCode` heartbeat with source `DISPARITY` every 50 ms.
- Smooths steering by averaging recent commands.
- Commands the minimum of proportional clearance speed, estimated cornering limit, and `max_speed`.

## ROS interface

| Direction | Topic | Type | Purpose |
| --- | --- | --- | --- |
| Subscribe | `lidar_scan_topic` (`/scan`) | `sensor_msgs/msg/LaserScan` | Raw scan |
| Publish | `drive_topic` (`/disparity/drive`) | `ackermann_msgs/msg/AckermannDriveStamped` | Reactive command |
| Publish | `/node_error_code` | `arcus_msgs/msg/ErrorCode` | Liveness |
| Publish, debug | `/processed_scan` | `sensor_msgs/msg/LaserScan` | Extended front scan |
| Publish, debug | `/target_waypoint` | `geometry_msgs/msg/PointStamped` | Selected point |
| Publish, debug | `/vector` | `geometry_msgs/msg/PoseStamped` | Steering visualization |

## Parameters

| Name | Config value | Description |
| --- | ---: | --- |
| `static_friction_coeff` | `0.7` | Tire/ground friction estimate |
| `wheel_base` | `0.324` | Wheelbase in metres |
| `bubble_radius` | `0.30` | Disparity clearance width |
| `speed_distance_factor` | `1.2` | Clearance-to-speed gain |
| `max_speed` | `5.0` | Speed cap in m/s |
| `disparity_threshold` | `0.15` | Range jump defining a disparity |
| `default_qos` | `1` | History depth |
| `rolling_average_window` | `1` | Steering smoothing samples |
| `debug` | `false` | Enable visualization outputs |

Numeric tuning parameters and `debug` support runtime updates. A zero rolling window is forced to one.

The scan must cover at least -90 to +90 degrees and have a non-zero `angle_increment`. Infinite, NaN, and out-of-range samples are replaced using adjacent readings.
