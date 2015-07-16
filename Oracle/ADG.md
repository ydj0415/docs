# ORACLE ADG 설정

## On primary node01

### 아카이브 모드 및 force_logging 설정

	SHUTDOWN IMMEDIATE;
	STARTUP MOUNT;
	ALTER DATABASE ARCHIVELOG;
	ALTER SYSTEM SET DB_RECOVERY_FILE_DEST_SIZE=6G;
	ALTER DATABASE FORCE LOGGING;
	SELECT FORCE_LOGGING FROM V$DATABASE;
	ALTER DATABASE OPEN;

### 로그 그룹 확인

	SELECT GROUP#, BYTES/1024/1024 MB FROM V$LOG;
		GROUP#        MB
	---------- ----------
		 1        50
		 2        50
		 3        50

### Standby logfile 추가

	ALTER DATABASE ADD STANDBY LOGFILE GROUP 4 '/app/oracle/oradata/oraclem/sredo04.log' size 50m;
	ALTER DATABASE ADD STANDBY LOGFILE GROUP 5 '/app/oracle/oradata/oraclem/sredo05.log' size 50m;
	ALTER DATABASE ADD STANDBY LOGFILE GROUP 6 '/app/oracle/oradata/oraclem/sredo06.log' size 50m;
	ALTER DATABASE ADD STANDBY LOGFILE GROUP 7 '/app/oracle/oradata/oraclem/sredo07.log' size 50m;

### Standby logfile이 만들어졌는지 확인

	COL MEMBER FOR A50
	SELECT GROUP#, TYPE, MEMBER FROM V$LOGFILE WHERE TYPE = 'STANDBY';
	GROUP# TYPE    MEMBER
	———- ——- ——————————
	4 STANDBY /app/oracle/oradata/oraclem/sredo04.log
	5 STANDBY /app/oracle/oradata/oraclem/sredo05.log
	6 STANDBY /app/oracle/oradata/oraclem/sredo06.log
	7 STANDBY /app/oracle/oradata/oraclem/sredo07.log

	SELECT GROUP#, BYTES/1024/1024 MB FROM V$STANDBY_LOG;
	GROUP#         MB
	———- ———-
	4         50
	5         50
	6         50
	7         50

### 백업용 임시 디렉토리 생성

	oracle@vm01:~ (oraclem)$ mkdir /home/oracle/stage

### 파라미터 수정

	oracle@vm01:~ (oraclem)$ vi $ORACLE_HOME/dbs/initoraclem.ora
	#******************
	# Primary Role
	#******************
	*.compatible='11.2.0'
	*.db_create_file_dest = '/app/oracle/oradata'
	*.db_create_online_log_dest_1 = '/app/oracle/oradata'
	*.db_create_online_log_dest_2 = '/app/oracle/flash_recovery_area'
	*.db_name = oraclem
	*.db_unique_name = oraclem
	*.instance_name = oraclem
	*.log_archive_config = 'DG_CONFIG=(oraclem,oracles)'
	*.log_archive_dest_1 = 'location=/app/oracle/oradata/oraclem/archivelog valid_for=(all_logfiles, all_roles) db_unique_name=oraclem'
	*.log_archive_dest_2 = 'service=oracles async valid_for=(online_logfiles, primary_role) db_unique_name=oracles'
	*.log_archive_dest_state_1 = ENABLE
	*.log_archive_dest_state_2 = ENABLE
	*.remote_login_passwordfile = exclusive
	*.standby_file_management = auto
	*.dg_broker_start=TRUE
	 *.control_files='/app/oracle/oradata/oraclem/control01.ctl','/app/oracle/flash_recovery_area/oraclem/control02.ctl'
	*.db_block_size=8192
	*.db_domain=''
	*.diagnostic_dest='/app/oracle/product/11.2.0/db_1'
	*.nls_language='AMERICAN'
	*.open_cursors=300
	*.pga_aggregate_target=536870912
	*.processes=150
	*.sga_target=1610612736
	*.undo_tablespace='UNDOTBS1'
	*.open_links=100

### 재시작

	shutdown immediate;
	create spfile from pfile='/app/oracle/product/11.2.0/db_1/dbs/initoraclem.ora';
	startup;

### rman으로 데이터 백업

	oracle@vm01:~ (oraclem)$ rman target /
	CONFIGURE CONTROLFILE AUTOBACKUP ON;
	BACKUP FORMAT '/home/oracle/stage/%U' DATABASE PLUS ARCHIVELOG;
	BACKUP FORMAT '/home/oracle/stage/%U' CURRENT CONTROLFILE FOR STANDBY;
	sql "alter system archive log current";

