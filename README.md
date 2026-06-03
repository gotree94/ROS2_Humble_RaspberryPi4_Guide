# 라즈베리파이 4B 8GB - ROS2 Humble 설치 가이드

**작성일**: 2026년 6월  
**대상**: Raspberry Pi 4 Model B 8GB  
**ROS2 버전**: Humble Hawksbill (Ubuntu 22.04 기반)

이 문서는 Raspberry Pi 4B에서 ROS2 Humble을 설치하는 **주요 4가지 방법**을 상세히 설명합니다.

---

1. 설치 관련 파일
   * [ROS2_Humble_Installation_Methods.md](ROS2_Humble_Installation_Methods.md)
   * [ROS2_Installation_Verification.md](ROS2_Installation_Verification.md)

2. 최적화 & 모니터링
   * [ROS2_Performance_Optimization.md](ROS2_Performance_Optimization.md)
   * [ROS2_Monitoring_Tools.md](ROS2_Monitoring_Tools.md)

3. 시각화 도구
   * [ROS2_Visualization_Tools_Comparison.md](ROS2_Visualization_Tools_Comparison.md)

4. QoS 관련
   * [ROS2_QoS_Guide.md](ROS2_QoS_Guide.md)
   * [ROS2_QoS_Examples_Package.md](ROS2_QoS_Examples_Package.md)

5. Foxglove & 압축
   * [ROS2_Foxglove_Guide.md](ROS2_Foxglove_Guide.md)
   * [ROS2_Message_Compression.md](ROS2_Message_Compression.md)

---

