# track_zone_manager

See the [system overview](overview.md) for how this node interacts with the rest of Arcus.

`track_zone_manager` changes vehicle behavior according to polygonal zones on the map. A zone can impose a speed limit or request a particular driving algorithm.

The node reads the zones from CSV files. Whenever it receives a new estimated position, it checks which polygon contains the car and publishes the instructions for that zone.

## Node and interfaces

- Executable: `track_zone_manager_node`; launched as `/arcus/track_zone_manager`.
- Subscribes to `odom_topic` (`/pf/pose/odom`, `nav_msgs/msg/Odometry`) to learn the car's estimated position.
- Publishes `speed_limit_topic` (`/track_manager/speed_limit`, `std_msgs/msg/Float64`) in m/s.
- Publishes `force_algo_topic` (`/track_manager/forced_algo`, `std_msgs/msg/String`).

Polygons are checked in ascending `polygon_id` order, so the lowest ID wins if zones overlap. When the car is outside every zone, the node publishes nothing. `arcus_master` automatically stops using an old zone instruction after a short timeout.

## Parameters

| Name | Configured value | Description |
| --- | --- | --- |
| `odom_topic` | `/pf/pose/odom` | Localization input |
| `speed_limit_topic` | `/track_manager/speed_limit` | Speed directive output |
| `force_algo_topic` | `/track_manager/forced_algo` | Algorithm directive output |
| `speed_zones_csv_path` | `/home/arcus/arcus/resources/speed_zones.csv` | Speed polygon file |
| `algos_csv_path` | `/home/arcus/arcus/resources/algos.csv` | Algorithm polygon file |

The supplied paths are absolute and must be changed for a different installation.

## CSV formats

Both files require a header. Each following row is a vertex; rows are grouped by `polygon_id` and sorted by `vertex_index`.

```csv
polygon_id,vertex_index,x,y,max_speed
1,1,-17.94,-6.22,1.0
1,2,-19.47,-1.99,1.0
1,3,-4.38,1.97,1.0
```

The algorithm file has the same first four columns and an `algorithm` fifth column. Values understood by `arcus_master` are `controller`, `safety`, `pure_pursuit`, and `disparity`. A polygon needs at least three vertices. Numeric conversion errors are not caught, so validate CSV data before launch.
