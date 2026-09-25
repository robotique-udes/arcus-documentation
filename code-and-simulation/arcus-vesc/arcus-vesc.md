# Arcus VESC car: a beginner's guide

This repository contains the ROS 2 software that connects an Ackermann-steered car to its sensors and motor controller, estimates where it is, and starts parts of its driving software.

If autonomous navigation is new to you, the key idea is a repeated loop:

1. **Sense:** a laser scanner measures distances to nearby objects. The motor controller also reports wheel/motor measurements and an IMU reports movement and rotation.
2. **Estimate:** ROS nodes combine those measurements to estimate the car's motion and, in some modes, its position relative to a map.
3. **Choose a motion:** a driving algorithm uses the sensor data and estimated motion to choose a speed and steering angle.
4. **Move:** the VESC converts those requested controls into motor and steering actions. New measurements start the loop again.

ROS 2 nodes are separate programs that exchange typed messages on named **topics**. For example, `/scan` carries laser measurements and `/drive` carries a requested speed and steering angle. A **launch file** starts a group of related nodes and connects their settings.

## What's in the repository?

| Package or folder | In plain terms |
| --- | --- |
| `arcus_bringup` | The startup coordinator. It contains the main launch file, the car's shared settings, localization and mapping settings, and the robot description. It also has a small node that publishes the front wheel positions for visualization and transforms. |
| `urg_node2` | The driver for a Hokuyo laser scanner. It connects to the scanner and publishes distance readings. |
| `vesc/vesc_driver` | The driver that communicates with the VESC motor controller over a serial port. |
| `vesc/vesc_ackermann` | Converts between the car-style command (speed and steering angle), VESC commands, and odometry (an estimate of how the car has moved). |
| `vesc/vesc_msgs` | Message formats used to report VESC measurements, including IMU data. |
| `transport_drivers/asio_cmake_module`, `io_context`, `serial_driver` | Reusable low-level support used to communicate with the VESC over a serial connection. |
| `transport_drivers/udp_driver` | A reusable networking package included in the workspace; the main car launch does not start it directly. |

