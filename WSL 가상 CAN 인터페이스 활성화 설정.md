1. [필수] WSL 가상 CAN 인터페이스 활성화 설정

- WSL 창을 열고 아래 명령어를 순서대로 입력하여 가상 CAN 네트워크를 활성화해야 합니다.
- 이 작업이 선행되지 않으면 파이썬의 vcan0 버스 선언이나 C++의 소켓 바인딩이 에러 없이 먹통이 됩니다.

#
1. vcan 커널 모듈 로드
2. vcan0 가상 인터페이스 생성 (이미 존재한다는 에러가 나면 패스해도 됨)
3. vcan0 인터페이스 활성화 (Up)
#

Bash

# 커널 모듈 로드 및 가상 CAN 인터페이스 생성

sudo modprobe vcan

sudo ip link add dev vcan0 type vcan

sudo ip link set up vcan0

# 인터페이스가 정상적으로 켜졌는지 확인 (vcan0가 보이면 성공)
ifconfig vcan0