### Standby 서버에 디렉토리 생성

	oracle@vm02:~ (oracles)$
	mkdir /home/oracle/stage
	mkdir -p /app/oracle/admin/oracles/adump
	mkdir -p /app/oracle/admin/oracles/dpdump
	mkdir -p /app/oracle/admin/oracles/pfile
	mkdir -p /app/oracle/cfgtoollogs/dbca/oracles
	mkdir -p /app/oracle/flash_recovery_area
	mkdir -p /app/oracle/flash_recovery_area/oracles
	mkdir -p /app/oracle/oradata/oracles
	mkdir -p /app/oracle/product/11.2.0/db_1/dbs

### 데이터베이스 삭제

	oracle@vm02:~ (oracles)$
	mkdir /home/oracle/stage
	rm -rf /app/oracle/admin/oracles/adump
	rm -rf /app/oracle/admin/oracles/dpdump
	rm -rf /app/oracle/admin/oracles/pfile
	rm -rf /app/oracle/cfgtoollogs/dbca/oracles
	rm -rf /app/oracle/flash_recovery_area
	rm -rf /app/oracle/flash_recovery_area/oracles
	rm -rf /app/oracle/oradata/oracles
	rm -rf /app/oracle/product/11.2.0/db_1/dbs

### 패스워드 파일 생성 및 Standby 서버에 전달

	oracle@vm01:~ (oraclem)$
	cd /app/oracle/product/11.2.0/db_1/dbs
	orapwd file=orapworaclem password=oracle ignorecase=y
	scp /app/oracle/product/11.2.0/db_1/dbs/orapworaclem vm02:/app/oracle/product/11.2.0/db_1/dbs/orapworacles


### 리스너 설정

	oracle@vm01:~ (oraclem)$
	vi /app/oracle/product/11.2.0/db_1/network/admin/listener.ora
	LISTENER =
	  (DESCRIPTION_LIST =
		(DESCRIPTION =
		  (ADDRESS_LIST =
			(ADDRESS = (PROTOCOL = TCP)(HOST = vm01)(PORT = 1521))
		  )
		  (ADDRESS_LIST =
			(ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC))
		  )
		)
	  )
	SID_LIST_LISTENER =
	  (SID_LIST =
		(SID_DESC =
		  (GLOBAL_DBNAME = oraclem)
		  (ORACLE_HOME = /app/oracle/product/11.2.0/db_1)
		  (SID_NAME = oraclem)
		)
	  )
	ADR_BASE_LISTENER = /app/oracle

### tnsname 설정

	oracle@vm01:~ (oraclem)$ vi /app/oracle/product/11.2.0/db_1/network/admin/tnsnames.ora
	ORACLEM =
	  (DESCRIPTION =
	   (ADDRESS_LIST =
		(ADDRESS=(PROTOCOL=TCP)(HOST=vm01)(PORT=1521))
		)
		(CONNECT_DATA =
		(SERVER = DEDICATED)
		(SID = ORACLEM)
	   )
	  )

	ORACLES =
	  (DESCRIPTION =
	   (ADDRESS_LIST =
		(ADDRESS=(PROTOCOL=TCP)(HOST=vm02)(PORT= 1521))
		)
		(CONNECT_DATA =
		(SERVER = DEDICATED)
		(SID = ORACLES)
	   )
	  )

## On standby node02

