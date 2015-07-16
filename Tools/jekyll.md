## ruby install

### 필요 파일

* Ruby 2.2.2 (x64) 인스톨러  
* DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe

### rubyinstaller-2.2.2-x64.exe 실행

인스톨 시 묻는 체크 사항 3가지 다 체크

### gem 재설치

[SSL 관련](https://gist.github.com/luislavena/f064211759ee0f806c88)

### DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe 실행

압축 해제 디렉토리로 이동 및 config 파일 생성

	cd C:\Users\Administrator\Downloads
	ruby dk.rb init

### config.yml 파일 수정

config.yml 파일의 마지막 라인에 루비 디렉토리를 명시해준다.

	- C:/Ruby22-x64


### 패키지 인스톨

	ruby dk.rb install


### 에러해결

> Couldn't reserve space for cygwin's heap (0x60E90000 <0x2590000>) in child, Win32 error 0  

[rebase](http://azza.tistory.com/152) 라는 프로그램으로 msys 수정

	rebase -b 0x30000000 msys-1.0.dll


## jekyll install

	gem install jekyll

## 시작하기

[설명 블로그](https://nolboo.github.io/blog/2013/10/15/free-blog-with-github-jekyll/)를 참조하여 설정한다.

### Github에 repository 생성

github 사이트에서 blog라는 repository를 생성한다.

### 사이트 생성

jekyll new 명령으로 사이트를 생성한다.

	jekyll new blog
	cd blog
	jekyll serve --watch

### 접속 확인

사이트를 생성했다면 localhost:4000로 접속이 가능하다.  
크롬을 열고 localhost:4000를 입력하여 접속한다.

#### github에 등록

다음의 명령으로 github에 사이트를 등록한다. (blog 디렉토리 안에서)

	git init
	git checkout --orphan gh-pages

## URL

#### ruby
[한국어 매뉴얼](http://ruby-korea.github.io/)  
[한국어 사이트](https://www.ruby-lang.org/ko/)  
[설치 파일](http://rubyinstaller.org/downloads/)  

#### jekyll
[jekyll](http://jekyllrb.com/)