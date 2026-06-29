# 1. vcan 커널 모듈 로드
sudo modprobe vcan

# 2. vcan0 가상 인터페이스 생성 (이미 존재한다는 에러가 나면 패스해도 됨)
sudo ip link add dev vcan0 type vcan

# 3. vcan0 인터페이스 활성화 (Up)
sudo ip link set up vcan0
