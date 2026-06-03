# ROS2 Humble Installation Methods on Raspberry Pi 4B 8GB

## 1. Ubuntu 22.04 Native (Recommended)
**장점**: 최고 안정성, Tier 1 지원  
**단점**: Ubuntu Desktop은 무거움 (Server 추천)

### 설치 단계
```bash
# 1. Ubuntu 22.04 Server 64-bit 설치
# Raspberry Pi Imager 사용 추천

# 2. ROS2 Humble 설치
sudo apt update && sudo apt upgrade -y
sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update
sudo apt install ros-humble-ros-base -y   # 또는 ros-humble-desktop

# 3. Environment Setup
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 설치 확인
```bash
ros2 --version
ros2 doctor
ros2 run demo_nodes_cpp talker & ros2 run demo_nodes_py listener
```

---

## 2. Pre-built Real-time Image
- ros-realtime 프로젝트 이미지 사용
- Real-time Kernel 포함

### 설치 확인
```bash
uname -a | grep PREEMPT_RT
ros2 doctor
```

---

## 3. Raspberry Pi OS + Docker
- Raspberry Pi OS Bookworm 64-bit + Docker

### 설치 확인
```bash
docker run --rm osrf/ros:humble-desktop ros2 doctor
```

---

## 4. Docker 중심 설치
- Custom Dockerfile 또는 docker-compose

**자세한 내용은 별도 파일 참조**