The main launch also expects other ROS packages (including `robot_localization`, `slam_toolbox`, and Nav2's map server) and Arcus driving packages for functions such as safety, gap following, and pure pursuit. Several Arcus packages and the particle-filter configuration are referenced by absolute paths under `/home/arcus`; they are not included in this repository. A launch failure about a missing package or file may mean that machine-specific software has not been installed or its path needs updating.

## How the pieces fit together

```text
Hokuyo laser ──> /scan ───────────────────┐
                                          ├─> mapping / localization
VESC ──> speed, odometry, and IMU ─────────┘           │
                                                       v
                                            estimated car movement
                                                       │
                         driving algorithm <──────────┘
                                │
                         /drive command
                                │
                                v
                         VESC ──> car moves
```

**Odometry** is a running estimate of movement from the car's sensors; it can drift over time. **Localization** means estimating where the car is on a known map. **Mapping** means building or updating a map from laser readings as the car moves. [**SLAM**](../slam.md) does both together: it uses the map and the car's estimated motion to improve its estimate of where it is.

A **frame** is a named coordinate reference, such as `map`, `odom`, or the car's `base_link`. **TF** is ROS's system for describing how these coordinate frames relate. Consistent frame names matter because they let nodes agree on where the car and sensor are.

## Settings to know first

Most day-to-day choices are in `arcus_bringup/config/arcus.yaml`. It is read by `arcus_bringup/launch/arcus_bringup.launch.py`.

### Topics and car name

- `namespace`: the car name used in frame names, currently `ego_racecar`.
- `scan_topic`: where the laser driver publishes distance readings, currently `/scan`.
- `odom_topic`: the raw movement estimate, currently `/odom`.
- `ekf_odom_topic`: the filtered movement estimate, currently `/odometry/filtered`.
- `drive_topic`: where a driving node sends requested speed and steering, currently `/drive`.
- `slam_map_topic`: where SLAM publishes its changing map, currently `/slam_map`.

These names are how nodes find one another. If you change a topic name, the nodes that publish and subscribe to it must agree.

### Map and start position

- `map_path`: path to a saved map, without the `.yaml` ending. The launch adds that ending when it starts the map server.
- `slam_maps_dir`: folder searched for a saved map when localization is on and SLAM is off. In that case the launch picks the most recently modified `.yaml` file in that folder.
- `sx`, `sy`, `stheta`: intended starting x-position, y-position, and heading on the map. The current TF publisher does not use these values.
- `map_img_ext`: declared here, but the current top-level launch does not use it.

### Choosing mapping or localization

- `run_slam`: start SLAM. This takes precedence over `localize` in the launch's mode selection.
- `localize`: when true and `run_slam` is false, start the particle-filter localization node using a saved map.
- `run_ekf`: controls the EKF-only branch when neither localization nor SLAM is selected. In the current launch logic, EKF is also started in the SLAM and particle-filter branches.
- `pure_pursuit`: start the pure-pursuit driving package if available.
- `disparity`: start the gap-follow/disparity driving package if available.
- `kb_teleop`: a keyboard-teleoperation setting declared by the node, but it is not used by the current top-level launch logic.

In the checked-in config, `localize` and `run_slam` are both `true`. Since SLAM takes precedence, this starts EKF and SLAM rather than the particle filter. `disparity` is also true, so the launch attempts to start the gap-follow package. Pure pursuit is off. The map server is started regardless, using `map_path`. Understand this combination before running; changing a mode switch changes which processes start, but does not by itself install missing packages or make a saved map available.

## Other configuration files

### `arcus_bringup/config/ekf.yaml`

The [**EKF**](../ekf.md) (extended Kalman filter) combines measurements into a smoother movement estimate. Here it reads `/odom` and `/imu/raw`, assumes the car moves in a flat 2D world (`two_d_mode`), and publishes filtered output and coordinate transforms. The `odom0_config` and `imu0_config` lists say which values in each sensor message the filter should trust (for example, forward speed or turning rate). You usually should not edit these lists without checking what the sensors actually report.

### `arcus_bringup/config/mapper_params_online_async.yaml`

These settings control `slam_toolbox`, the mapping program. Useful concepts:

- `mode: mapping` asks it to build a map. Localization mode instead uses an existing map.
- `resolution` is the approximate size of each map grid cell in metres; smaller cells make a more detailed, larger map.
- `min_laser_range` and `max_laser_range` limit which laser measurements are used.
- `minimum_travel_distance` and `minimum_travel_heading` set how far the car must move before a new scan is processed.
- Scan-matching and loop-closing settings control how the program aligns scans and recognizes a previously visited place. They affect map consistency and are more advanced tuning options.

The launch connects SLAM to the scan, odometry, and map topics selected in `arcus.yaml`.

### Laser scanner: `urg_node2/config/`

`params_ether.yaml` and `params_ether_2nd.yaml` are examples for network-connected scanners; `params_serial.yaml` is for a serial-connected scanner. They set the scanner address or serial port, the sensor frame name, scan angle range, and options such as intensity readings. Use the file and connection settings that match the actual scanner. The launch file `urg_node2/launch/urg_node2.launch.py` determines which settings are loaded by default.

### Motor controller: `vesc/vesc_driver/params/vesc_config.yaml`

This file sets the VESC serial port (`/dev/ttyACM0`), allowed ranges for commands such as speed and steering servo, and the IMU frame name. The port and limits must match the car's hardware. Incorrect command limits can change how the car responds, so do not copy values blindly between vehicles. Check `vesc/vesc_driver/launch/vesc_driver_node.launch.py` to see which parameter file is loaded.

## Run
Before launching, check the topic names, map path, mode switches, serial device, and scanner connection settings described above. Then start the car stack from the same sourced terminal:

```bash
./scripts/launch_car_packages.sh
```

Before running it, open a ROS 2 terminal where both ROS 2 and this workspace have been sourced. The script then:

1. Runs `sudo chmod 777 /dev/ttyACM0` to change access permissions on the VESC serial device.
2. Sets Wi-Fi power saving to `2` on the NetworkManager connection named `ARCUS-5Ghz`.
3. Starts `arcus_bringup.launch.py` with ROS 2.

The script assumes those exact device and Wi-Fi names exist. It has no stop-on-error setting: if a setup command fails, it can still proceed to launch. The launch starts the car's shared bringup node, map server, robot model publisher, laser and VESC drivers, vehicle conversion nodes, and several Arcus application launches. It then adds mapping/localization and optional driving processes according to `arcus.yaml`.

The launch file contains absolute paths to some Arcus packages and the particle-filter config under `/home/arcus`. Those dependencies and paths need to exist on the computer running the car.

## Where to look in the code

- Main startup and node selection: `arcus_bringup/launch/arcus_bringup.launch.py`
- Topics, map, and mode switches: `arcus_bringup/config/arcus.yaml`
- Filtered movement estimate: `arcus_bringup/config/ekf.yaml`
- Mapping settings: `arcus_bringup/config/mapper_params_online_async.yaml`
- Robot model: `arcus_bringup/launch/ego_racecar.xacro`
- Startup helper: `scripts/launch_car_packages.sh`
