# particle_filter

See the [system overview](../documentation/main.md) for how this node interacts with the rest of Arcus.

`particle_filter` estimates the car's pose on the saved map.

It keeps many possible car positions called *particles*. As the car moves, those guesses move using wheel odometry. The node then compares the real lidar scan with the walls expected at each guessed position. Guesses that better match the map receive more importance. Their weighted average becomes the published position.

## Runtime components

- Component plugin: `particle_filter::ParticleFilter`
- Standalone executable: `particle_filter_node`
- Launch file: `launch/localize_launch.py`
- Configuration: `config/localize.yaml`

The launch file also starts the ROS map server, which publishes the saved map. `map_server.ros__parameters.map` selects a YAML file from `particle_filter/maps/`.

## ROS interface

| Direction | Topic | Type | Notes |
| --- | --- | --- | --- |
| Subscribe | `scan_topic` (`/scan`) | `sensor_msgs/msg/LaserScan` | Sensor-data QoS, depth 1 |
| Subscribe | `odometry_topic` (`/odometry/filtered`) | `nav_msgs/msg/Odometry` | Motion input |
| Subscribe | `/map` | `nav_msgs/msg/OccupancyGrid` | Transient-local map |
| Subscribe | `/initialpose` | `geometry_msgs/msg/PoseWithCovarianceStamped` | Manual initialization |
| Publish | `/pf/pose/odom` | `nav_msgs/msg/Odometry` | Estimated pose and measured twist |
| Publish | `/pf/viz/inferred_pose` | `geometry_msgs/msg/PoseStamped` | RViz estimate |
| Publish | `/pf/viz/particles` | `geometry_msgs/msg/PoseArray` | Particle sample |
| Publish | `/pf/viz/fake_scan` | `sensor_msgs/msg/LaserScan` | Predicted scan |
| TF | `map` -> `odom` | coordinate transform | Connects map coordinates to odometry coordinates |

## Important parameters

| Name | Config value | Meaning |
| --- | ---: | --- |
| `angle_step` | `10` | Use every nth lidar ray |
| `max_particles` | `2000` | Particle count |
| `squash_factor` | `2.2` | Softens sensor likelihoods |
| `range_method` | `rmgpu` | Backend; current code supports `rmgpu` and `glt` |
| `theta_discretization` | `112.0` | Ray-casting angular discretization |
| `max_range` | `10.0` | Modeled range in metres |
| `rangelib_variant` | `2` | Sensor implementation; only variant 2 is implemented |
| `publish_odom` | `1` | Enable odometry output |
| `viz` | `0` | Enable visualization |
| `max_viz_particles` | `60` | Visualized particle count |

The sensor-model parameters (`z_short`, `z_max`, `z_rand`, `z_hit`, and `sigma_hit`) describe how much to trust different kinds of lidar readings. The motion-dispersion parameters describe uncertainty in forward, sideways, and rotational motion. New members should normally start with the supplied values and change one parameter at a time.

An initial pose can be supplied through `/initialpose`, usually with RViz's **2D Pose Estimate** tool. Otherwise the node spreads guesses across the whole map after the map arrives.

## Tuning and constraints

- The filter waits until the map, odometry, and lidar have all arrived. A lack of output often means one of these inputs is missing.
- For CPU-only use, `range_method` must be set to `glt`; `rmgpu` uses the GPU.
- More particles improve robustness but increase ray-casting cost.
- A larger `angle_step` reduces work and scan detail.
- Output represents the lidar pose; `map`, `odom`, and `ego_racecar/laser` are fixed frame names in the implementation.
- Visualization messages are produced only when `viz` is enabled and, where applicable, a subscriber exists.
