# 운영 관리

## PostgreSQL 운영 관리에 필요한 작업

* 유지 보수
	> DB는 매일 운영하여 내부 상태가 변화하고 있다. 항상 일정한 성능을 발휘하려면 좋은 상태를 유지하기 위한 유지 보수가 필요하다. VACUUM 및 ANALYZE

* 모니터링
	> 장애를 사전에 감지하거나 발생 후 원인을 조사하기 위해 DB 나 OS의 상태를 모니터링 한다. 적절한 문제 해결을 위해 시스템 리소스와 PostgreSQL을 실행 통계 정보의 취득이 필요하다.

* 백업 / 복원  
	> 정전 정도라면 트랜잭션 결과는 응급 복구 할 수 있지만 디스크의 고장이나 실수로 인한 데이터 손실에 대처하기 위해서는 백업이 필요하다. 복구 요구 사항이나 백업 / 복구에 걸리는 시간을 고려하여 백업 방식을 선택한다.
 
* 업그레이드 / 업데이트  
	> PostgreSQL의 새로운 릴리스에는 크게 2 가지가 있다. 버전 이름 (xyz)의 x 또는 y가 변화하는 메이저 릴리스 (8.3 → 8.4)와 z가 변화하는 부 릴리스 (8.3.6 → 8.3.7)이다. 부 릴리스는 호환성이 완벽하게 유지 한 채, 버그 나 보안 문제가 수정된다. 부 릴리스에 유연하게 할 수 있도록 한다.

## 운영하기 전에 할 일

* 로그 관련 설정  
	> 서버 로그는 어떤 문제가 있는 경우 먼저 확인해야 할 중요한 정보이다. 반드시 어딘가에 로그를 남기도록 설정한다.
 
* 실행 통계 관련 설정  
	> PostgreSQL은 스스로 DB 내부의 활동 상태를 모니터링하고, 그것을 테이블 정보로 축적하고 있다 (이 정보를 실행 통계라고 함). 매우 유용하기 때문에 꼭 가져 오도록 설정해 둔다.
 
* autovacuum  
	> autovacuum 기능은 테이블의 상태를 모니터링하고 그에 적합한 타이밍에 자동으로 VACUUM을 하는 기능이다. 8.3 이상 부터 기본 on. autovacuum을 지연시키면서하거나, 규정 시간 이상 걸린 경우에 기록하는 등의 설정도 가능하다.

## 일 단위로 해야할 일

VACUUM, ANALYZE는 autovacuum에게 맡기지만, 기타 정기적으로 작업을 수행하려면 별도 작업 실행 도구가 필요하다. UNIX / Linux는 crontab, Windows는 작업 스케줄러가 일반적이지만, 멀티 플랫폼에서 PostgreSQL 전용 pgAgent 도 사용할 수 있다.

* VACUUM  
	> PostgreSQL은 재기록 아키텍처로 인해, 업데이트 및 삭제 처리하여 DB 내부에 쓰레기가 발생한다. 쓰레기는 DB의 비대화와 캐시 이용 효율의 저하를 초래한다. 이 쓰레기를 회수하는 유지 보수가 VACUUM 이다. autovacuum에 맡겨 버릴 수도 있지만 VACUUM을 수동으로 할 경우 VERBOSE 옵션을 부여하여 소요 시간과 회수 한 쓰레기 량이 확인할 수 있기 때문에 유용하다.

* ANALYZE  
	> DBMS가 DB에 데이터를 검색 할 때 데이터 정렬 및 물리적 배치 등의 통계를 이용하여 가장 효율적인 방법으로 데이터를 검색한다. ANALYZE는 이 통계를 최신 데이터 상태를 기반으로 재생하는 명령이다. autovacuum 기능에 의해 자동으로 수행 할 수 있다.
 
* 시스템 리소스 검색  
	> DBMS 나 OS 등 소프트웨어가 소비하는 H/W 자원을 시스템 리소스라고 한다. 일반적으로 CPU 사용률 및 각 프로세스의 활동 상태 등의 정보이다. 이들은 기록하는 것이 아니라 문제 발생의 사전 파악 및 문제 발생시 원인의 엄선이 가능하다. 특히 아카이브 로그의 축적 및 DB 비대화에 대비해 디스크 사용율을 항상 확인한다.
 
