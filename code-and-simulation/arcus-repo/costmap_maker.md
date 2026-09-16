# costmap_maker

See the [system overview](overview.md) for how this node interacts with the rest of Arcus.

`costmap_maker` creates a local obstacle costmap in front of the car. Each grid cell contains a cost from 0 for free space to 100 for a lethal obstacle.

The node uses lidar readings inside a forward cone. It can remove walls that already exist in the saved map, leaving unexpected obstacles such as cones or another vehicle. It also expands obstacles by the car's safety radius so planners do not treat a narrow, unsafe gap as passable.

## Node and interfaces

The `costmap_node` executable runs as `costmap_maker`.

| Direction | Configured topic | Type |
| --- | --- | --- |
| Subscribe | `/scan` | `sensor_msgs/msg/LaserScan` |
| Subscribe | `/map` | `nav_msgs/msg/OccupancyGrid` |
| Subscribe | `/pf/pose/odom` | `nav_msgs/msg/Odometry` |
| Publish | `/local_costmap` | `nav_msgs/msg/OccupancyGrid` |
| Publish | `/costmap_maker/localization_score` | `std_msgs/msg/Float32` |

All topic names are parameters. The output grid uses `lidar_frame` and is rebuilt each cycle. *Inflation* means adding decreasing cost around an obstacle, giving the car a safety margin instead of representing it as a single point.

## Parameters

| Name | Value | Meaning |
| --- | ---: | --- |
| `lidar_frame` | `ego_racecar/laser` | Output frame |
| `resolution_m` | `0.05` | Cell size |
| `cone_range_m` | `7.0` | Forward extent / maximum hit range |
| `cone_fov_deg` | `120.0` | Accepted angular width |
| `update_rate_hz` | `30.0` | Rebuild rate |
| `min_valid_range_m` | `0.05` | Minimum hit range |
| `inflation_radius_m` | `0.15` | Inflation extent |
| `inscribed_radius_m` | `0.14` | Lethal-radius boundary |
| `cost_scaling_factor` | `9.0` | Inflation decay |
| `only_unmapped_obstacles` | `true` | Suppress hits matching the map |
| `global_obstacle_threshold` | `50` | Occupancy treated as blocked |
| `global_obstacle_neighborhood_radius_m` | `0.20` | Map-match search radius |
| `localization_decay_factor` | `1.0` | Mismatch score decay |

With `only_unmapped_obstacles` enabled, the global map and localized pose are needed to classify hits. The localization score measures disagreement between the current scan and the saved map. Larger values are treated as worse by the current master logic.
