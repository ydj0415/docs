# pg_statsinfo

pg_statsinfo는 PostgreSQL의 통계 및 활동을 감시하는 유틸리티 입니다. pg_statsinfo는 쉽게 설치가 가능하며, 레파지토리 서버에 타겟 PostgreSQL 서버의 활동 정보와 시스템 리소스에 대한 내용을 정기적으로 스냅샷을 찍어 저장합니다. 이러한 스냅샷을 pg_stats_reporter라는 도구를 이용해 Web 기반으로 시각화하여 보여줍니다. 이 도구는 PostgreSQL 데이터베이스 관리자를 위해 상태 체크 또는 쉽게 서버의 활동 정보를 분석합니다.

## 동작 방식

#### 구성

pg_statsinfo는 3가지 주요 부분으로 구성됩니다.

* pg_statsinfod : 에이전트 프로그램으로 대상서버의 스냅샷 정보를 주기적으로 레파지토리 서버에 보냅니다.
* repository database : 스냅샷의 시간 순서에 맞게 수집정보를 저장하는 서버
* pg_stats_reporter : 사용자가 직관적인 방식으로 레파지토리 서버를 확인 할 수 있는 그래픽 뷰어

#### 아키텍쳐

![pg_statsinfo architecture](https://lh3.googleusercontent.com/Z7AeEBW3NWCBd4ZMvJoT6QMCkR3kxewy9xRlFeqgErA=w660-h495-no)

## 설치

[pg_statsinfo](http://sourceforge.net/projects/pgstatsinfo/files/pg_statsinfo/), [pg_stats_reporter](http://sourceforge.net/projects/pgstatsinfo/files/pg_stats_reporter/) 에서 최신 버전을 다운 받는다. rpm의 경우 OS 버전 및 데이터베이스 버전에 따라 해당하는 파일을 다운로드한다. 

#### pg_statsinfo 설치

	# PATH에 pg_config가 위치한 bin 디렉토리 지정
	vi ~/.bash_profile
	PATH=$PATH:$HOME/bin:/opt/PostgresPlus/9.4AS/bin

	# 압축해제 및 make
	tar xvf pg_statsinfo-3.1.0.tar.gz
	cd pg_statsinfo-3.1.0
	make && make install

	# $PGDATA/postgresql.conf 추가
	vi $PGDATA/postgresql.conf
	shared_preload_libraries = '$libdir/dbms_pipe,$libdir/edb_gen,pg_statsinfo,pg_stat_statements'
	## add config
	logging_collector = on
	log_autovacuum_min_duration = 0
	log_checkpoints = on
	log_destination = 'stderr, csvlog'
	log_line_prefix = '%t %u@%r/%d %p '
	log_lock_waits = on
	log_min_duration_statement = 3000
	log_temp_files = 0
	timed_statistics = on
	pg_stat_statements.max = 10000
	pg_stat_statements.track = all
	track_activity_query_size = 65536
	tcp_keepalives_idle = 300
	tcp_keepalives_interval = 15
	tcp_keepalives_count = 5
	track_functions = 'all'
	## pg_statsinfo config
	pg_statsinfo.sampling_interval = 5s
	pg_statsinfo.snapshot_interval = 1min
	pg_statsinfo.enable_maintenance = 'on'
	pg_statsinfo.maintenance_time = '00:02:00'
	pg_statsinfo.syslog_min_messages = 'disable'
	pg_statsinfo.textlog_min_messages = 'warning'
	pg_statsinfo.textlog_filename = 'pg_statsinfo.log'
	pg_statsinfo.textlog_line_prefix = '%t %u@%r/%d %p '
	pg_statsinfo.excluded_dbnames = 'template0, template1, edb, postgres'
	pg_statsinfo.excluded_schemas = 'pg_catalog, information_schema, sys, pg_toast'
	pg_statsinfo.repository_server = 'host=10.0.2.12 port=5432 dbname=repos user=repos'
	pg_statsinfo.repository_keepday = 7

	# 확장 모듈 설치
	psql dbname -c "CREATE EXTENSION pg_stat_statements"