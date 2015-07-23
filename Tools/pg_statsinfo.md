# pg_statsinfo
변경
pg_statsinfo는 PostgreSQL의 통계 및 활동을 감시하는 유틸리티 입니다. pg_statsinfo는 쉽게 설치가 가능하며, 레파지토리 서버에 타겟 PostgreSQL 서버의 활동 정보와 시스템 리소스에 대한 내용을 정기적으로 스냅샷을 찍어 저장합니다. 이러한 스냅샷을 pg_stats_reporter라는 도구를 이용해 Web 기반으로 시각화하여 보여줍니다. 이 도구는 PostgreSQL 데이터베이스 관리자를 위해 상태 체크 또는 쉽게 서버의 활동 정보를 분석합니다.

## 동작 방식

#### 구성

pg_statsinfo는 3가지 주요 부분으로 구성됩니다.

* pg_statsinfod : 에이전트 프로그램으로 대상서버의 스냅샷 정보를 주기적으로 레파지토리 서버에 보냅니다.
* repository database : 스냅샷의 시간 순서에 맞게 수집정보를 저장하는 서버
* pg_stats_reporter : 사용자가 직관적인 방식으로 레파지토리 서버를 확인 할 수 있는 그래픽 뷰어

#### 아키텍쳐

![pg_statsinfo architecture](https://lh3.googleusercontent.com/Z7AeEBW3NWCBd4ZMvJoT6QMCkR3kxewy9xRlFeqgErA=w660-h495-no)

## pg_statsinfo 설치

[pg_statsinfo](http://sourceforge.net/projects/pgstatsinfo/files/pg_statsinfo/) 에서 최신 버전을 다운 받는다.

#### PATH에 pg_config가 위치한 bin 디렉토리 지정

	vi ~/.bash_profile
	PATH=$PATH:$HOME/bin:/opt/PostgresPlus/9.4AS/bin

#### 압축해제 및 make

	tar xvf pg_statsinfo-3.1.0.tar.gz
	cd pg_statsinfo-3.1.0
	make && make install

#### 운영서버 postgresql.conf 수정

	vi $PGDATA/postgresql.conf
	shared_preload_libraries = '$libdir/dbms_pipe,$libdir/edb_gen,pg_statsinfo,pg_stat_statements'
	logging_collector = on
	log_autovacuum_min_duration = 0
	log_checkpoints = on
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


#### 확장 모듈 설치

	psql dbname -c "CREATE EXTENSION pg_stat_statements"

## pg_statsinfo 명령어

####타겟 서버

	pg_statsinfo -d postgres --start
	pg_statsinfo -d postgres --stop

#### repo 서버

	# 스냅샷 리스트
	pg_statsinfo -l

	# 리포트 작성
	pg_statsinfo -r All -i 1 -o snap_20150722.txt

	# 스냅샷 생성
	pg_statsinfo -S COMMENT

	# 스냅샷 삭제
	pg_statsinfo -D SNAPID

## pg_stats_repoter 설치

[pg_stats_reporter](http://sourceforge.net/projects/pgstatsinfo/files/pg_stats_reporter/) 에서 최신 버전을 다운 받는다.

#### 필요 패키지 설치

	yum install httpd php php-pgsql php-intl php-cli

#### 소스 설치

	tar xvfz pg_stats_reporter-3.1.0.tar.gz
	cp -R pg_stats_reporter-3.1.0/html/pg_stats_reporter /var/www/html
	cp -R pg_stats_reporter-3.1.0/pg_stats_reporter_lib /var/www
	cp pg_stats_reporter-3.1.0/bin/pg_stats_reporter /usr/local/bin
	cd /var/www/pg_stats_reporter_lib
	chown apache.apache cache compiled

#### 변수 파일 설정

	cd /var/www/pg_stats_reporter_lib
	cp pg_stats_reporter.ini.sample /etc/pg_stats_reporter.ini
	cd /etc

	# ini 파일 수정 (레파지토리 서버)
	vi pg_stats_reporter.ini
	----- Set your repository database -----
	host = localhost
	port = 5444
	dbname = postgres
	username = enterprisedb
	password = edb

#### httpd 시작 및 확인

	service httpd start

	http://192.168.56.102/pg_stats_reporter/pg_stats_reporter.php