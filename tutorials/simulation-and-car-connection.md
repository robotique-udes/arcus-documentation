# Arcus – Simulation & Car Connection

This guide covers how to run the Arcus stack in simulation, connect to the physical car, build a map, and launch the autonomous driving pipeline.

> **Note:** Everything in this guide (simulation, car connection, mapping, and autonomous driving) is done from a **terminal on a Linux machine**.

## Table of Contents

- [1. Simulation](#1-simulation)
  - [1.1 Launch the simulator](#11-launch-the-simulator)
  - [1.2 Run Gap Follow in simulation](#12-run-gap-follow-in-simulation)
- [2. Connecting to the Car](#2-connecting-to-the-car)
  - [2.1 SSH into the Jetson](#21-ssh-into-the-jetson)
  - [2.2 Connect with VS Code](#22-connect-with-vs-code)
  - [2.3 Project structure on the car](#23-project-structure-on-the-car)
- [3. Mapping a Track (SLAM)](#3-mapping-a-track-slam)
  - [3.1 Configure the car for mapping](#31-configure-the-car-for-mapping)
  - [3.2 Launch the car packages](#32-launch-the-car-packages)
  - [3.3 Drive manually](#33-drive-manually)
  - [3.4 Launch the GUI](#34-launch-the-gui)
  - [3.5 Save the map](#35-save-the-map)
- [4. Running the Autonomous Robot](#4-running-the-autonomous-robot)
  - [4.1 Reconfigure the car for autonomous driving](#41-reconfigure-the-car-for-autonomous-driving)
  - [4.2 Prepare the map and race line](#42-prepare-the-map-and-race-line)

---

## 1. Simulation

### 1.1 Launch the simulator

From the simulation workspace:

```bash
cd /arcus-simulation/sim_ws
```

1. Start the Arcus environment:

   ```bash
   arcus-up
   ```

2. Open a shell inside the environment:

   ```bash
   arcus-bash
   ```

3. **First time only** (the first time you run `arcus-up`), run:

   ```bash
   b
   ```

4. Launch the simulator bridge:

   ```bash
   ros2 launch f1tenth_gym_ros gym_bridge_launch.py
   ```

5. In a **second terminal** (run `arcus-bash` again), start keyboard teleoperation:

   ```bash
   ros2 run teleop_twist_keyboard teleop_twist_keyboard
   ```

### 1.2 Run Gap Follow in simulation

1. Open the Gap Follow config file in VS Code:

   ```bash
   cd arcus/gap_follow/config
   code gap_follow.yaml
   ```

2. Set the drive topic to `/drive` (it was previously `/disparity/drive`):

   ```yaml
   drive_topic: /drive
   ```

3. Go back to an `arcus-bash` terminal and launch the node:

   ```bash
   ros2 launch gap_follow gap_follow.launch.py
   ```

---

## 2. Connecting to the Car

### 2.1 SSH into the Jetson

You can run this from any directory in your terminal:

```bash
ssh arcus@arcus-jetson.local
```

Password: `arcus`

### 2.2 Connect with VS Code

1. Open VS Code.
2. Click the **two small arrows** (Remote indicator) in the bottom-left corner and choose **Connect to Host… (SSH)**.
3. Choose **Add New SSH Host**, enter `arcus@arcus-jetson.local`, and select the first option that appears.
4. Click the two small arrows again, choose **Connect to Host**, and click `arcus@arcus-jetson.local`.
5. Enter the password: `arcus`.
6. Once connected, open the folder `/home/arcus`.

### 2.3 Project structure on the car

| Folder / File | Description |
| --- | --- |
| `arcus_vesc` | All code specific to the car (hardware-level). |
| `arcus_bringup` | Contains the launch files used to start the car (the nodes). |
| `config/arcus.yaml` | Car parameters (see below). |

Key parameters in `config/arcus.yaml`:

| Parameter | Description |
| --- | --- |
| `map_path` | Path to the map used by the car. |
| `run_slam` | Enables SLAM (map building). |
| `pure_pursuit` | Enables the Pure Pursuit controller. |
| `disparity` | Enables the Disparity Extender. |

---

## 3. Mapping a Track (SLAM)

### 3.1 Configure the car for mapping

In `config/arcus.yaml`, set:

```yaml
run_slam: true
pure_pursuit: false
disparity: false
```

### 3.2 Launch the car packages

In a new terminal:

```bash
cd /arcus-vesc/scripts
sh launch_car_packages.sh
```

Password (if prompted): `arcus`

### 3.3 Drive manually

Open the keyboard teleoperation node:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

#### Using a controller (gamepad)

To use a controller:

1. Stop the stack: `arcus-down`
2. Plug in the controller.
3. Start the stack again: `arcus-up`

### 3.4 Launch the GUI

```bash
arcus-bash
ros2 launch arcus_gui gui.launch.py
```

- To enable the controller, click **controller driver** in the GUI.
- To visualize the car, click **visualization** in the GUI and, in the **Map** section, select `/slam_map`.

### 3.5 Save the map

In a new terminal:

```bash
arcus-bash
ros2 run slam_map_server slam_map_saver
```

---

## 4. Running the Autonomous Robot

### 4.1 Reconfigure the car for autonomous driving

Go back to `config/arcus.yaml` in VS Code and set:

```yaml
pure_pursuit: true
disparity: true
run_slam: false
```

### 4.2 Prepare the map and race line

1. In another terminal, open `arcus-bash` and go to the map preparation folder:

   ```bash
   cd src/scripts/map_preparation
   ```

2. Run the race line utility (a menu will appear):

   ```bash
   python3 prepare_map.py
   ```

3. Go through **all menu options in the order proposed** (password if prompted: `arcus`).

4. When running **Clean map**:
   - A new window opens. Click on it, then use the **keyboard arrow keys** to adjust the **inner** and **outer** boundaries of the race track.
   - When asked `use auto-detected contours (Y/N)`, answer **N**.
   - When you are done, press **Enter** to exit Clean map.

5. Run **Export raceline** to send the race line to the car.
