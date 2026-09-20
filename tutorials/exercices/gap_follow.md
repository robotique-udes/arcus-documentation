# Gap follow exercice

In this exercice, you will learn how to code a simple gap-following algorithm and will be able to test it on our simulation.

**Tip** : Avoid looking up existing gap-follow solutions online. Instead, we recommend consulting documentation like the [ROS2 Humble tutorials](https://docs.ros.org/en/humble/Tutorials.html) or running targeted searches for specific components rather than the full solution.

## The node

Your algorithm will be handled in a ROS node. In C++ code, a node is a class that **inherits** from the `public rclcpp::Node` class. You can create a `.cpp` and `.hpp`file for it in a package:

```
ros2 pkg create my_gap_follow
```

## Variables needed

You need many variables in you node. They can be stored in the private section of you class (`.hpp` file).

### Strings

`std::string` variables that can store static text

* The lidar scan topic name to access it : `"/scan"`

* The drive topic to send movement instructions to the car : `"/drive"`

### ROS handles

* A subscriber to fetch the [lidar scans](https://docs.ros.org/en/noetic/api/sensor_msgs/html/msg/LaserScan.html) (subscribes to `sensor_msgs::msg::LaserScan`)

* A publisher to send [drive](https://docs.ros.org/en/noetic/api/ackermann_msgs/html/msg/AckermannDriveStamped.html) commands to the car (publishes to `ackermann_msgs::msg::AckermannDriveStamped`)

### Algorithm parameters and tuning Constants

* `BUBBLE_RADIUS` : Radius (in meters or array indices) used to inflate nearby obstacles (ex: car width safety margin).
* `PREPROCESS_DIST_MAX` : Maximum range cap for LIDAR readings to filter out far-away walls/noise.

## Suggested functions

### Main function

```cpp
int main(int argc, char ** argv) {
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<MyGapFollow>());
    rclcpp::shutdown();
    return 0;
}
```

### Constructor

A constructor function to initialize your variables and create your subscribers/publishers (ex: `MyGapFollow()`).

### Lidar callback

A function that will be connected to the subscriber (ex: `lidar_callback`). When the suscriber recieves a value, this function is called. Therefore, this is where your main logic will be.

This function will recieve the `LaserScan` as a **parameter**.

#### Preprocess function

A function to preprocess the scan data just after recieving it (in the lidar callback). This function will be used to filter out invalid readings (`NaN`, `Inf`, or ranges outside `range_min` / `range_max`). Cap readings greater than a chosen threshold (`PREPROCESS_DIST_MAX`) to remove noise that may affect the selection of the gap.

#### Find closest obstacle

A function that scans the preprocessed array to locate the index of the minimum distance reading (closest obstacle). Returns the index of the closest point.

#### Bubble around closest obstacle

A function that protects the vehicle from clipping walls or obstacles. Around the closest point (using it's index found earlier), set all values within a physical radius (`BUBBLE_RADIUS`) to `0.0`.

#### Find max gap

A function that finds the largest consecutive sequence of non-zero (valid) readings in the range array. The valid readings will be the ones that were kept in the preprocess function. Returns the indexes of the start and end of the gap.

#### Find best point

A function that determines the target heading inside the selected gap. Returns the index of the best point.

**Strategies**:
* **Deepest point**: Find the maximum distance value within the gap.
* **Center point**: Pick the midpoint (center of the gap) for smoother driving down corridor centers.

#### Publish drive

A function that converts the array index into an actual steering angle and publishes an `AckermannDriveStamped` message.

**Steering Formula**: $\text{steering\_angle} = \text{angle\_min} + (\text{target\_id} \times \text{angle\_increment})$
* Adjust speed dynamically: reduce velocity for sharp steering angles, increase velocity on straight paths.
* `angle_min` and `angle_increment` are found in the LaserScan message.

## Test your code

To test you code :

* Open a terminal to launch the simulation

```bash
arcus-bash

ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

* Open another terminal to run your gap-follow code

```bash
arcus-bash

b # Build your node package

ros2 run my_gap_follow my_gap_follow
```