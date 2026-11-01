# ParamSaverNode

`ParamSaverNode` is a centralized ROS 2 node designed for persisting, restoring, and dynamically switching parameter configuration profiles across multiple target nodes in the ARCUS workspace.

By centralizing configuration management into this node, target nodes do not need custom code to save or load their own state. Instead, `ParamSaverNode` communicates with target nodes using standard ROS 2 parameter services.

---

## Core Capabilities & Features

### 1. Parameter Persistence (Saving)
When requested, the node queries all managed target nodes for their active runtime parameter values via standard ROS 2 services (`ListParameters` and `GetParameters`). It serializes the parameter values into ROS 2-compliant YAML configuration files on disk.

### 2. Dynamic Profile Management (Switching & Creation)
Profiles represent named sets of configuration files (e.g., `default`, `racing`, `testing`) stored under a common root directory (`config_profiles/`).
When the active profile (`config_name`) changes:
* **Existing Profile:** If the target profile directory exists and contains valid YAML files, `ParamSaverNode` parses them and pushes the saved parameter values to the target nodes using their `SetParameters` services.
* **New Profile:** If the requested profile directory does not exist, `ParamSaverNode` automatically creates it and dumps the current runtime parameters of all target nodes to serve as the new profile's baseline.
* **Redundant Switch Prevention:** If the requested profile matches the currently active profile, the operation is skipped to avoid unnecessary I/O and service call overhead.

### 3. Automatic Node State Persistence
Whenever the active profile is changed, `ParamSaverNode` writes its own updated `config_name` state to its dedicated configuration file (`param_saver_node.yaml`). This ensures that the selected profile persists across system reboots.

### 4. Profile Discovery
The node scans the configuration root directory (`config_profiles/`) and exports the list of available profile names as a string array parameter named `available_profiles`. External orchestrators and graphical tools can read this parameter to automatically discover saved profiles.

---

## Arcus GUI Integration

The **Arcus GUI** includes a dedicated **Parameter Manager Widget** designed specifically to interact with `ParamSaverNode`. 

### Key Features of the GUI Widget:
* **Visual Profile Selection:** Displays the currently active profile and automatically populates a drop-down selector using the `available_profiles` parameter.
* **One-Click Profile Switching:** Allows operators to switch profiles on the fly without needing command-line access.
* **One-Click Save (Sync):** Triggers the `/arcus/save_parameters` service directly from the interface to capture current runtime changes into the active profile.
* **Profile Creation:** Enables creation of new baseline profiles directly through the interface.

---

## Technical Overview & ROS 2 Interfaces

### Node Parameters
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `config_name` | `string` | The currently active profile name (default: `"default"`). Changing this triggers a profile load or creation. |
| `available_profiles` | `string[]` | List of detected profile directory names found in the configuration root. |

### Services
| Service Name | Service Type | Description |
| :--- | :--- | :--- |
| `/arcus/save_parameters` | `std_srvs/srv/Trigger` | Triggers a parameter dump of all managed nodes into the active profile's YAML files. |

---

## Command Line Usage (CLI)

While the Arcus GUI widget is the recommended interface, the node can also be controlled via standard ROS 2 CLI commands:

### 1. Save Current Parameters
Trigger a parameter dump to save runtime values across all managed nodes to the active profile:
```bash
ros2 service call /arcus/save_parameters std_srvs/srv/Trigger "{}"
```

### 2. Switch Active Profile
Set the active profile name to automatically switch parameter sets across target nodes:
```bash
ros2 param set /arcus/param_saver_node config_name "racing"
```

### 3. Query Active Profile Name
Retrieve the currently active configuration profile name:
```bash
ros2 param get /arcus/param_saver_node config_name
```

### 4. Query Available Profiles List
Retrieve the list of detected configuration profile directories:
```bash
ros2 param get /arcus/param_saver_node available_profiles
```
