# FreeXN

FreeNX 는 SSH Port 를 이용하여 X-Window 에 접속할 수 있게하여 줍니다.

## 설치

#### yum을 이용하여 FreeNX Server 설치

	yum install nx freenx

#### 설치 확인

	nxserver --list

	NX> 100 NXSERVER - Version 3.2.0-74-SVN OS (GPL, using backend: not detected)
	NX> 127 Sessions list:

	Server     Display Username        Remote IP       Session ID
	------ ------- --------------- --------------- --------------------------------
	NX> 999 Bye

## 설정

#### FreeNX Server 파라미터 파일 수정

	vi /etc/nxserver/node.conf
	
	# This adds the passdb to the possible authentication methods
	ENABLE_PASSDB_AUTHENTICATION="1"
	* 인증 설정 - 주석 제거 후 1로 변경

	# The port number where local 'sshd' is listening.
	SSHD_PORT=22
	* SSH Port 설정

#### 사용자 생성 및 패스워드 설정

/etc/passwd에 동록되어 있어야 한다.

	* 사용자 등록 root 계정
	nxserver --adduser root

	NX> 100 NXSERVER - Version 3.2.0-74-SVN OS (GPL, using backend: not detected)
	NX> 1000 NXNODE - Version 3.2.0-74-SVN OS (GPL, using backend: not detected)
	NX> 716 Public key added to: /root/.ssh/authorized_keys2
	NX> 1001 Bye.
	NX> 999 Bye

	* root 계정의 패스워드 설정(freenx 전용)
	nxserver --passwd root

	NX> 100 NXSERVER - Version 3.2.0-74-SVN OS (GPL, using backend: not detected)
	New password: 
	Password changed.
	NX> 999 Bye

#### 인증키 확인

인증키를 복사하여 추후 NoMachine 클라이언트에 붙여 넣는다.

	cat /var/lib/nxserver/home/.ssh/client.id_dsa.key

	-----BEGIN DSA PRIVATE KEY-----
	MIIBuwIBAAKBgQC7x8GrqDJrQOPlmzRq+YF9YBV93KF9fiXMDie5vnUVbVBIYq0g
	6Dl/v2VctQVB91IiQwdrDB8r2lNFNkFjXQn6X845A//Uy/A7J7IsnFZeHNhRkZZ3
	FhuhTSERDt+LDa14lTXZH7oKfd5XTLRRVj9K4T0JYz2tAy8hlcl2kkx1XQIVAKYm
	euWOG3Ik518j34iTRC/hUpbdAoGAHGgmsfrKyibb72IcTjztr/Tml3Dw5S4awGo4
	yOJfclxCXj5MzP6U21WsULnWJCevue3jtBNdbBx3r5zR8Ox03SSus7Ca9T+6r5y8
	kd8pZAJz/IRv1oL9bSAMXbBEFqCdXD6GxKHgqzGlYugKmGsPBSjRmHrpD2dtr6vY
	qUGfj6sCgYBSouvVJPcSDuS7rxkiYilfR4Vw18ICGbYasKeXo0Wekkm59TflL0Pb
	EioEhzkv7nxJ9OxG3nroLLUHHZNU5YapFJ9ctupDeXGdQcK2r4RWT+waylp+4V98
	5SnFtz4uch2XypPJ7Rm15MPLfDDAjJ1ZnDuc+3pf/8oKxbN2HxnfSAIVAIWF2B1p
	8ZJZDgBwsCslUYHsm5iv
	-----END DSA PRIVATE KEY-----

#### 서버 시작

FreeNX 서버를 재시작 한다.

	nxserver --restart

	NX> 100 NXSERVER - Version 3.2.0-74-SVN OS (GPL, using backend: not detected)
	NX> 123 Service stopped
	NX> 122 Service started
	NX> 999 Bye

## NoMachine 클라이언트 시작

[NX Client 3.5버전](http://nx-client-for-windows.software.informer.com/3.5/)을 다운로드 한다.

#### configure

* Host : FreeNX 서버 아이피 정보
* Port : FreeNX 서버 포트 정보
* Key : FreeNX 서버에서 client.id_dsa.key 파일의 키를 복사하여 붙여 넣는다
* Unix GNOME 선택
* Display : 1024x768 선택

#### 접속

* Login : FreeNX 서버에 등록된 유저
* Password : FreeNX 서버에 등록된 유저 패스워드
* Session : 접속 세션 별칭
