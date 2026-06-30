- 가장 안전한 방법은 가상환경을 만들어서 그 안에 설치하는 겁니다.


# Bash
1. venv 생성

    python3 -m venv ~/venvs/influx

2. 가상환경 활성화

    source ~/venvs/influx/bin/activate

3. 패키지 설치

    pip install influxdb-client

4. 확인

    python -c "from influxdb_client import InfluxDBClient; print('OK')"



