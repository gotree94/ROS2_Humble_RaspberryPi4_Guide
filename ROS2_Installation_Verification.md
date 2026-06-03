# ROS2 Humble Installation Verification Guide

## 방법 1: Ubuntu Native
- `ros2 --version`
- `ros2 doctor`
- talker/listener 테스트

## 방법 2: Pre-built Image
- Kernel 확인 (`uname -a`)
- ROS2 기본 노드 실행

## 방법 3: Docker
- Container 내부 `ros2 doctor`

## 방법 4: Docker Compose
- `docker compose ps`
- Topic list 확인
