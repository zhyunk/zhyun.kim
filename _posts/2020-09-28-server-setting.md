---
title: ipTIME NAS2dual로 서버구축하기 - 01. APM 설정
date: 2020-09-28 23:36:20 
categories: [blog, develop]
tags: [blog]     # TAG names should always be lowercase
---

# 개발 환경

> server
> - iptime nas2dual - BusyBox v1.00-rc3 Built-in shell (ash)
> - apache2
>
> php 7.3
> - framework: codeigniter v3.1.11
>
> database
> - mysql 5.6.17  
  
----  

# Apache 설정  

 iptime nas에서 사용하는 경우, <u>기본 서비스</u>로 제공하기 때문에 따로 설치를 해줄 필요가 없다.  
Apache 서비스를 실행하도록 설정만 해주면 된다.  
설정하는 방법은 다음과 같다.  

## ※ 설치 (with PHP) 

1. 관리툴에 접속  
![1번](/assets/img/blog/nas1.png)  

2. `서비스 관리` - `Apache 서버` 선택  
> 이 때, 원하는 버전의 PHP를 체크해주면 해당 버전의 PHP와 함께 설치된다.
> ![2번](/assets/img/blog/nas2.png)  

3. 서비스 `실행` 선택 ,  
포트번호는 `기본포트사용` 혹은 원하는 포트 번호를 입력한다.  
![3-1번](/assets/img/blog/nas3-1.png)  

4. 아파치 서버 설정파일 등 주요 관련 파일들이 저장될 Server Root를 지정한다.  
![3-2번](/assets/img/blog/nas3-2.png)  
![3-3번](/assets/img/blog/nas3-3.png)  

5. 작성한 웹문서가 실행될 폴더인 Document Root를 지정한다.
![4-1번](/assets/img/blog/nas4-1.png)  
![4-2번](/assets/img/blog/nas4-2.png)  

## ※ 설정  

1. 탐색기모드에 진입힌다.  
![5번](/assets/img/blog/nas5.png)   

2. 관리툴에서 설정한 Server Root 폴더로 찾아간다.  

3. `httpd.conf` 파일에서 Apache 관련 설정을 업데이트 하면 된다.  
> 이 파일을 수정할 경우,  
> 관리툴로 돌아가서 Apache 서버 - `재시작하기` 버튼을 꼭 눌러주어야 적용되어 실행된다.  
> ![6번](/assets/img/blog/nas6.png)   
  
----  

# PHP  

## ※ 설정  

아파치 설정 폴더(Server Root 위치)에 진입하여 `php.ini` 파일을 수정하면 된다.

## ※ PHP Framework 설치  

1. http://codeigniter.com/download 접속 후 *~~최신버전~~* v3.1.11 다운로드
  - 최신버전이 2020.09.27 기준 v4.0.4인데,   
  내가 사용할 iptime nas에 telnet이 지원되서   
  telnet으로 접속해보니  
  BusyBox v1.00-rc3 Built-in shell (ash) 어쩌구저쩌구 ㅠㅠㅠㅠㅠ  
  쉘에서 help를 입력해보면 다음과 같이 나온다. 아마 해당 명령어만 사용하단 거겠지?!   
    ```
    Built-in commands:
    -------------------
        . : alias bg break cd chdir continue eval exec exit export false
        fg hash help jobs kill let local pwd read readonly return set
        shift times trap true type ulimit umask unalias unset wait
    ```  
    구글링 해보니 busybox는  
    하나의 실행파일 안에 스트립다운된 일부 유닉스 도구들을 제공하는 소프트웨어로,  
    자원이 매우 적은 임베디드 운영체제를 위해 작성된거라고 한다.   
    우분투, centOs, redhat 등등 리눅스 OS가 설치된 환경이라면  
    OS에 이것저것 설치해보며 개발을 할수 있지만  
    이건 도저히 내가 건들수가 없다 ㅠㅠ   
    iptime nas 말고 시놀로지 nas는 소프트웨어가 정말 잘 되어있다고 하던데,  
    *~~시놀로지 넘 비싸..~~*  
    나중에 형편이 좋아지면 시놀로지로 갈아타야지 ㅠㅠ  
    그래서 코드이그나이터 최신버전말구 버전 3.* 로 설치하게 되었다.  
    최신버전이 4.0.4인데 그건 지금 환경에선 다루지 못하기때문..ㅠㅠ  
    

2. 다운로드한 파일의 압축을 해제하여 웹 루트폴더로 이동시킨다.

3. 웹 주소로 접속하여 welcome 화면이 나타나는지 확인한다.

4. 주소에 index.php 제거하기  
- 아파치 설정파일에서  
서버 디렉터리의 `AllowOverride None` 옵션을 `AllowOverride All`로 수정  
- 코드이그나이터 설정파일인  
application/config/config.php 파일에서  
`$config['index_page'] = 'index.php'`값을  
`$config['index_page'] = ''`으로 수정한다.  
- 서버 디렉터리의 루트 위치에 다음의 내용을 입력한 .htaccess 파일을 생성한다.  
  
  ```
  <IfModule mod_rewrite.c>
      RewriteEngine On
      RewriteCond $1 !^(index\.php|images|captcha|data|include|uploads|robots\.txt)
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteCond %{REQUEST_FILENAME} !-d
      RewriteRule ^(.*)$ /index.php/$1 [L]
      # 만약 서브 디렉터리에서 테스트용 코드이그나이터 프레임워크를 따로 실행할 경우, 
      # RewriteRule 부분에 "/서브_디렉터리"를 추가하여 작성한다.
      # e.g: RewriteRule ^(.*)$ /서브_디렉터리/index.php/$1 [L]
  </IfModule>
  ```  
- .htaccess 파일의 권한을 755로 수정한다.  
  .htaccess 파일 작성/수정 후 아파치를 재실행할 필요는 없다.  
  
----  

# MySql 설치  

 iptime nas에서 사용하는 경우, <u>기본 서비스</u>로 제공하기 때문에 따로 설치를 해줄 필요가 없다.  
MySql 서비스를 실행하도록 설정만 해주면 된다.  
설정하는 방법은 다음과 같다.  

## ※ 설치  

1. iptime 관리툴에 접속  

2. `서비스 관리` - `MYSQL 서버` 선택  

3. 서비스 `실행`을 선택하고,  
포트번호, DB 저장 위치, 서버문자셋 등등의 설정을 한뒤 저장하기를 누르면 기본 설치가 완료되었다.  
![2번](/assets/img/blog/db1.png)  

4. MYSQL 설치 후, 접속 로그인 계정 및 데이터베이스를 설정하기 위해 플러그인을 추가로 설치한다.  
`iptime 관리툴` - `Plug-in APP 관리` - `Plug-in APP 검색/설치` - `phpMyAdmin` 설치  
![phpMyAdmin](/assets/img/blog/phpMyAdmin.png)  

5. 설치 완료 후 `Plug-in APP 설정`으로 이동하여 `phpMyAdmin - 설정`을 누른다.
> root 계정으로 접속하여 (초기 비밀번호 없음) root계정의 비밀번호를 설정해준다.

6. APM 초기 세팅이 끝났으니,  
마지막으로 개인적으로 자주 사용하는 DataGrip이라는 프로그램에 접속 정보를 설정해준다.  
> `ServerTimeZone`이 빈 값으로 조회되어 오류가 뜨는 경우 `Asia/Seoul` 로 설정해주면 된다.
