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



- `root` 계정으로 로그인한 후 터미널에서 아래의 작업을 진행한다.

  ```
  # yum -y update
  ```

- 업데이트 진행 중에 다음과 같은 에러가 발생하면

  ```
  Error: Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
  ```
  아래와 같은 조치를 하여 해결할 수 있다. 
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

- `Development Tools` 를 설치할 때 `git`도 함께 설치된다. (2016년 12월 3일  현재 - 1.8.3.1 버전, 그러나 최신버전은 2.11.0)
  > 참고 : `git` 최신버전으로 업그레이드할 경우에는 아래의 `git 업그레이드하기` 가 도움이 될 것이다.  

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
  # systemctl restart sshd
  ```


###Firewall 설정

* 참고 : 
  - [RHEL/CentOS 7 에서 방화벽(firewalld) 설정하기](https://www.lesstif.com/pages/viewpage.action?pageId=22053128)


* 방화벽의 설치 

  ```
  # yum install firewalld
  # systemctl start firewalld
  #  systemctl enable firewalld
  ```

* `/etc/firewalld/zones/public.xml` 파일을 생성하고 아래의 내용을 붙여 넣기 한다. 

  ```
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accept
ed.</description>
  <service name="dhcpv6-client"/>
  <service name="http"/>
  <service name="ssh"/>
  <service name="https"/>