* 백업  
	> 백업은 DB의 내용을 논리적인 형식 (SQL)으로 검색하여 가져오는 방법과 파일 시스템의 파일을 가져 오는 방법 크게 두 가지가 있다. 모두 PostgreSQL 실행 중에 할 수 있지만 각각의 차이가 있다.

	* 논리적 백업 (pg_dump)  
	pg_dump 를 사용하여 DB 데이터를 덤프한다. 일부 테이블 및 DB 스키마 데이터 내용만을 끄집어 선택이 가능하다. SQL의 형태로 데이터를 검색하고 주로 소규모의 DB 및 주 버전간에 마이그레이션 다른 DBMS로 마이그레이션 할 때 사용한다.

	* 온라인 백업 (full backup)  
	온라인 백업은 DB 클러스터를 rsync와 cp 명령을 사용하여 파일로 가져온다. DB 및 테이블 단위의 지정은 할 수없고 DB 클러스터 전체가 백업 된다. 아카이브 로그를 설정해 두는 것이 필요하다. 아카이브 로그와 함께 다운 직전까지 복구가 가능한 PITR이 필요한 경우에 사용된다.

## 월 단위로 해야할 일

* 월별 유지 보수  
	> 일상 유지 보수 외에 다음 명령을 실시하여 정기적으로 DB 상태를 최적화하는 것이 좋다. 다음 명령을 수행하는 동안 테이블이나 인덱스에 액세스 할 수 없게되므로 시스템을 정지 할 수 있는 정기 점검 범위를 확보하고 거기서 실시하면 좋을 것 이다.

* REINDEX  
	> 인덱스 다시 작성한다.
 
* CLUSTER  
	> 인덱스 순서로 테이블 데이터를 물리적으로 재구성한다. 테이블의 물리적 압축 + 재구성 + REINDEX의 효과가 있다. pg_reorg (CLUSTER를 온라인으로 할수 있는 툴)

* VACUUM FULL  
	> 테이블을 물리적으로 압축한다. DB가 비대화 되었을 때, 디스크의 사용량을 줄일 수 있다.

* 업그레이드 / 업데이트  
	* 업그레이드  
	PostgreSQL에서는 주요 버전에서 DB 클러스터와 호환되지 않는다. 따라서 메이저 버전이 다른 PostgreSQL로 마이그레이션 할 때 pg_upgrade 에 의한 변환하거나 pg_dump 에서 데이터를 추출하고 그것을 새로운 메이저 버전의 PostgreSQL로 이관하는 작업이 필요하다. 또한, 주요 버전에서는 PostgreSQL의 행동이 변화 할 수 있기 때문에 AP 검사 및 매개 변수의 재 설계 등이 필요하다.
	* 업데이트  
	PostgreSQL의 마이너 버전간에서는 DB 클러스터 호환성이 있기 때문에 기본적으로 업데이트만 하면 됩니다. AP에서 보이는 PostgreSQL의 동작은 원칙적으로는 변경되지 않는다. 보안 및 데이터 손상에 대한 문제를 해결할 수있는 일도 있기 때문에 업데이트는 가능한 한 적극적으로 할 수 있도록 한다.
 
## 비 정기적으로 해야할 일

어떠한 원인에 의해, DB 서버가 무거워 지거나 PostgreSQL가 반응하지 않는 경우에는 응급 처치가 필요하다. 시스템 다운 시간을 최대한 단축하기 위하여, 만일의 경우의 대처 방법은 준비해 둔다.

* 시스템 / PostgreSQL을 다시 시작  
	> OS가 중단 해 버리거나, PostgreSQL 프로세스가 다운 된 경우는 OS의 재부팅이나 PostgreSQL을 다시 시작한다.

* 장애  
	> 일반적으로 어떤 복제 도구와 Heartbeat 등의 감시 도구를 사용하여 Active, Standby 2 대의 서버를 준비해두고, Active 서버에 문제가 발생 하였을 경우에 신속하게 Standby 서버로의 전환 작업을 수행한다. 장애가 발생했을 때는 Active 서버에 기록 된 로그 등으로 부터 원인 규명을하고 즉시 새로운 Standby 시스템으로 사용할 수 있도록 한다.