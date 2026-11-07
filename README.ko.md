<!-- README.md(EN)와 README.ko.md(KO)는 함께 고친다 -->
[English](README.md) | **한국어**

# Ego-Planner Swarm

여러 계획 알고리즘으로 다중 드론 군집을 조율하는 ego-planner의 ROS2 Humble 구현이다.

## 개요

여러 드론이 함께 날 때, 예를 들어 드론쇼에서 한 형상이 다음 형상으로 바뀔 때는 드론마다 다른 드론과 부딪히지 않는 궤적이 있어야 한다. 이 작업 공간은 ego-planner 군집 플래너를 중심으로 그런 궤적을 만드는 여러 방법을 모았고, 드론을 형상의 점에 배정하는 도구와 궤적을 기록하는 도구도 함께 담았다.

| 폴더 | 하는 일 | 문서 |
|---|---|---|
| `src/planner/plan_manage` (ROS 패키지 `ego_planner`) | ego-planner 군집 플래너. 아래 '실행하기'의 launch 파일이 여기 있다. | — |
| `src/planner/headway_planner` | 드론쇼용 오프라인 플래너. 출발 시각 차이(headway)와 최소한의 고도 칸으로 드론을 떼어 놓고, PX4 `.plan` 파일을 만든다. | [README](src/planner/headway_planner/README.md) |
| `src/planner/orca_planner` | 3차원 ORCA(Optimal Reciprocal Collision Avoidance). 장애물이 없는 환경에서 충돌 없는 궤적을 오프라인으로 만든다. | [README](src/planner/orca_planner/README.md) |
| `src/planner/dl_planner` | 드론쇼용 딥러닝 오프라인 플래너. 신경망이 드론 사이의 충돌을 예측한다. | [README](src/planner/dl_planner/README.md) |
| `src/planner/rl_planner` | 강화학습 플래너(PPO, MAPPO). | [README](src/planner/rl_planner/README.md) |
| `src/planner/layer_planner` | 층(layer)으로 나누는 충돌 회피. | — |
| `src/planner/catora_planner` | CAT-ORA 형상 재배치를 ROS1 Noetic에서 ROS2 Humble로 옮긴 것. 충돌을 고려한 시간 최적 배정과 궤적을 만든다(가장 긴 궤적 시간을 최소로 한다). | [README](src/planner/catora_planner/README.md) |
| `src/planner/traj_recorder` | 여러 드론의 ego-planner B-spline 궤적을 동시에 TXT 파일로 기록한다. | [README](src/planner/traj_recorder/README.md) |
| `src/planner/drone_detect` | 깊이 영상에서 다른 드론을 찾고, 관측하는 드론의 월드 좌표계에서 그 드론의 자세 오차를 계산한다. | [README](src/planner/drone_detect/README.md) |
| `scripts/Allocation` | 드론-목표점 배정(Fair Hungarian, CAT-ORA)과 25대·36대 배정 결과. | [README](scripts/Allocation/README.md) |

## 폴더 구조

```
.
├── src/
│   ├── planner/          # 위 플래너들과 ego-planner 보조 패키지
│   │                     #   (plan_env, path_searching, bspline_opt, traj_utils 등)
│   └── uav_simulator/    # 쿼드로터 시뮬레이터와 제어기, 지도 생성기
│                         #   (map_generator, mockamap), 센싱과 유틸리티
├── scripts/              # 배정, 궤적 샘플링, 형상 계획 스크립트
├── docs/                 # 궤적 샘플링 안내. 예전 기록은 docs/archived/
├── TODO.md, KEY_FILES_REFERENCE.md, CATORA_*.md    # 문서 (아래 참고)
└── LICENSE               # GPL-3.0
```

스크립트 설명은 [scripts/README.md](scripts/README.md)에 있다.

## 문서

아래 문서 가운데 KEY_FILES_REFERENCE.md만 영어이고, 나머지는 한국어다.

### 주요 문서
- **README.ko.md** — 이 파일(빠른 시작 안내)
- **[TODO.md](TODO.md)** — 할 일 목록과 진행 상태
- **[KEY_FILES_REFERENCE.md](KEY_FILES_REFERENCE.md)** — 충돌 회피 구현의 주요 파일 안내
- **[docs/TRAJECTORY_SAMPLING_GUIDE.md](docs/TRAJECTORY_SAMPLING_GUIDE.md)** — `traj_recorder_node`가 ego-planner의 B-spline 궤적을 샘플링해 DroneShow 형식으로 저장하는 방법

### CAT-ORA 통합
- **[CATORA_INTEGRATION_SUMMARY.md](CATORA_INTEGRATION_SUMMARY.md)** — CAT-ORA 통합 전체 요약
- **[CATORA_PLANNER_GUIDE.md](CATORA_PLANNER_GUIDE.md)** — CAT-ORA 플래너 사용 안내

