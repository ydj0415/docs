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

### 명령어 모음

|명령어|내용|
|-|-|
|shutdown all|전체 프로세스 종료|
|pause apply|프로세스 일시 정지|
