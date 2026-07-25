MINI_MES 작업영역을 검토해서, 전체적인 기획안을 만들어주고, 앞으로 확장해나갈 방향을 제시해줘
또한, OOP로 방향을 잡아야한다면 그 이유에 대해 설명해줘






오전 10:52






3m 22s 동안 작업



결론
현재 MINI_MES는 MES 핵심 개념을 학습하고 시연하기 위한 프로토타입으로는 방향이 좋습니다. 이미 다음 흐름을 갖추고 있습니다.
원자재 입고 → BOM 기반 생산 등록 → 완제품 LOT 생성 → 원자재 투입 이력 → 정·역방향 추적 → 생산 현황

다만 실제 현장 운영에 사용하려면 신규 기능보다 먼저 데이터 무결성과 재고 정확성을 안정화해야 합니다. 현재 상태에서 기능만 확장하면 잘못된 재고와 불완전한 LOT 추적 데이터가 누적될 가능성이 큽니다.
1. 현재 프로젝트 평가
영역	평가	설명
업무 흐름	양호	입고·생산·LOT 추적의 기본 흐름이 연결되어 있음
화면 구성	보통	조회·등록·대시보드가 있으나 실행 진입점과 페이지 구성이 혼재
코드 구조	보통	UI, 쿼리, 서비스가 일부 분리되어 확장 기반은 있음
데이터 모델	초기 단계	품목·BOM·LOT·생산 중심이며 재고 이동·품질·설비 모델은 없음
데이터 무결성	위험	실제 DB에 외래키 위반과 추적 이력 누락이 존재
테스트·배포	미흡	테스트, 의존성 정의, 마이그레이션, README가 없음
현장 운영성	미흡	사용자·권한·감사 로그·동시성·백업 정책이 없음

잘된 부분
DB 접근이 [src/db.py](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/src/db.py)로 모여 있습니다.
조회 SQL이 대부분 매개변수 방식이어서 기본적인 SQL 주입 위험을 줄였습니다.
생산 등록 시 완제품 LOT, 생산실적, 원자재 투입 이력을 한 트랜잭션으로 저장합니다.
[src/services.py (line 123)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/src/services.py:123)에 UI와 저장 로직을 분리하려는 방향이 보입니다.
dataclass를 이미 사용하고 있어 향후 도메인 모델로 확장하기 좋습니다.
정방향·역방향 LOT 추적이라는 MES의 중요한 가치를 이미 구현했습니다.
2. 우선 해결해야 할 문제
P0: 실제 DB 무결성 문제
실제 사용 DB인 [sql/m_mes_ramen.db](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/sql/m_mes_ramen.db)를 검사한 결과:
외래키 위반 14건
생산실적 31건 중 21건에 원자재 투입 이력이 없음
원자재 입고량보다 계산상 사용량이 많은 LOT 2건면 블록: -6,500
매운맛 스프: -5,500