### 파라미터 수정 

	oracle@vm02:~ (oracles)$
	vi $ORACLE_HOME/dbs/initoracles.ora
	#******************
	# Standby Role
	#******************
	*.compatible='11.2.0'
	*.db_create_file_dest = '/app/oracle/oradata'
	*.db_create_online_log_dest_1 = '/app/oracle/oradata'
	*.db_create_online_log_dest_2 = '/app/oracle/flash_recovery_area'
	*.db_file_name_convert = 'oraclem','oracles'
	*.db_name = oraclem
	*.db_unique_name = oracles
	*.fal_client = oracles
	*.fal_server = oraclem
	*.instance_name = oracles
	*.log_archive_config = 'DG_CONFIG=(oraclem,oracles)'
	*.log_archive_dest_1 = 'location=/app/oracle/oradata/oracles/archivelog valid_for=(all_logfiles, all_roles) db_unique_name=oracles'
	*.log_archive_dest_2 ='service=oraclem async valid_for=(online_logfiles, primary_role) db_unique_name=oraclem'
	*.log_archive_dest_state_1 = ENABLE
	*.log_archive_dest_state_2 = ENABLE
	*.log_archive_max_processes = 5
	*.log_file_name_convert = 'oraclem','oracles'
	*.remote_login_passwordfile = exclusive
	*.standby_file_management = auto
	*.dg_broker_start=TRUE
	*.control_files='/app/oracle/oradata/oracles/control01.ctl','/app/oracle/flash_recovery_area/oracles/control02.ctl'
	*.db_block_size=8192
	*.db_domain=''
	*.diagnostic_dest='/app/oracle/product/11.2.0/db_1'
	*.nls_language='AMERICAN'
	*.open_cursors=300
	*.pga_aggregate_target=536870912
	*.processes=150
	*.sga_target=1610612736
	*.undo_tablespace='UNDOTBS1'
	*.open_links=100

### 리스너 설정

	oracle@vm02:~ (oracles)$
	vi /app/oracle/product/11.2.0/db_1/network/admin/listener.ora
	LISTENER =
	  (DESCRIPTION_LIST =
		(DESCRIPTION =
		  (ADDRESS_LIST =
			(ADDRESS = (PROTOCOL = TCP)(HOST = vm02)(PORT = 1521))
		  )
		  (ADDRESS_LIST =
			(ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC))
		  )
		)
	  )
	SID_LIST_LISTENER =
	  (SID_LIST =
		(SID_DESC =
		  (GLOBAL_DBNAME = oracles)
		  (ORACLE_HOME = /app/oracle/product/11.2.0/db_1)
		  (SID_NAME = oracles)
		)
	  )
	ADR_BASE_LISTENER = /app/oracle

### tnsname 설정

	oracle@vm02:~ (oracles)$
	vi /app/oracle/product/11.2.0/db_1/network/admin/tnsnames.ora
	ORACLES =
	  (DESCRIPTION =
	   (ADDRESS_LIST =
		(ADDRESS=(PROTOCOL=TCP)(HOST=vm02)(PORT= 1521))
		)
		(CONNECT_DATA =
		(SERVER = DEDICATED)
		(SID = ORACLES)
	   )
	  )

	ORACLEM =
	  (DESCRIPTION =
	   (ADDRESS_LIST =
		(ADDRESS=(PROTOCOL=TCP)(HOST=vm01)(PORT=1521))
		)
		(CONNECT_DATA =
		(SERVER = DEDICATED)
		(SID = ORACLEM)
	   )
	  )


### Standby DB pfile 생성

	oracle@vm02:~ (oracles)$
	export ORACLE_SID=oracles
	CREATE SPFILE FROM PFILE='/app/oracle/product/11.2.0/db_1/dbs/initoracles.ora';
	STARTUP NOMOUNT;
	EXIT

### rman으로 duplicate 수행 oraclem -> oracles

	oracle@vm02:~ (oracles)$
	export ORACLE_SID=oracles
	rman TARGET SYS/oracle@oraclem  AUXILIARY SYS/oracle@oracles 
	DUPLICATE TARGET DATABASE FOR STANDBY FROM ACTIVE DATABASE;

### Primary DB의 현재 로그를 스위칭

	ALTER SYSTEM SWITCH LOGFILE;

### Standby DB의 redo apply 수행

	ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT;

### Standby DB가 정상적으로 복제 중인지 확인 

	SELECT SEQUENCE#, FIRST_TIME, NEXT_TIME,APPLIED FROM V$ARCHIVED_LOG ORDER BY SEQUENCE#;

### Primary DB에서 로그 스위칭을 하여 반복 테스트

	ALTER SYSTEM SWITCH LOGFILE;
	ALTER SYSTEM SWITCH LOGFILE;
	ALTER SYSTEM SWITCH LOGFILE;
	ALTER SYSTEM SWITCH LOGFILE;

### Standby DB가 정상적으로 복제 중인지 확인 및 프로세스 확인

	SELECT SEQUENCE#, FIRST_TIME, NEXT_TIME,APPLIED FROM V$ARCHIVED_LOG ORDER BY SEQUENCE#;
	SELECT PROCESS, STATUS FROM V$MANAGED_STANDBY;

### 아래 프로세스가 떠있으면 정상

	MRP0       APPLYING_LOG

## URL

http://oracleinaction.com/dg-setup-active-dataguard/