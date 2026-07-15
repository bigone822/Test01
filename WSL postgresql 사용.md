WSL(Windows Subsystem for Linux) 환경(Ubuntu 기준)에서 PostgreSQL을 설치하고 실행하는 방법입니다. WSL은 Windows와 파일 시스템 및 네트워크 구성이 조금 다르기 때문에, Linux 본래의 설치 과정을 따르되 WSL 전용 명령어를 몇 가지 알아두면 편리합니다.

1. 패키지 목록 업데이트 및 설치
먼저 WSL 터미널을 열고 패키지 저장소를 최신 상태로 업데이트한 뒤, PostgreSQL과 추가 도구(contrib) 패키지를 설치합니다.

```Bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```
2. PostgreSQL 서비스 시작
WSL은 일반적인 Linux와 달리 systemd가 기본적으로 활성화되어 있지 않은 경우가 많습니다 (최근 WSL2는 지원하지만 설정이 필요함). 따라서 가장 확실하고 전통적인 service 명령어를 사용하여 실행하는 것이 좋습니다.

서비스 시작:

```Bash
sudo service postgresql start
```
서비스 상태 확인:

```Bash
sudo service postgresql status
```
online 또는 active (exited/running) 상태로 표시되면 정상적으로 구동된 것입니다.

3. PostgreSQL 접속 및 초기 설정
PostgreSQL을 설치하면 기본적으로 postgres라는 이름의 시스템 유저와 DB 최고 관리자(Superuser) 계정이 자동으로 생성됩니다.

3.1. 관리자 계정(postgres)으로 접속
아래 명령어를 입력하여 postgres 유저로 전환함과 동시에 대화형 쉘(psql)을 실행합니다.

```Bash
sudo -i -u postgres psql
```
접속에 성공하면 프롬프트가 postgres=# 형태로 바뀝니다.

3.2. 관리자 비밀번호 설정
보안과 외부 툴(DBeaver, pgAdmin 등) 연동을 위해 관리자 계정의 비밀번호를 설정해 줍니다.

```SQL
ALTER USER postgres PASSWORD '원하는_비밀번호';
```
주의: SQL 명령어 끝에는 반드시 세미콜론(;)을 붙여야 실행됩니다.

3.3. 테스트용 데이터베이스 및 유저 생성 (권장)
실제 개발 시에는 postgres 계정을 그대로 쓰기보다 전용 유저와 DB를 만들어 쓰는 것이 좋습니다.

```SQL
-- 유저 생성
CREATE USER myuser WITH PASSWORD 'mypassword';

-- 데이터베이스 생성
CREATE DATABASE mydb OWNER myuser;

-- 권한 부여
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```
3.4. psql 종료
데이터베이스 쉘에서 나가려면 다음 명령어를 입력합니다.

```SQL
\q
```
4. Windows 툴(DBeaver, pgAdmin 등)에서 WSL PostgreSQL 연결하기
WSL 내부에 설치된 PostgreSQL은 Windows 호스트와 네트워크를 공유하므로, Windows에 설치된 DB 관리 툴에서 쉽게 접속할 수 있습니다.
```text
Host: localhost 또는 127.0.0.1

Port: 5432 (기본값)

Database: postgres (또는 새로 만든 데이터베이스 이름)

Username: postgres
```
Password: 3.2단계에서 설정한 비밀번호

💡 만약 Windows에서 연결이 안 된다면?
PostgreSQL 설정 파일(postgresql.conf, pg_hba.conf)에서 외부 접근 허용 설정을 해주어야 할 수 있습니다. 일반적으로 WSL2의 localhost 포트 포워딩 기능 덕분에 별도 설정 없이 작동하지만, 오류가 발생하면 /etc/postgresql/{버전}/main/ 경로의 설정 파일들을 확인해 보세요.

5. 자주 쓰는 명령어 요약

작업	명령어
서비스 시작	sudo service postgresql start
서비스 중지	sudo service postgresql stop
서비스 재시작	sudo service postgresql restart
관리자 접속	sudo -i -u postgres psql
일반 유저 접속	psql -U 유저명 -d DB명 -h localhost
설치 중 막히는 부분이 있거나 특정 에러 메시지가 뜨면 언제든 말씀해 주세요!