## 목차
1. [방법 1: Ubuntu 22.04 Native 설치](#방법-1-ubuntu-2204-native-설치)
2. [방법 2: Pre-built Image 사용](#방법-2-pre-built-image-사용)
3. [방법 3: Raspberry Pi OS + Docker](#방법-3-raspberry-pi-os--docker)
4. [방법 4: Docker 중심 설치 (Compose 포함)](#방법-4-docker-중심-설치)
5. [micro-ROS 설치 및 연동](#micro-ros-설치-및-연동)
6. [공통 설정 및 팁](#공통-설정-및-팁)

---

## 방법 1: Ubuntu 22.04 Native 설치 (가장 추천)

### 장단점
- **장점**: 최고의 성능, 공식 지원, 디버깅 용이, micro-ROS 연동 편리
- **단점**: Ubuntu Desktop은 RAM을 많이 사용 (Server 추천)

### 설치 단계

1. **이미지 다운로드 및 플래싱**
   ```bash
   # Raspberry Pi Imager 사용
   # OS 선택 → Other general purpose OS → Ubuntu → Ubuntu 22.04.XX (64-bit)
   # Server 버전 강력 추천
   ```

2. **라즈베리파이 부팅 후 기본 설정**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y locales
   sudo locale-gen en_US en_US.UTF-8
   sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
   export LANG=en_US.UTF-8
   ```

3. **ROS2 Humble 설치**
   ```bash
   # ROS2 저장소 추가
   sudo apt install -y software-properties-common
   sudo add-apt-repository universe
   sudo apt update && sudo apt install -y curl
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

   sudo apt update
   sudo apt install -y ros-humble-ros-base   # 기본 설치 (추천)
   # sudo apt install -y ros-humble-desktop   # GUI 필요 시 (8GB에서 가능)
   ```

4. **개발 환경 설정**
   ```bash
   sudo apt install -y python3-colcon-common-extensions python3-rosdep python3-vcstool
   sudo rosdep init
   rosdep update

   # .bashrc에 추가
   echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```

### micro-ROS 연동
```bash
sudo apt install -y ros-humble-micro-ros-agent ros-humble-micro-ros-msgs
```

---

## 방법 2: Pre-built Image 사용 (가장 빠름)

### 추천 이미지
- **ros-realtime/ros-realtime-rpi4-image** (Ubuntu 22.04 + ROS2 Humble + Real-time Kernel)

### 설치 방법
1. [GitHub Repository](https://github.com/ros-realtime/ros-realtime-rpi4-image) 방문
2. Latest Release에서 `.img.xz` 파일 다운로드
3. balenaEtcher 또는 Raspberry Pi Imager로 SD카드에 플래싱
4. 부팅 후 `pi` 계정으로 로그인 (기본 비밀번호 확인 필요)

### 장점
- Real-time kernel 포함 (제어 성능 우수)
- ROS2 Humble + 주요 패키지 미리 설치됨
- 바로 사용 가능

### 단점
- 이미지 크기가 크고, 커스터마이징이 상대적으로 어려움

---

## 방법 3: Raspberry Pi OS + Docker

### 장점
- Raspberry Pi 하드웨어 지원 (CSI 카메라, HAT 등) 최고
- 호스트 OS는 가볍게 유지

### 설치 단계

1. **Raspberry Pi OS 64-bit 설치**
   - Raspberry Pi Imager → Raspberry Pi OS (64-bit) → Bookworm

2. **Docker 설치**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   # 재로그인
   ```

3. **ROS2 Humble Docker 이미지 실행**
   ```bash
   # 기본 ROS2 Humble
   docker run -it --rm \
     --net=host \
     --privileged \
     osrf/ros:humble-desktop \
     bash

   # 또는 docker-compose 사용 추천
   ```

### Docker Compose 예시 (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  ros2:
    image: osrf/ros:humble-desktop
    container_name: ros2_humble
    privileged: true
    network_mode: host
    environment:
      - DISPLAY=${DISPLAY}
    volumes:
      - /dev:/dev
      - ./ros2_ws:/root/ros2_ws
    command: bash
```

---

## 방법 4: Docker 중심 설치 (Compose + Custom Image)

### 추천 상황
- 여러 프로젝트 분리
- 개발/배포 환경 철저히 분리
- micro-ROS Agent를 별도 컨테이너로 운영

### Custom Dockerfile 예시
```dockerfile
FROM osrf/ros:humble-ros-base

RUN apt-get update && apt-get install -y \
    python3-colcon-common-extensions \
    ros-humble-micro-ros-agent \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /root/ros2_ws
```

### micro-ROS Agent Docker
```bash
docker run -it --rm \
  --net=host \
  microros/micro-ros-agent:humble \
  serial --dev /dev/ttyACM0 -b 115200
```

---

## micro-ROS 설치 및 연동

### 1. micro-ROS Agent 설치 (Ubuntu Native)
```bash
# Humble 환경에서
source /opt/ros/humble/setup.bash
sudo apt install -y ros-humble-micro-ros-agent

# Agent 실행 (Serial)
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0 -b 115200
```

### 2. Firmware 빌드 (ESP32 / Pico)
```bash
# micro-ros setup
git clone --recursive https://github.com/micro-ROS/micro_ros_setup.git
cd micro_ros_setup
./install.sh
source install/local_setup.bash

# Firmware 생성
ros2 run micro_ros_setup create_firmware_ws.sh build farm
```

---

## 공통 설정 및 팁

### 성능 최적화
```bash
# Swap 증가 (8GB 모델)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### ROS2 Workspace 생성
```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```

### 유용한 명령어
- `ros2 doctor` : 시스템 진단
- `ros2 topic list` : 토픽 확인
- `htop` : 리소스 모니터링

### 주의사항
- Raspberry Pi 4B는 발열이 심하니 **히트싱크 + 팬** 권장
- 8GB 모델이라도 `ros-humble-desktop` + GUI 사용 시 RAM 관리가 중요
- micro-ROS 사용 시 baudrate와 serial 권한(`sudo usermod -aG dialout $USER`) 확인

---

**문의사항 있으시면 언제든 말씀해주세요!**

이 가이드를 기반으로 원하는 방법으로 설치하시면 됩니다.
```

---

이제 파일이 생성되었습니다. 다운로드 받으실 수 있습니다.
