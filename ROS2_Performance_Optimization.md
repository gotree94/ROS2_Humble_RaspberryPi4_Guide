# ROS2 Humble Performance Optimization Tips (Raspberry Pi 4B)

## 1. OS & Hardware 1. 하드웨어 / OS 레벨 최적화
- Ubuntu Server 사용
- Preempt_RT Kernel
- CPU Affinity
- ZRAM

Ubuntu 22.04 Server 사용
Desktop 대신 Server 이미지를 사용하세요. GUI가 없어 RAM/CPU를 크게 절약할 수 있습니다.
Preempt_RT Kernel (Real-time Kernel) 적용
Real-time 성능(저지연, jitter 감소)이 중요하다면 ros-realtime 프로젝트의 Pre-built Image를 사용하거나, 직접 RT 패치를 적용하세요.
→ 메시지 지연이 크게 줄어듭니다.
Overclocking (선택)
/boot/config.txt에서 CPU/GPU 오버클럭:Basharm_freq=1800
gpu_freq=750
over_voltage=6(안정성을 위해 cooling fan 필수, 8GB 모델에서 효과 좋음)
CPU Isolation & Affinity
중요한 ROS2 노드를 특정 코어에 고정:Bashtaskset -c 2,3 ros2 run my_package my_node또는 isolcpus 커널 파라미터로 코어 격리.
Swap & ZRAM
RAM 부족 시 ZRAM(압축 RAM) 활성화 추천. SD카드 Swap은 느리니 피하세요.

## 2. ROS2 Settings 2. ROS2 설정 최적화
- Cyclone DDS 사용
- Multi-threaded Executor vs Single
- QoS 최적화 (Best Effort)

RMW (Middleware) 변경
기본 Fast DDS 대신 Cyclone DDS 사용 (고주파/저지연에 유리):Bashsudo apt install ros-humble-rmw-cyclonedds-cpp
echo 'export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp' >> ~/.bashrc
Executor & Callback Group
Multi-threaded Executor 대신 Single-threaded + 적절한 Callback Group 사용
Reentrant Callback Group 적극 활용 (병렬 처리)
불필요한 intra-process communication 피하기

QoS 설정 최적화Pythonqos = rclpy.qos.QoSProfile(
    depth=10,
    reliability=rclpy.qos.ReliabilityPolicy.BEST_EFFORT,  # Real-time 시
    durability=rclpy.qos.DurabilityPolicy.VOLATILE
)이미지/포인트클라우드처럼 큰 데이터는 Best Effort + Keep Last 추천.
Zero-Copy Transport
Intra-process communication + Shared Memory 사용 (Fast DDS 설정).

## 3. Node Level 3. 노드 / 코드 레벨 최적화
- Component (Composition)
- Logging Level 낮추기
Node Composition (Component)
여러 노드를 하나의 프로세스로 합쳐 context switching 감소.
불필요한 Publishing 줄이기
publish_frequency 낮추기
transient_local Durability는 꼭 필요할 때만

메모리 관리
Large arrays (이미지 등)는 numpy + sensor_msgs_py 활용
Garbage Collection tuning (Python 노드)

Logging Level 낮추기Bashexport RCUTILS_LOGGING_BUFFERED_STREAM=1
ros2 run ... --ros-args --log-level WARN

4. Docker 환경에서 최적화 (RPi OS + Docker 사용 시)

--privileged, --shm-size=512m 옵션 사용
Host network mode (--network host)
GPU passthrough (필요 시)

5. micro-ROS 연동 시 최적화

UDP over Serial보다 UDP 또는 Best Effort QoS 사용
Agent를 별도 코어에 고정
Pico/ESP32 쪽에서는 High Priority Thread 사용

성능 확인 명령어
Bash# ROS2 시스템 상태
ros2 doctor
ros2 node list
ros2 topic hz /your_topic

# CPU / Memory 모니터링
htop
ros2 run system_metrics_collector system_metrics_collector

# Latency 측정
ros2 run performance_test performance_test --ros-args -p reliability:=best_effort

**자세한 명령어와 예시는 본문 참조**
