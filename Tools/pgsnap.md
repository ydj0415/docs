# pgsnap

php를 이용하여 데이터베이스 상태 정보를 스냅샷 찍는 프로그램

## 설치

#### 1. php pgsql 모듈 설치

	yum -y install php-pgsql php-cli

#### 2. pgsnap 다운로드

	curl -O -L https://github.com/downloads/dalibo/pgsnap/pgsnap-0.8.0.tar.gz

#### 3. 압축 해제

	tar xvf pgsnap-0.8.0.tar.gz
	cd pgsnap-0.8.0

#### 4. 9.4 사용 시 reltoastidxid 이 있는 라인을 제거해준다.

	vi lib/tables.php
	vi lib/views.php
	vi lib/sequences.php
	vi lib/indexes.php 

## 스냅샷

다음의 명령어로 데이터베이스 상태를 저장한다. edb_snap_yyyymmdd 형태의 디렉토리에 저장된다.

	./pgsnap.php --without-sysobjects --with-old-libpq
	
	# 압축
	tar cvf edb_snap_20141224.tar edb_snap_20141224/