</zone>
  ```

* 그리고 재구동한다. 

  ```
  # firewall-cmd --reload
  ```

* 방화벽 포트 추가 

  ```
  # firewall-cmd --permanent --zone=public --add-service=http
  # firewall-cmd --permanent --zone=public --add-service=https
  ```

* firewalld 재시작

  ```
  # firewall-cmd --reload
  ```

* 제대로 반영되었는지 확인한다.

  ```
  # firewall-cmd --list-services --zone=public
    ===> dhcpv6-client http https ssh
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
  # systemctl enable nginx.service
  # systemctl start nginx.service
  ```


* 설치하고 `/etc/nginx` 디렉토리에 `sites-enabled` 와 `sites-available`폴더를 생성해 준다. 

  ```
  # mkdir -p /etc/nginx/sites-enabled
  # mkdir -p /etc/nginx/sites-available
  ```

* 그리고 `/etc/nginx/nginx.conf` 파일을 열고 32번 줄에 `include /etc/nginx/sites-enabled/*;` 추가하고,

  ```
  # vi /etc/nginx/nginx.conf
  ```

* 최상단에 있는 `user` 값을 `deployer`로 변경한다.


* 이제 파일을 닫고, `nginx`를 시작한다.

  ```
  # systemctl restart nginx.service
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
  # yum install -y postgresql-server postgresql-devel postgresql-contrib
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
  # systemctl enable postgresql
  # systemctl start postgresql
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

- 설치

  ```
  # cd /usr/src
  # yum install -y wget
  # wget http://nodejs.org/dist/v0.10.30/node-v0.10.30.tar.gz
  # tar xzvf node-v* && cd node-v*
  # ./configure
  # make
  # make install
  ```

### ImageMagick 설치 (시간 많이 걸리네~)

* 설치

  ```
  # yum install -y tcl-devel libpng-devel libjpeg-devel ghostscript-devel bzip2-devel freetype-devel libtiff-devel
  # cd /usr/src
  # wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz
  # tar -xzvf ImageMagick.tar.gz
  # cd ImageMagick-7.0.3-9
  # ./configure --prefix=/usr/local --with-bzlib=yes --with-fontconfig=yes --with-freetype=yes --with-gslib=yes --with-gvc=yes --with-jpeg=yes --with-jp2=yes --with-png=yes --with-tiff=yes
  # make
  # make install
  # convert --version
  # yum install -y ImageMagick-devel
  ```

* `deployer` 계정 홈페이지의 `.bashrc` 파일에 아래를 추가한다.

 ```
 # sudo -i -u deployer
 # vi ~/.bashrc

 export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
 ```


* 이제 다시, root 계정으로 빠져나와

  ```
  # exit
  # ln -s /usr/local/include/ImageMagick/wand /usr/local/include/wand
  # ln -s /usr/local/include/ImageMagick/magick /usr/local/include/magick
  # ldconfig /usr/local/lib
  ```

* 서버에서 루비를 설치할 때 필요한 모듈을 추가로 설치한다. 

  ```
  # yum install -y readline-devel openssl-devel
  ```

---

###배포

: 이후부터는 로컬에서 `capistrano`로 배포작업을 수행한다.


* 프로젝트 `Gemfile` 파일에 추가해야할 최소한의 젬.

  ```
  gem 'capistrano-rbenv', '~> 2.0'
  gem 'capistrano-rbenv-install', '~> 1.2.0'
  gem 'capistrano-rails', group: :development
  gem 'capistrano3-puma' , group: :development
  gem 'capistrano-figaro-yml', '~> 1.0.2'
  ```


* `bundle install` 명령을 실행한 후 프로젝트를 `capify`한다. 

  ```
  $ cap install
  ├── Capfile
  ├── config
  │   ├── deploy
  │   │   ├── production.rb
  │   │   └── staging.rb
  │   └── deploy.rb
  └── lib
      └── capistrano
              └── tasks
  ```

* `Capfile`은 다음과 같이 작성한다. 

  ```
  # Load DSL and set up stages
  require 'capistrano/setup'

  # Include default deployment tasks
  require 'capistrano/deploy'

  # Include tasks from other gems included in your Gemfile
  require 'capistrano/rbenv'
  require 'capistrano/rbenv_install'
  require 'capistrano/bundler'
  require 'capistrano/rails/assets'
  require 'capistrano/rails/migrations'
  require 'capistrano/puma'
  require 'capistrano/figaro_yml'
  require 'capistrano/puma/nginx'   # if you want to upload a nginx site template

  # Load custom tasks from 'lib/capistrano/tasks' if you have any defined
  Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
  ```

* `config/deploy/staging.rb` 파일은 다음과 같이 작성한다. 

  ```
  server '[server-ip]', user: 'deployer', roles: %w{app db web}
  set :nginx_server_name, '[domain-name]'
  ```

* 원격 서버에서 사용할 환경변수를 지정한다. 여기서는 [`capistrano-figaro-yml`](https://github.com/chouandy/capistrano-figaro-yml) 젬을 사용하였다. 우선 `config/application.yml` 파일을 생성한 후 아래와 같이 작성한다. 

  ```
  staging:
    DATABASE_USERNAME: [username]
    DATABASE_PASSWORD: [password]
    SECRET_KEY_BASE: 9cf54********ee00f66162b7f0ce12********789b5ce782a8b1fb95d********06d78608eb9bc*********2abb
  ```

  > 주의 : 이 파일은 소스관리에 포함해서는 안된다는 것이다. 따라서 `.gitignore` 파일에 추가한다.

* 위에서 `Capfile`에 이미 아래와 같이 추가하였다. 

  ```
  require 'capistrano/figaro_yml'
  ```

* `staging` 서버로 `config/application.yml` 파일을 업로드하기 위해서 다음과 같이 명령을 실행한다.

  ```
  $ cap staging setup
  ```

* `config/database.yml` 파일은 다음과 같이 작성한다.

  ```
  default: &default
    adapter: postgresql
    encoding: unicode
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

  staging:
    <<: *default
    database: blog5_staging
    username: <%= ENV['DATABASE_USERNAME'] %>
    password: <%= ENV['DATABASE_PASSWORD'] %>
  ```

* `config/secrets.yml` 파일은 다음과 같이 작성한다. 

  ```
  staging:
    secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
  ```

* `config/deploy.rb` 파일은 다음과 같이 작성한다. (디폴트 설정은 생략하였다)

  ```
  lock '3.5.0'
  
  set :application, '[project-name]'
  set :repo_url, 'git@bitbucket.org:[git-account]/[project-name].git'
  
  set :migration_role, :app
  
  set :rbenv_type, :user 
  set :rbenv_ruby, File.read('.ruby-version').strip
  
  set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
  set :rbenv_map_bins, %w{rake gem bundle ruby rails}
  set :rbenv_roles, :all # default value
    
  set :deploy_to, '/home/[deploy-account]/apps/[project-name]'
  
  set :pty, true
  
  set :linked_files, fetch(:linked_files, []).push('config/database.yml', 'config/secrets.yml')
  
  set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'public/system')
  
  before 'deploy:starting', 'figaro_yml:setup'
  
  namespace :deploy do
    after :restart, :clear_cache do
      on roles(:web), in: :groups, limit: 3, wait: 10 do
        # Here we can do anything such as:
        # within release_path do
        #   execute :rake, 'cache:clear'
        # end
      end
    end
  
  end
  ```

  위의 코드 중의 `[...]` 부분은 각자의 상황에 맞는 값으로 변경한다.(5군데)

* 실제 staging 서버로 배포 명령은 실행하는 순서는 다음과 같다. 

  ```
  $ cap staging deploy:check
  $ cap staging doctor
  $ cap staging deploy 
  ```  

* 이후 코드 수정후 반복해서 배포할 때는 마지막에 실행한 명령을 반복해서 실행하면 된다. 





---

###부록

: 아래의 설치는 옵션이다.

###Sendmail 설치

- 참고 
  - [How to Install Sendmail Server on CentOS/RHEL 7/6/5](http://tecadmin.net/install-sendmail-server-on-centos-rhel-server/)
  

- 설치 

  ```
  # yum install sendmail sendmail-cf m4
  ```

### Git 업그레이드하기

- 참고
  - [How to install the latest GIT version on CentOS](https://www.howtoforge.com/how-to-install-the-latest-git-version-on-centos)
  - [“Can’t locate ExtUtils/MakeMaker.pm” while compile git](https://madcoda.com/2013/09/cant-locate-extutilsmakemaker-pm-while-compile-git/)
  - [Uninstall git on linux, fedora](http://superuser.com/questions/973961/uninstall-git-on-linux-fedora)


- 설치 및 업그레이드하기

  ```
  # yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils
  # cd /usr/src
  # wget https://www.kernel.org/pub/software/scm/git/git-2.11.0.tar.gz
  # tar xzf git-2.11.0.tar.gz
  # cd git-2.11.0
  # make prefix=/usr/local/git all
  # make prefix=/usr/local/git install
  # echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
  # source /etc/bashrc
  ```
  > 설치 시점에서의 최신 버전으로 파일명을 변경하면 된다.

- 기존의 `git` 은 삭제한다.

  ```
  # yum remove git
  # yum clean all
  ```

  > `git version` 명령을 실행하여 최신 버전을 확인한다. 이전 버전으로 표시될 경우에는 다시 로그인하면 제대로 반영된다. 

- git 설정
  : 최초 설치시 다음과 같이 사용자 정보를 설정한다.

  ```
  # git config --global user.name "Your Name"
  # git config --global user.email "you@example.com"
  # git config -l
  ```

### CentOS 7 시스템 시간의 동기화

* 서버시간과 실제시간을 동기화 해 주어야 시간이 제대로 표시된다. 

* 참고 
  - [CentOS 7 | ntp로 시간 동기화 하는 방법](https://www.cmsfactory.net/node/11424)

* 우선 ntp를 설치한다..

  ```
  # yum install ntp
  ```

* 방화벽을 설치한다.

  ```
  # firewall-cmd --add-service=ntp --permanent
  ```

* 방화벽을 다시 로드한다. 

  ```
  # firewall-cmd --reload
  ```

* ntp 서비스 시작
  
  ```
  # systemctl start ntpd
  ```

* 시스템 재부팅 후에도 자동으로 시작할 수 있도록 한다.

  ```
  # systemctl enable ntpd
  ```

* 이제 터미널 에서 `date` 명령을 실행하여 로컬 시간이 제대로 설정되었는지 확인한다.

  ```
  # date
  ```


