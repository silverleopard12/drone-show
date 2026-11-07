<!-- README.md(EN)와 README.ko.md(KO)는 함께 고친다 -->
**English** | [한국어](README.ko.md)

# Ego-Planner Swarm

ROS2 Humble implementation of ego-planner for multi-drone swarm coordination with multiple planning algorithms.

## Overview

When many drones fly together, for example in a drone show that moves from one formation to the next, every drone needs a trajectory that does not collide with the others. This workspace is built around the ego-planner swarm planner and collects several other ways of planning such trajectories, together with tools that assign drones to formation points and record their trajectories.

| Folder | What it does | Docs |
|---|---|---|
| `src/planner/plan_manage` (ROS package `ego_planner`) | The ego-planner swarm planner. The launch files in [Running the code](#running-the-code) are here. | — |
| `src/planner/headway_planner` | Offline planner for drone shows. It separates drones by departure time (headway) and minimal altitude slots, and writes PX4 `.plan` files. | [README](src/planner/headway_planner/README.md) |
| `src/planner/orca_planner` | ORCA (Optimal Reciprocal Collision Avoidance) in 3D. Computes collision-free trajectories offline for environments without obstacles. | [README](src/planner/orca_planner/README.md) |
| `src/planner/dl_planner` | Deep-learning offline planner for drone shows. A neural network predicts collisions between drones. | [README](src/planner/dl_planner/README.md) |
| `src/planner/rl_planner` | Reinforcement-learning planner (PPO, MAPPO). | [README](src/planner/rl_planner/README.md) |
| `src/planner/layer_planner` | Layer-based collision avoidance. | — |
| `src/planner/catora_planner` | CAT-ORA formation reshaping, ported from ROS1 Noetic to ROS2 Humble. Collision-aware, time-optimal assignment and trajectories (it minimizes the longest trajectory duration). | [README](src/planner/catora_planner/README.md) |
| `src/planner/traj_recorder` | Records the ego-planner B-spline trajectories of several drones to TXT files at the same time. | [README](src/planner/traj_recorder/README.md) |
| `src/planner/drone_detect` | Detects other drones in depth images and computes their pose error in the observer drone's world frame. | [README](src/planner/drone_detect/README.md) |
| `scripts/Allocation` | Drone-to-target assignment (Fair Hungarian, CAT-ORA) and saved assignments for 25 and 36 drones. | [README](scripts/Allocation/README.md) |

## Repository layout

```
.
├── src/
│   ├── planner/          # the planners above and ego-planner support packages
│   │                     #   (plan_env, path_searching, bspline_opt, traj_utils, ...)
│   └── uav_simulator/    # quadrotor simulator and controller, map generators
│                         #   (map_generator, mockamap), sensing and utilities
├── scripts/              # assignment, trajectory sampling and formation-planning scripts
├── docs/                 # trajectory sampling guide; older notes in docs/archived/
├── TODO.md, KEY_FILES_REFERENCE.md, CATORA_*.md    # documentation (see below)
└── LICENSE               # GPL-3.0
```

The scripts are described in [scripts/README.md](scripts/README.md).

## Documentation

Of the documents below, this README and KEY_FILES_REFERENCE.md are in English; the others are in Korean.

### Main documentation
- **README.md** — this file (quick start guide)
- **[TODO.md](TODO.md)** — project TODO list and status
- **[KEY_FILES_REFERENCE.md](KEY_FILES_REFERENCE.md)** — key files reference for collision avoidance
- **[docs/TRAJECTORY_SAMPLING_GUIDE.md](docs/TRAJECTORY_SAMPLING_GUIDE.md)** — how `traj_recorder_node` samples ego-planner B-spline trajectories and saves them in the DroneShow format

### CAT-ORA integration
- **[CATORA_INTEGRATION_SUMMARY.md](CATORA_INTEGRATION_SUMMARY.md)** — complete CAT-ORA integration summary
- **[CATORA_PLANNER_GUIDE.md](CATORA_PLANNER_GUIDE.md)** — CAT-ORA planner usage guide

### Archived documentation
See [docs/archived/](docs/archived/) for historical documentation:
- [CLEANUP_PLAN.md](docs/archived/CLEANUP_PLAN.md)
- [OFFLINE_TRAJECTORY_PLANNING.md](docs/archived/OFFLINE_TRAJECTORY_PLANNING.md) — offline trajectory planning experiments. With 36 drones, a 70 m planning horizon (replanning off) gave no crashes and 34 of 36 drones ready (94%).
- [TRAJECTORY_SAMPLING.md](docs/archived/TRAJECTORY_SAMPLING.md)
- [QUICK_SAMPLING_GUIDE.md](docs/archived/QUICK_SAMPLING_GUIDE.md)

## Manual installation

### 1. Required libraries
* ROS 2 Humble
* vtk (a dependency library for installing PCL; you need to check Qt during compilation)
* PCL

### 2. Prerequisites: switch the DDS to Cyclone DDS
With ROS2's default FastDDS, the program lags significantly while it runs. The reason hasn't been identified yet; it might be due to some incorrect settings in my publish/subscribe configurations. Follow the steps below to change the DDS to cyclonedds.

#### 2.1 Install cyclonedds
```bash
sudo apt install ros-humble-rmw-cyclonedds-cpp
```

#### 2.2 Change the default DDS
```bash
echo "export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp" >> ~/.bashrc
source ~/.bashrc
```

#### 2.3 Verify the change
```bash
ros2 doctor --report | grep "RMW middleware"
```
If the output shows rmw_cyclonedds_cpp, the change worked.

### 3. Clone and build the workspace
Clone the repository and build the workspace from its root:
```bash
git clone https://github.com/silverleopard12/drone-show.git
cd drone-show
colcon build --symlink-install
source install/setup.bash
```
Run `source install/setup.bash` again in every new terminal.

## Using Docker (alternative)

The Docker image is not publicly available on Docker Hub at the moment.

### 1. Clone the repository
```bash
git clone https://github.com/silverleopard12/drone-show.git
cd drone-show
```

### 2. Pull the Docker image
```bash
docker pull ds70768/droneshow_trajectory_docker:latest
```

### 3. Allow X server access (for GUI/RViz)
```bash
xhost +local:docker
```

### 4. Run the Docker container
```bash
docker run -it --rm \
    --network host \
    --privileged \
    -e DISPLAY=$DISPLAY \
    -e QT_X11_NO_MITSHM=1 \
    -v $(pwd):/workspace/ego-planner-swarm \
    -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
    ds70768/droneshow_trajectory_docker:latest
```

### 5. Build the workspace inside the container
```bash
cd /workspace/ego-planner-swarm
colcon build --symlink-install
source install/setup.bash
```

### 6. Run the planner
See [Running the code](#running-the-code) below for detailed instructions.

## Running the code

### 1. Launch RViz
```bash
ros2 launch ego_planner rviz.launch.py
```

### 2. Run the planning program
Open a new terminal and run one of the launch files below.

**Docker users:** in the new terminal, enter the running container first:
```bash
docker exec -it <container_name> bash
cd /workspace/ego-planner-swarm
source install/setup.bash
```

**Available launch options:**

* **Single drone**
```bash
ros2 launch ego_planner single_run_in_sim.launch.py
```

* **Swarm**
```bash
ros2 launch ego_planner swarm.launch.py
```

* **Large swarm**
```bash
ros2 launch ego_planner swarm_large.launch.py
```

* **Additional parameters (optional):**
    * `use_mockamap`: map generation method. Default: False (uses Random Forest); True uses mockamap.
    * `use_dynamic`: whether to consider dynamics. Default: False (disabled); True enables dynamics.

**Example with parameters:**
```bash
ros2 launch ego_planner single_run_in_sim.launch.py use_mockamap:=True use_dynamic:=False
```

## Related papers and repositories

- **Ego-Planner swarm**: https://github.com/ZJU-FAST-Lab/ego-planner-swarm
- **CAT-ORA**, ported to ROS2 Humble in [src/planner/catora_planner](src/planner/catora_planner/README.md)
  - Paper: V. Kratky, R. Penicka, J. Horyna, P. Stibinger, T. Baca, M. Petrlik, P. Stepan, M. Saska, "CAT-ORA: Collision-Aware Time-Optimal Formation Reshaping for Efficient Robot Coordination in 3-D Environments," *IEEE Transactions on Robotics*, vol. 41, pp. 2950–2969, 2025. [doi:10.1109/TRO.2025.3547296](https://doi.org/10.1109/TRO.2025.3547296) · [arXiv:2412.00603](https://arxiv.org/abs/2412.00603)
  - Original code: https://github.com/ctu-mrs/catora

## License

GNU General Public License v3.0. See [LICENSE](LICENSE).