일부 생산 투입 이력이 존재하지 않는 생산 또는 LOT를 참조
BOM의 bom_id가 모두 NULL
따라서 현재 추적 결과와 재고 수량은 운영 데이터로 신뢰하기 어렵습니다.
P0: DB 초기화 SQL 문제
[sql/schema.sql (line 9)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/sql/schema.sql:9)은 item보다 bom을 먼저 생성하고, [sample_data.sql (line 4)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/sql/sample_data.sql:4)도 품목보다 BOM을 먼저 입력합니다.
외래키를 활성화한 새 DB에서 초기화하면 BOM 입력이 실패합니다. 또한:
bom_id INT AUTO_INCREMENT PRIMARY KEY
은 MySQL 문법에 가깝고 SQLite의 올바른 자동 증가 키가 아닙니다. SQLite에서는 일반적으로 다음처럼 정의해야 합니다.
bom_id INTEGER PRIMARY KEY
P0: 실행 기준점 혼재
루트의 m_mes_ramen.db, ramen_mes.db는 0바이트입니다.
실제 앱 공통 모듈은 sql/m_mes_ramen.db를 사용합니다.
하지만 [app.py (line 9)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/app.py:9)는 루트의 빈 DB에 직접 연결합니다.
실제 실행 검사에서 app.py는 no such table: item 오류가 발생했습니다.
app2.py, app3.py, app5.py, app_00.py, 000.py, db2.py 같은 실험 파일이 운영 코드와 섞여 있습니다.
[pages/05_생산이력.py](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/pages/05_생산이력.py)는 빈 파일입니다.
하나의 공식 실행점과 하나의 DB 경로를 확정해야 합니다.
P1: 재고가 실제로 차감되지 않음
[생산등록 화면 (line 158)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/pages/04_생산등록.py:158)에도 명시된 것처럼 생산 시 원자재 재고를 차감하지 않습니다.
현재 자동 LOT 선택도 LOT 번호가 가장 빠른 행 하나를 고를 뿐입니다. 다음 조건이 없습니다.
실제 잔량
필요량 충족 여부
유효기한
품질검사 합격 여부
FIFO/FEFO
여러 LOT 분할 투입
이 상태에서는 생산 이력은 만들어져도 재고 장부는 맞지 않습니다.
기타 개선점
원자재 입고 화면에 유효기한 입력이 중복되어 있습니다: [03_원자재입고.py (line 54)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/pages/03_원자재입고.py:54)
생산번호의 날짜는 선택한 생산일자가 아니라 시스템 오늘 날짜를 사용합니다: [04_생산등록.py (line 47)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/pages/04_생산등록.py:47)
MAX(id) + 1 방식은 동시 등록 시 충돌할 수 있습니다.
Streamlit 1.59.2에서 제거 예정인 use_container_width를 사용합니다: [src/ui.py (line 38)](/mnt/s/내 드라이브/AI 융합 스마트 팩토리/work/mini_MES/src/ui.py:38)
테스트, 마이그레이션 도구, 의존성 파일, 실행 설명서가 없습니다.
3. MINI_MES 전체 기획안
제품 목표
“소규모 식품 생산 공정에서 원자재 입고부터 완제품 생산과 LOT 추적까지 관리하는 경량 MES”
주요 사용자
생산 작업자: 작업지시 확인 및 생산실적 등록
자재 담당자: 입고·출고·LOT·재고 관리
생산 관리자: 계획 대비 실적과 생산 진행상태 확인
품질 담당자: 입고검사·공정검사·완제품검사·부적합 처리
시스템 관리자: 품목·BOM·사용자·설비 기준정보 관리
목표 업무 흐름
품목·BOM 관리
    ↓
원자재 입고 → 입고검사 → 가용재고
    ↓
생산계획 → 작업지시 → 원자재 LOT 할당
    ↓
자재투입 → 생산실적 → 양품·불량 등록
    ↓
완제품 LOT → 품질판정 → 재고입고
    ↓
출하 → 고객/출하 LOT 추적
권장 기능 모듈
기준정보
품목
BOM 및 BOM 버전
공정·라우팅
창고·보관위치
설비·작업장
사용자·권한

자재·재고
원자재 입고
입고검사 및 합격/보류/불합격 상태
재고 이동 장부
LOT별 가용·할당·보류 수량
FIFO/FEFO 출고
유효기한 임박 알림

생산관리
생산계획
작업지시 발행
작업 시작·중지·완료
원자재 LOT 할당 및 실제 투입
양품·불량·재작업 수량
완제품 LOT 자동 발행

품질관리
입고검사
공정검사
완제품검사
불량유형과 원인
LOT 보류·해제·폐기

추적성
원자재 → 생산 → 완제품 정방향 추적
완제품 → 원자재 역방향 추적
출하처 연결
리콜 대상 LOT 및 수량 산출
추적 데이터 완전성 검사

대시보드
계획 대비 실적
품목별 생산량
수율과 불량률
재고 부족 및 유효기한 임박
작업지시 진행상태
LOT 추적 완전성