### 보관 문서
예전 문서는 [docs/archived/](docs/archived/)에 있다.
- [CLEANUP_PLAN.md](docs/archived/CLEANUP_PLAN.md)
- [OFFLINE_TRAJECTORY_PLANNING.md](docs/archived/OFFLINE_TRAJECTORY_PLANNING.md) — 오프라인 궤적 계획 실험. 36대에서 planning horizon을 70 m로 두고 재계획을 끄면 충돌이 없었고 36대 중 34대가 준비를 마쳤다(94%).
- [TRAJECTORY_SAMPLING.md](docs/archived/TRAJECTORY_SAMPLING.md)
- [QUICK_SAMPLING_GUIDE.md](docs/archived/QUICK_SAMPLING_GUIDE.md)

## 직접 설치하기

### 1. 필요한 라이브러리
* ROS 2 Humble
* vtk (PCL을 설치하는 데 필요한 라이브러리. 컴파일할 때 Qt를 체크해야 한다)
* PCL

### 2. 준비: DDS를 Cyclone DDS로 바꾸기
ROS2 기본 DDS인 FastDDS를 쓰면 프로그램이 도는 동안 크게 끊긴다. 원인은 아직 찾지 못했고, 내 publish/subscribe 설정이 잘못되었기 때문일 수도 있다. 아래 순서로 DDS를 cyclonedds로 바꾼다.

#### 2.1 cyclonedds 설치
```bash
sudo apt install ros-humble-rmw-cyclonedds-cpp
```

#### 2.2 기본 DDS 바꾸기
```bash
echo "export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp" >> ~/.bashrc
source ~/.bashrc
```

#### 2.3 바뀌었는지 확인
```bash
ros2 doctor --report | grep "RMW middleware"
```
출력에 rmw_cyclonedds_cpp가 보이면 바뀐 것이다.

### 3. 레포 받기와 작업 공간 빌드
레포를 받고, 레포 맨 위 폴더에서 작업 공간을 빌드한다.
```bash
git clone https://github.com/silverleopard12/drone-show.git
cd drone-show
colcon build --symlink-install
source install/setup.bash
```
새 터미널을 열 때마다 `source install/setup.bash`를 다시 한다.

## Docker로 실행하기 (대안)

Docker 이미지는 지금 Docker Hub에 공개되어 있지 않다.

### 1. 레포 받기
```bash
git clone https://github.com/silverleopard12/drone-show.git
cd drone-show
```

### 2. Docker 이미지 받기
```bash
docker pull ds70768/droneshow_trajectory_docker:latest
```

### 3. X 서버 접근 허용 (GUI·RViz용)
```bash
xhost +local:docker
```

### 4. Docker 컨테이너 실행
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

### 5. 컨테이너 안에서 작업 공간 빌드
```bash
cd /workspace/ego-planner-swarm
colcon build --symlink-install
source install/setup.bash
```

### 6. 플래너 실행
자세한 방법은 아래 '실행하기'를 본다.

## 실행하기

### 1. RViz 띄우기
```bash
ros2 launch ego_planner rviz.launch.py
```

### 2. 계획 프로그램 실행
새 터미널을 열고 아래 launch 파일 가운데 하나를 실행한다.

**Docker를 쓰는 경우:** 새 터미널에서 먼저 실행 중인 컨테이너에 들어간다.
```bash
docker exec -it <container_name> bash
cd /workspace/ego-planner-swarm
source install/setup.bash
```

**고를 수 있는 launch 파일:**

* **드론 한 대**
```bash
ros2 launch ego_planner single_run_in_sim.launch.py
```

* **군집**
```bash
ros2 launch ego_planner swarm.launch.py
```

* **큰 군집**
```bash
ros2 launch ego_planner swarm_large.launch.py
```

* **추가 매개변수 (선택):**
    * `use_mockamap`: 지도 생성 방법. 기본값은 False(Random Forest 사용)이고, True면 mockamap을 쓴다.
    * `use_dynamic`: 동역학을 고려할지. 기본값은 False(끔)이고, True면 동역학을 고려한다.

**매개변수를 준 예:**
```bash
ros2 launch ego_planner single_run_in_sim.launch.py use_mockamap:=True use_dynamic:=False
```

## 관련 논문·레포

- **Ego-Planner swarm**: https://github.com/ZJU-FAST-Lab/ego-planner-swarm
- **CAT-ORA** — [src/planner/catora_planner](src/planner/catora_planner/README.md)에 ROS2 Humble로 옮겨 넣었다.
  - 논문: V. Kratky, R. Penicka, J. Horyna, P. Stibinger, T. Baca, M. Petrlik, P. Stepan, M. Saska, "CAT-ORA: Collision-Aware Time-Optimal Formation Reshaping for Efficient Robot Coordination in 3-D Environments," *IEEE Transactions on Robotics*, vol. 41, pp. 2950–2969, 2025. [doi:10.1109/TRO.2025.3547296](https://doi.org/10.1109/TRO.2025.3547296) · [arXiv:2412.00603](https://arxiv.org/abs/2412.00603)
  - 원본 코드: https://github.com/ctu-mrs/catora

## 라이선스

GNU General Public License v3.0. [LICENSE](LICENSE)를 본다.
