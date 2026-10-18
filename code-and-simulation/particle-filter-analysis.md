# Particle filter code analysis

This page describes the particle filter initialization and update pipeline, sensor model, and potential areas for further investigation.

## Initialization

Most initialization happens in the map callback (`map_cb`):

- The map is converted to OMap for `range_lib`.
- The range method is created according to the selected GPU setting.
- The sensor model is precomputed using the configured sensor-model values.
- If no initial pose was provided, particles are initialized with random positions and weights.
- If an initial pose was set, particles are generated randomly around that pose.

## `lidar_cb`

- Initialize multiple arrays when the first scan arrives.
- Downsample the lidar scan.
- Call the update step once for each scan.

## `odom_cb`

- Calculate the delta (x,y, theta) between the old and new poses, and transform it into local coordinates (`odometry_data`).
- Store linear and angular velocity to reuse for odometry publishing if that option is enabled.

## `update()`

### `sampling()`

- Create a distribution based on the particle weights.
- Randomly select particles according to this distribution until the maximum particle count is reached. If a particle had a high weight it will be selected multiple times.

### `motionModel()`

- Update particle positions from the odometry delta, adding normally distributed random noise.
- The motion-dispersion parameters affect this noise.

### `sensorModel()`

- For each particle, use ray casting with `range_lib` to calculate hypothetical ranges.
- Evaluate each particle against the actual observations using the sensor model.
- Apply a squash factor to the calculated weights to avoid overly rapid convergence.
- Normalize the weights (looks like from 0.0 to 1.0).

### Sensor model

The sensor model uses a precomputed table generated for `range_lib`. It represents the probability of a lidar beam measurement given a theoretical range from ray casting. The table is indexed by simulated range `r` and observed range `d`; each cell represents the probability of that simulated/observed range combination.

After some research the model is based on a range-finder beam sensor model described in this book p.153: [Probabilistic Robotics](https://gaoyichao.com/Xiaotu/resource/refs/PR.MIT.en.pdf)

There is also an algorithm described in the book to determine the intrinsic parameters at p.159. We should definitely look into this.

| Case | Contribution to the beam probability |
| --- | --- |
| Hit | Gaussian likelihood based on `z = r - d`, weighted by `z_hit`: `z_hit / (sigma_hit * sqrt(2π)) * exp(-z² / (2 * sigma_hit²))` |
| Short reading (`r < d`) | `2 * z_short * (d - r) / d` |
| Maximum-range reading (`r == max_range_px`) | `z_max` |
| Random reading (`d < max_range_px`) | `z_rand / max_range_px` |

The table can be pictured with simulated range `r` on one axis and observed range `d` on the other. Each cell stores the probability for that pair. For example, a particle predicting 20 m when the observed range is 5 m should receive a low hit likelihood.

The hit term is a Gaussian likelihood using the difference `z = r - d` and configured hit spread/variance `sigma_hit`:

```text
z = r - d
hit likelihood ∝ (z_hit / (sigma_hit * sqrt(2π))) * exp(-z² / (2 * sigma_hit²))
```

The beam contributions are combined by the sensor model, and particle weights are normalized. Confirm branch conditions, units, and constants against the implementation when changing this model.

### `applyRoughening()` (commented out)

- Calculate the particles' effective sample size (ESS) to detect whether they have become too uniform, such as during recovery, turning, or other situations with little information.
- Add noise to particles if the ESS is too low.
- Investigate why this code is commented out.

### `expectedPose()`

- Calculate the mean particle pose using the weights.
I dont think this is optimal because of the nature of particle filters where there can be multiple particle clusters and instead of choosing one of them, a random particle in between the clusters is returned.
### `publishTfOdom()`

- Publish the transform and odometry for the calculated pose.

## Questions and possible investigations

- Visualization: color particles according to their weights.
- Check the sensor-model parameters against the formulas.
- Verify the sensor-model calculations.
- Calculate sensor-model parameters using algorithm described in the Probalistic Robotics book.
- Consider whether the calculation used to choose particles should change.
- Consider how `applyRoughening()` should be uncommented and what parameters it would need.
- Compare the approach with Nav2 AMCL to identify techniques that could be reused.
- Improve localization score calculations using particles distribution and weights.

## Main differences from Nav2 AMCL

This comparison uses the behavior described above for this particle filter and the current Nav2 AMCL configuration and implementation. AMCL has several configurable options, so behavior depends on its selected model and parameters.

| Area | This particle filter | Nav2 AMCL |
| --- | --- | --- |
| Particle count | Uses a configured `max_particles` count. | Supports minimum and maximum particle counts and an adaptive particle filter. |
| Scan update timing | Processes each lidar scan after downsampling it by `angle_step`. | Can wait for configured minimum translation or rotation before updating; `max_beams` controls the scan beams used. |
| Sensor model | Uses a precomputed range table and ray-cast predicted ranges, with hit, short, max-range, and random terms. | Selects a beam, likelihood-field, or likelihood-field-prob sensor model. Likelihood-field-prob can skip beams that disagree with most particles. |
| Motion model | Adds normally distributed noise to particle motion using motion-dispersion parameters. | Uses a selectable differential or omnidirectional motion model, with configurable `alpha` noise parameters. |
| Resampling and recovery | Resamples by particle weights; `applyRoughening()` is commented out in the analyzed implementation. | Has a configurable resampling interval and optional recovery using fast and slow weight averages. |
| Pose estimate | Computes a weighted mean particle pose. | Tracks particle clusters/hypotheses and publishes the highest-weight hypothesis. |

Nav2 AMCL details: [configuration guide](https://docs.nav2.org/configuration/packages/configuring-amcl.html) and [AMCL implementation](https://github.com/ros-navigation/navigation2/tree/main/nav2_amcl).
