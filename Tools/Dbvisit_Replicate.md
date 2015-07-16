## Dbvisit Replicate

### 스크립트 파일

설치 시 디렉토리로 이동하여 파일을 확인한다.

	root@ip-10-0-1-114:/app/oracle/dbvisit (oracles)$

	ls -ltrh

	-rwxr----- 1 oracle oinstall  233 Jul  8 08:02 APPLY.sh
	-rw-r----- 1 oracle oinstall 1.3K Jul  8 08:02 Nextsteps.txt
	-rw-r--r-- 1 oracle oinstall 5.1K Jul  8 08:02 oraclem-all.log
	-rw-r----- 1 oracle oinstall  523 Jul  8 08:02 oraclem-APPLY.ddc
	-rwxr----- 1 oracle oinstall 1.4K Jul  8 08:02 oraclem-all.sh
	-rw-r----- 1 oracle oinstall  523 Jul  8 08:02 oraclem-MINE.ddc
	-rwxr----- 1 oracle oinstall  100 Jul  8 08:02 oraclem-run-ip-10-0-1-114.sh
	-rwxr----- 1 oracle oinstall   98 Jul  8 08:02 oraclem-run-ip-10-0-1-207.sh
	-rwxr-x--- 1 oracle oinstall   68 Jul  8 08:02 start-console.sh
	drwxrwxrwx 3 root   root     4.0K Jul 16 02:05 log
	drwxrwxrwx 2 root   root     4.0K Jul 16 02:05 ddc_backup
	drwxrwxrwx 2 root   root     4.0K Jul 16 02:06 apply

* oraclem-all.sh 초기 설정을 하는 스크립트 파일(최초 1회만 수행)
* sid-run-hostname.sh 형태의 파일이 생성되는데 각각의 호스트에서 mine -> apply 순으로 수행해 주면 된다.
* start-console.sh Dbvisit Replicate를 관리해 주는 스크립트 파일

### 프로그램 시작

각각의 호스트에서 스트립트 파일을 수행한다.

> MINE 프로세스 수행

	root@ip-10-0-1-207:/app/oracle/dbvisit (oraclem)$

	./oraclem-run-ip-10-0-1-207.sh

	Initializing......done
	DDC loaded from database (363 variables).
	Dbvisit Replicate version 2.7.12.4968
	Copyright (C) Dbvisit Software Limited.  All rights reserved.
	DDC file /app/oracle/dbvisit/oraclem-MINE.ddc loaded.
	Starting process MINE...started

> APPLY 프로세스 수행

	root@ip-10-0-1-114:/app/oracle/dbvisit (oracles)$

	./oraclem-run-ip-10-0-1-114.sh

	Initializing......done
	DDC loaded from database (363 variables).
	Dbvisit Replicate version 2.7.12.4968
	Copyright (C) Dbvisit Software Limited.  All rights reserved.
	DDC file /app/oracle/dbvisit/oraclem-APPLY.ddc loaded.
	Starting process APPLY...started

### 콘솔 수행

콘솔은 다음의 스크립트로 수행할 수 있다.

	root@ip-10-0-1-207:/app/oracle/dbvisit (oraclem)$

	./start-console.sh

	Initializing......done
	DDC loaded from database (363 variables).
	Dbvisit Replicate version 2.7.12.4968
	Copyright (C) Dbvisit Software Limited.  All rights reserved.
	\ Dbvisit Replicate 2.7.12.4968(MAX edition) - Evaluation License expires in 22 days
	MINE IS running. Currently at plog 24 and SCN 1109984 (07/16/2015 02:26:27).
	APPLY IS running. Currently at plog 24 and SCN 1109976 (07/16/2015 02:26:27).
	Progress of replication oraclem:MINE->APPLY: total/this execution (stale)
	-------------------------------------------------------------------------------------
	0 tables listed.

	dbvrep> 

### 주요 명령어

* 전체 프로세스 종료  
`shutdown all`

* 일시정지  
`pause apply`

* 재시작  
`resume apply`

* 현재 충돌 리스트 확인  
`list conflict current`

* 해결된 충돌 리스트 확인  
`list conflict last`

* 화면 지우기  
`cls`

* 양방향 또는 여러 Replication 구성 시 Replication 선택  
`choose replication mine1`  
`choose replication apply1`

* 파라미터 파일 위치 확인  
`show setup_script_path`

* 도움말 확인  
`help list`


### Replication schema or table 등록

스키마 등록 시, 양쪽 노드 모두 다음과 같이 스키마를 생성하고 권한을 부여한다.

	conn sys/oracle@oraclem as sysdba
	Connected.
	create user new_schema identified by new_schema;
	User created.
	grant connect, resource to new_schema;
	Grant succeeded.

	conn sys/oracle@oracles as sysdba
	Connected.
	create user new_schema identified by new_schema;
	User created.
	grant connect, resource to new_schema;
	Grant succeeded.

Dbvisit 관리 콘솔에 접속 후 스키마 및 테이블을 등록 또는 해제 해 준다.

	prepare schema new_schema
	unprepare schema old_schema

다시 Database에 접속하여 테이블 생성 및 row를 추가한다.

	conn new_schema/new_schema@oraclem
	Connected.
	create table tab1 (i number primary key, a varchar2(5));
	Table created.
	insert into tab1 values (1, 'a');
	1 row created.
	commit;
	Commit complete.

동기화가 정상적으로 수행되고 있는지 확인 한다.  

> 콘솔

	\ Dbvisit Replicate 2.7.12.4968(MAX edition) - Evaluation License expires in 22 days
	MINE IS running. Currently at plog 25 and SCN 1155234 (07/16/2015 04:44:03).
	APPLY IS running. Currently at plog 25 and SCN 1155214 (07/16/2015 04:43:59).
	Progress of replication oraclem:MINE->APPLY: total/this execution
	-----------------------------------------------------------------------------------------------------------------------------
	SCOTT.TAB1:                   100%  Mine:1/1             Unrecov:0/0         Applied:1/1         Conflicts:0/0       Last:16/07/2015 04:08:50/OK
	-----------------------------------------------------------------------------------------------------------------------------
	1 tables listed.

> SQL

	conn scott/oracle;
	Connected.
	select * from tab1;

		 A B
	---------- -----
		 1 a