설비 가동시간과 비가동 사유 데이터가 확보되기 전에는 OEE를 표시하지 않는 것이 좋습니다.
4. 단계별 확장 로드맵
단계	목표	핵심 작업
0단계	기반 안정화	DB 복구, 스키마 수정, 공식 실행점 확정, 실험 파일 분리, 테스트 환경 구축
1단계	재고 정확성	inventory_transaction 도입, 잔량 계산, FEFO 할당, 다중 LOT 투입
2단계	생산 실행	생산계획·작업지시·상태 전환·양품/불량·작업자 등록
3단계	품질·추적	검사결과, LOT 보류, 출하 연결, 리콜 범위 계산
4단계	운영 시스템화	로그인, 역할별 권한, 감사 로그, PostgreSQL, 백업·모니터링
5단계	스마트팩토리 연계	PLC·센서 데이터, 설비상태, 비가동 분석, 예지보전·AI 분석

가장 먼저 추가할 테이블
inventory_transaction
production_order
production_result
quality_inspection
warehouse
location
user
audit_log
lot.qty를 직접 수정하는 방식보다 모든 증감 내역을 inventory_transaction에 기록하고 합산하여 재고를 계산하는 방식이 추적성과 감사 측면에서 좋습니다.
5. OOP 방향에 대한 판단
결론: 선택적으로 OOP를 적용하는 것이 좋습니다
현재 코드를 전부 클래스로 다시 만드는 것은 권장하지 않습니다. Streamlit 페이지와 단순 조회 함수는 지금처럼 절차형 코드가 더 읽기 쉽습니다.
대신 다음 영역에는 OOP가 필요해질 가능성이 높습니다.
생산 등록과 취소
재고 할당과 차감
FIFO/FEFO 정책
품질 판정
LOT 번호 발행
트랜잭션 관리
SQLite에서 PostgreSQL로의 전환
OOP가 필요한 이유
MES 규칙은 단순 CRUD보다 “상태와 규칙”이 중요하기 때문입니다.
예를 들면:
유효기한이 지난 LOT는 투입할 수 없다.
검사 합격 LOT만 사용 가능하다.
가용재고보다 많이 투입할 수 없다.
생산 완료 시 원자재 출고와 완제품 입고가 함께 성공해야 한다.
완료된 생산은 직접 삭제하지 않고 취소 트랜잭션을 만들어야 한다.
이 규칙을 화면과 SQL 함수에 흩어 놓으면 페이지마다 검증이 달라집니다. 도메인 객체와 서비스에 모으면 모든 화면이 같은 규칙을 사용하고 단위 테스트도 쉬워집니다.
권장 구조
streamlit_app.py
app_pages/                 화면
application/               업무 흐름
    production_service.py
    inventory_service.py
    traceability_service.py
domain/                    MES 규칙과 모델
    lot.py
    production_order.py
    inventory.py
    policies.py
infrastructure/            DB 구현
    sqlite/
    repositories/
    unit_of_work.py
tests/
migrations/
권장 클래스와 역할은 다음 정도면 충분합니다.
ProductionService: 생산 등록·완료·취소
InventoryService: 입고·할당·투입·재고 조정
TraceabilityService: 정방향·역방향·리콜 추적
LotAllocationPolicy: LOT 할당 규칙
FEFOPolicy: 유효기한 우선 할당 구현
LotRepository, ProductionRepository: DB 접근 인터페이스
SqliteUnitOfWork: 트랜잭션 관리
LotNumberGenerator: LOT 및 생산번호 발행
상속 계층을 깊게 만드는 것보다 dataclass, Enum, Protocol과 객체 조합을 사용하는 것이 적합합니다.
최종 권고
기능 확장보다 먼저 다음 순서로 진행하는 것이 안전합니다.
현재 DB 백업 및 무결성 복구
streamlit_app.py 하나를 공식 실행점으로 지정
스키마·샘플 데이터·DB 생성 절차 수정
재고 이동 장부와 FEFO LOT 할당 구현
핵심 서비스부터 선택적으로 OOP 적용
단위 테스트와 DB 통합 테스트 추가
이후 작업지시·품질·출하 기능 확장
이번 검토에서는 코드를 변경하지 않았습니다. 현재 프로젝트는 버릴 단계가 아니라, 교육용 프로토타입에서 운영 가능한 경량 MES로 넘어가기 직전의 단계로 판단됩니다.
