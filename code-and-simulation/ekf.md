# EKF

An **Extended Kalman Filter (EKF)** is a method that keeps a running best guess of a system’s state by predicting where it should be, comparing that prediction to noisy sensor measurements, and making small corrections using a linear approximation when the system or sensors are nonlinear.

An EKF reduces drift by fusing wheel odometry and IMU measurements: it predicts the vehicle's pose from the motion model and wheel/IMU inputs, then corrects that prediction when new sensor measurements arrive. The filter tracks uncertainty (covariances) so sensor inputs are weighted according to their trustworthiness, which helps limit dead-reckoning drift.

**Note**: the EKF's performance depends on correct sensor noise/covariance tuning and accurate motion and measurement models—poor tuning can degrade results.

For a better and simpler explanaiton of kalman filters see this video:
https://www.youtube.com/watch?v=IFeCIbljreY

For the EKF, we use robot_localization's. Here is the wiki of each of its parameters: 
https://docs.ros.org/en/melodic/api/robot_localization/html/state_estimation_nodes.html#ekf-localization-node

It's config file can be found here:
```bash
/sim_ws/src/f1tenth_gym_ros/config/ekf.yaml
```

To use the EKF in simulation, set the following parameter in `/sim_ws/src/f1tenth_gym_ros/config/sim.yaml`:
```yaml
use_sim_localization: True
```
The EKF and particle filter will automatically be launched when launching the sim:
```bash
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```