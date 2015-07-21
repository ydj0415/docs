# pg_statsinfo

pg_statsinfo는 PostgreSQL의 통계 및 활동을 감시하는 유틸리티 입니다. pg_statsinfo는 쉽게 설치가 가능하며, 레파지토리 서버에 타겟 PostgreSQL 서버의 활동 정보와 시스템 리소스에 대한 내용을 정기적으로 스냅샷을 찍어 저장합니다. 이러한 스냅샷을 pg_stats_reporter라는 도구를 이용해 Web 기반으로 시각화하여 보여줍니다. 이 도구는 PostgreSQL 데이터베이스 관리자를 위해 상태 체크 또는 쉽게 서버의 활동 정보를 분석합니다.

## 동작 방식

pg_statsinfo는 3가지 주요 부분으로 구성됩니다.

* pg_statsinfod : 에이전트 프로그램으로 대상서버의 스냅샷 정보를 주기적으로 레파지토리 서버에 보냅니다.
* repository database : 스냅샷의 순서에 맞게 수집정보를 저장하는 서버
* 