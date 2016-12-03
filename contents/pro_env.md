# 운영서버 환경구축

대개의 경우 리눅스 계열의 운영체제를 이용하여 서버를 구축한다.

서버에 설치해야 할 기본 프로그램은 아래와 같다.

1. Git 설치
2. NodeJS 설치
3. ImageMagick 설치
4. Nginx(또는 Apache) 웹서버 설치
5. MySQL(또는 PostgreSQL) DB서버 설치

이외에도 웹서버에 설치할 웹어플리케이션의 기능을 지원하기 위한 다양한 툴을 설치할 수 있다.

서버를 구축한 후 웹프로젝트를 서버로 배포하기 위해서는 [프로젝트 배포하기](../deployment/first.html)를 참고하면 된다.

---

# CentOS 7.2 서버 설치

아래의 링크에서 원하는 ISO 파일을 다운로드 받아 서버에 설치한다.

- [CentOS 다운로드](https://www.centos.org/download/)
  - [Minimal ISO](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1511.iso) 
  - [Everything ISO](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Everything-1511.iso)
  - [DVD ISO](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1511.iso)

여기서는 _**Minimal ISO 버전을 설치**_하여 작업할 것이다. 

---

###시스템 버전 

```
# cat /etc/centos-release
CentOS Linux release 7.2.1511 (Core)
```

###시스템 업데이트

: `Development Tools`를 설치할 때 `git`도 함께 설치된다. (2014년 3월 7일  현재 - 1.7.1 버전, 그러나 최신버전은 1.9.0)

`root` 계정으로 로그인한 후 터미널에서 아래의 작업을 진행한다.

```
# yum -y update
```

업데이트 진행 중에 다음과 같은 에러가 발생하면

```
Error: Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
```
아래와 같은 조치를 하면 해결할 수 있다. 
(https://www.conory.com/note_linux/44007)

아래 명령어를 실행하여 해당 파일이 속한 패키지이름을 알려 준다.

```
# yum provides '*/applydeltarpm'
```

그 결과 deltarpm 패키지를 설치해주면 된다.

```
# yum install deltarpm
```

###개발툴 설치

```
# yum groupinstall -y "Development Tools"
```

###Deployer 유저 등록

: 서버 배포를 위한 유저를 등록한다.

* 통상 `deployer`라는 유저명을 사용한다.

  ```
  # useradd deployer
  # passwd deployer
  ```


* `wheel` 그룹에 `deployer`를 추가한다.

  ```
  # usermod -aG wheel deployer
  ```


* `/etc/sudoers` 파일을 열어 `wheel` 그룹에 대해서 아래와 같이 코멘트 표시(#)를 삭제한다(108번째 코드라인).

  ```
  # vi /etc/sudoers

  ## Allows people in group wheel to run all commands
  %wheel  ALL=(ALL)       ALL
  ## Same thing without a password
  %wheel  ALL=(ALL)       NOPASSWD: ALL
  ```


* sshd를 재시작한다.

  ```
  # service sshd restart
  ```


###Firewall 설정

* 참고 : 
  - [CentOS 7 에서 iptables 방화벽 데몬 사용하기](http://luckyyowu.tistory.com/286)
  - [RHEL/CentOS 7 에서 방화벽(firewalld) 설정하기](https://www.lesstif.com/pages/viewpage.action?pageId=22053128)


* 방화벽의 설치 

  ```
  # yum install firewalld
  # systemctl stop firewalld
  # systemctl mask firewalld
  ```

* iptables 서비스 설치

  ```
  # yum install iptables-services
  # systemctl [stop | start | restart ] iptables
  # service iptables save
  ```

* 방화벽에서 80포트를 열어둔다.(9번째 코드라인 아래에 아래의 내용을 추가한다.)

  ```
  # vi /etc/sysconfig/iptables

  -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
  ```


* iptables를 재시작한다.

  ```
  # service iptables restart
  ```

###Nginx 서버


- 참고 : http://wiki.nginx.org/Install

- `nginx`의 yum 저장소를 추가하기 위해서, `/etc/yum.repos.d/nginx.repo` 파일을 생성하고 아래의 옵션을 붙여 넣는다.

  ```
  [nginx]
  name=nginx repo
  baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
  gpgcheck=0
  enabled=1
  ```

  > **_주의:_** CentOS, RHEL, Scientific Linux가 릴리스 버전($releasever) 변수에 따라 원하는 값으로 대체한다. OS 버전에 따라 6.x는 "6", 7.x 는 "7"을 사용하면 된다. $basesearch 에는 "x86_64" 값을 지정한다.

  ```
  # yum install -y nginx
  # chkconfig --levels 235 nginx on
  # mkdir -p /etc/nginx/sites-enabled
  # vi /etc/nginx/nginx.conf
  ```


* 설치하고 `/etc/nginx` 디렉토리에 `sites-enabled` 폴더를 생성해 준다. 그리고 `/etc/nginx/nginx.conf` 파일 32번 줄에 `include /etc/nginx/sites-enabled/*.conf;` 추가한다.

* `include /etc/nginx/sites-enabled/*.conf;` 을 추가하고, user를 `deployer`로 변경한다.


* nginx 를 시작한다.

  ```
  # systemctl start nginx
  ```

###MySQL (5.6.34) 서버

- 참고 : 
  - [How to Install MySQL on CentOS 7](https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7)
  - [Centos 7 - 데이타베이스(MySql) 설치](https://opentutorials.org/module/1701/10229)


- 설치하기 

 ```bash
 # yum -y install http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
 # yum -y install mysql-community-server
 # systemctl start mysqld
 # systemctl enable mysqld.service
 # mysql_secure_installation
 ```

* /etc/my.cnf 파일에 아래 줄을 추가해 준다.

  ```
  [mysqld]
  character-set-server = utf8
  ```

* 권한을 승인한다.

  ```bash
  # mysql -u root -p

  mysql> grant usage on *.* to deployer@localhost identified by 'password';
  ```

###PostgreSQL (9.2.15) 설치

- 참고:
  - [How To Install and Use PostgreSQL on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-centos-7)


- 설치
  ```
  # yum install postgresql-server postgresql-contrib
  ```

- 데이터베이스 초기화
  ```
  # postgresql-setup initdb
  ```
- HBA에서 password-based authentication 을 가능하도록 해준다.
  ``` 
  # vi /var/lib/pgsql/data/pg_hba.conf
  ```

  82, 84번째 `ident` 를 `md5`로 변경함. 

  ```
  ...
  82 | host    all      all      127.0.0.1/32      md5
  ...
  84 | host    all      all      ::1/128           md5
  ...
  ```
- PostgreSQL 서버를 시작하여 사용가능하도록 해준다.

  ```
  # systemctl start postgresql
  # systemctl enable postgresql
  ```

- `postgres` 계정(PostgreSQL 관리자 계정)으로 변경한 후,

  ```
  # sudo -i -u postgres
  ```
- `deployer` 계정(PostgreSQL용)을 생성한 후 데이터베이스까지 생성한다. (이 때 `deployer` 계정에는 데이터베이스 생성 권한을 부여한다.)

  ```
  # createuser --interactive
  # createdb [데이터베이스명] 
  ```

###NodeJs 설치

- [참고] : [How To Install Node.js on a CentOS 7 server](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-a-centos-7-server)

```
# cd /usr/src
# yum install -y wget
# wget http://nodejs.org/dist/v0.10.30/node-v0.10.30.tar.gz
# tar xzvf node-v* && cd node-v*
# sudo yum install gcc gcc-c++
# ./configure
# make   (6-7분 소요됨)  pc 사양이 좋으면 3분안에 끝나기도 함 ㅎㅎㅎ.
# make install
```

### ImageMagick 설치 (시간 많이 걸리네~)

```
# yum install -y tcl-devel libpng-devel libjpeg-devel ghostscript-devel bzip2-devel freetype-devel libtiff-devel
# wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz

# tar -xzvf ImageMagick.tar.gz
# cd ImageMagick-6.8.8-7
# ./configure --prefix=/usr/local --with-bzlib=yes --with-fontconfig=yes --with-freetype=yes --with-gslib=yes --with-gvc=yes --with-jpeg=yes --with-jp2=yes --with-png=yes --with-tiff=yes

# make
# make install
# convert --version
# yum install -y ImageMagick-devel
```

* `deployer` 계정 홈페이지의 `.bashrc` 파일에 아래를 추가한다.

 ```
 # su deployer
 # vi ~/.bashrc

 export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
 ```


* 그리고, root 계정으로 접속해서

  ```
  # ln -s /usr/local/include/ImageMagick/wand /usr/local/include/wand
  # ln -s /usr/local/include/ImageMagick/magick /usr/local/include/magick
  # ldconfig /usr/local/lib
  # su deployer
  # cd ~
  ```


###배포

: 이후부터는 로컬에서 capistrano로 배포작업을 수행한다.


* 프로젝트의 Gemfile 파일에 아래의 젬을 추가하고 bundle install 한다.

  ```
  gem 'capistrano-rbenv'
  ```


* `capistrano`와 `capistrano-rbenv` 젬을  레일스 프로젝트의 `gemfile`에 추가하여, `bundle install` 하면, `capistrano`로 배포시, 특히, `cap deploy:setup` 명령을 실행하면, 자동으로 `rbenv`와 `ruby-build`가 설치되고, 디폴트로 `1.9.3-p194` 버전 루비가 설치된다. `config/deploy.rb` 파일에 `set :rbenv_ruby_version, "1.9.3-p286"` 와 같이 추가해 주면, 원하는 루비 버전을 설치할 수도 있다. [ref.](https://github.com/yyuu/capistrano-rbenv)

<hr>

###부록

: 아래의 설치는 옵션이다.

###Postfix 설치


: 서버에서 이메일을 보내고 싶은 경우 설치한다.
* [Gmail을 이용해서 postfix 설정하기](http://freelinuxtutorials.com/quick-tips-and-tricks/configure-postfix-to-use-gmail-in-rhelcentos/)

  ```
  # yum install -y postfix
  ```

###Git 업그레이드하기

: 참고 - http://tecadmin.net/how-to-upgrade-git-version-1-7-10-on-centos-6

```
# rpm -i 'http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm'
# rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt
```


아래와 같이 rpmforge.repo 파일을 열어서 [rpmforge-extras] 부분의 enabled=0 to 1로 변경한다.

```
# vim /etc/yum.repos.d/rpmforge.repo
```


###Git 별도설치


: `git`를 별도록 설치할 때 아래와 같이 한다.
* `git`를 소스컴파일로 설치한다. ([ref.](http://rajivpandit.wordpress.com/2013/09/21/how-to-install-git-1-8-4-and-git-client-on-linux-centos-6-4-rhel6/)) 깔금하게 설치된다.
* 가장 최신버전 1.9.0 (3월 7일, 2014년 현재)
( [버전별 git 다운로드 리스트 보기](http://code.google.com/p/git-core/downloads/list) )

  ```bash
  # yum install gettext-devel expat-devel curl-devel zlib-devel openssl-devel
  # cd /usr/src
  # wget https://www.kernel.org/pub/software/scm/git/git-1.8.4.tar.gz
  # tar xzvf git-1.8.4.tar.gz
  # cd git-1.8.4
  # make prefix=/opt/git all
  # make prefix=/opt/git install
  # export PATH=”/opt/git/bin:$PATH”
  ```

###구글링 자료 모음

- http://www.centos.org [공식홈페이지]
- http://isoredirect.centos.org/centos/6/isos/x86_64/ [다운로드]
- http://faq.hostway.co.kr/story/4510 [CentOS 6.3 10분 안에 설치하기, 한글]
- http://mcchae.egloos.com/10937303 [지훈현서의 CentOS 6.3 미니멀(서버) 설치하기, 한글]


