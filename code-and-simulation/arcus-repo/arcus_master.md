# arcus_master

See the [system overview](overview.md) for how this node interacts with the rest of Arcus.

`arcus_master` arbitrates between candidate drive commands. It selects one speed and steering command, applies safety rules and speed limits, and publishes the result on `/drive`.

## Arbitration

The executable `master_node` launches as `/arcus/master_node`. Its arbitration loop runs at 50 Hz. If a controller heartbeat has not arrived for 300 ms, the master considers that source offline. Lower numeric priority values mean higher priority.

The deadman switch and emergency braking always take precedence. If the path ahead becomes risky, often because another car is occupying the intended line, or localization no longer agrees with the map, the master stops selecting pure pursuit and falls back to gap following. This fallback is most commonly used for overtaking: it looks for a nearby opening and steers into that safe gap. Track-zone instructions expire quickly so that a stale message cannot affect the car forever. Recognized algorithm names are `controller`, `safety`, `pure_pursuit`, and `disparity`.

## ROS interface

| Direction | Topic | Type |
| --- | --- | --- |
| Subscribe | `/controller/drive`, `/safety/drive`, `/pure_pursuit/drive`, `/disparity/drive` | `ackermann_msgs/msg/AckermannDriveStamped` |
| Subscribe | `/deadman_active` | `std_msgs/msg/Bool` |
| Subscribe | `/node_error_code` | `arcus_msgs/msg/ErrorCode` |
| Subscribe | `/track_manager/speed_limit` | `std_msgs/msg/Float64` |
| Subscribe | `/track_manager/forced_algo` | `std_msgs/msg/String` |
| Subscribe | `/pure_pursuit/trajectory_risk` | `std_msgs/msg/Float32` |
| Subscribe | `/costmap_maker/localization_score` | `std_msgs/msg/Float32` |
| Publish | `/drive` | `ackermann_msgs/msg/AckermannDriveStamped` |
| Publish | `/master_error_code` | `arcus_msgs/msg/ErrorCode` |
| Publish | `/master_heartbeat` | `std_msgs/msg/Bool` |

Topic names are configurable through matching parameters such as `drive_topic` and `disparity_drive_topic`.

## Behavior parameters

| Name | Config value | Description |
| --- | ---: | --- |
| `priority_controller` | `0` | Manual priority |
| `priority_safety_override` | `1` | Safety priority |
| `priority_pure_pursuit` | `2` | Pure-pursuit priority |
| `priority_disparity` | `3` | Gap-follow priority |
| `section_override_timeout_ms` | `500` | Zone-directive lifetime |
| `disparity_cooldown_ms` | `500` | Fallback hold after risk |
| `max_accepted_risk` | `70.0` | Pure-pursuit rejection threshold |

`max_accepted_risk` supports runtime updates. The code also refers to `min_accepted_localization_score`, but the constructor currently declares `max_accepted_risk` twice; correct that defect before configuring the localization threshold independently.

With no ready source, the node publishes a zero-speed, zero-steering command.
