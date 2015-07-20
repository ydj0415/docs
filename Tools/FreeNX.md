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

vi /etc/nxserver/node.conf