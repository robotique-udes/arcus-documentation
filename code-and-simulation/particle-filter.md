# Particle filter (AMCL)

We use the nav2 AMCL (Adaptive Monte Carlo Localization) particle filter to localize the vehicle against a known map. AMCL represents the robot's pose belief with a set of weighted particles; it predicts particle poses from wheel odometry (and optionally IMU) and reweights/resamples them using sensor observations (e.g., lidar) to converge on the robot's true pose.

AMCL is good at recovering from moderate pose uncertainty and represents localization uncertainty explicitly via the particle distribution.

**Note**: AMCL needs a reasonably good initial pose and well-tuned parameters (number of particles, sensor model noise, and resampling thresholds); too few particles or incorrect sensor noise settings can prevent successful localization, while too many particles increase CPU usage.

Best explanation for AMCL:
https://www.mathworks.com/videos/autonomous-navigation-part-2-understanding-the-particle-filter-1594903924427.html

For the implementation we use, see the Navigation2 `nav2_amcl` package:
https://github.com/ros-planning/navigation2/tree/main/nav2_amcl

Configuration for AMCL is available in our simulation config directory. Example config location:
```bash
/sim_ws/src/f1tenth_gym_ros/config/amcl.yaml
```

To enable localization in simulation, set the following parameter in `/sim_ws/src/f1tenth_gym_ros/config/sim.yaml`:
```yaml
use_sim_localization: True
```

When enabled, the sim launch will start the EKF and the particle filter automatically:
```bash
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

If you need to perform a global localization (kidnapped robot), you can publish an initial pose with a large covariance or increase the particle spread temporarily so AMCL can re-sample across the map.

