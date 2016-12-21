# 서버로 배포하기

우선 서버 셋팅이 완료된 상태로 가정하고 [Capistrano 3](http://capistranorb.com)를 이용하여 배포하는 과정을 진행해 보기로 한다.


### 서버 셋팅

배포할 서버에는 아래와 같은 프로그램을 미리 설치해 둔다. 여기서는 테스트 목적으로 배포할 것이기 때문에, 가상서버를 준비하기로 한다. 가상머신용 툴로는 [`VMware Fusion`](http://www.vmware.com/kr/products/fusion/) v8.5.4를 사용하였다. 무료로 사용할 수 있는 툴로는 오라클사의 `VM VirtualBox`가 있으며 [여기](https://www.virtualbox.org/wiki/Downloads)를 방문하면 패키지를 다운로드 받아 설치할 수 있다. 

또한, [우분투 16.04 서버 셋팅하기](/appendices/ubuntu16server.md)를 참고하면 `VirtualBox`로 가상 서버를 생성하여 서버 환경을 구축할 수 있다.

가상머신의 시스템에는 아래의 프로그램이 설치된다.

* git : 소스관리
* Nginx : 웹서버
* ImageMagick : 이미지 작업을 위한 툴
* MySQL/PostgreSQL : 데이터베이스 서버
* Nodejs : 서버사이드 자바스크립트

> #### Caution::주의
> 
> `MySQL DB 서버`를 사용할 경우 원격서버의 `/etc/mysql/conf.d/` 디렉토리에 `deployer.cnf` 파일을 새로 추가하고 아래와 같이 문자 인코딩을 `uft8`로 추가해 주어야 한글 인코딩 문제를 해결할 수 있다.
```
root@ubuntu $ sudo vi /etc/mysql/conf.d/deployer.cnf
[mysqld]
character-set-server = utf8
```

그리고 배포 전용 계정(`'deployer'`)도 추가해 둔다.

> #### Note::노트
> 
> 아직 서버에 배포용 계정을 생성하지 않았다면 서버로 접속한 후 아래와 같이 계정을 생성한다.
```bash
root@ubuntu $ sudo adduser deployer
root@ubutnu $ sudo addgroup admin
root@ubuntu $ sudo usermod -a -G admin deployer
```

서버로 접속해서 `nginx.conf` 파일의 `user` 속성을 변경한다.

```bash
root@ubuntu $ sudo vi /etc/nginx/nginx.conf
user deployer;
```

배포시에 `sudo` 실행시 `deployer` 계정에 대한 비밀번호를 묻게 되는데 이를 방지 하기 위해서, 서버로 접속한 후에 아래와 같이 명령을 실행하고,

```bash
root@ubuntu $ sudo visudo
```

맨 아래 줄에 아래와 같이 추가해 준다.

```
deployer ALL=(ALL) NOPASSWD: ALL
```

또한, 매번 `ssh`로 서버에 접속할 때마다 비밀번호를 입력하는 절차를 생략하기 위해서, 아래와 같이 `ssh` 키를 서버로 복사한다. 

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa deployer@ubuntu.vm
deployer@ubuntu.vm's password:
```

> #### Note::노트 
> 
> 해당 디렉토리에 `id_rsa` 파일이 없다면 [SSH Key 생성하기](http://webdir.tistory.com/200)를 참고하여 생성한다.

이후부터는 `ssh`로 연결시에 비밀번호 입력없이 바로 접속이 된다.

> #### Note::노트 
> 
> 여기서 사용한 `ubuntu.vm`은 가상머신으로 설치한 우분투 서버의 가상 도메인이다. 도메인 대신 서버의 ip를 지정해도 된다. 맥 사용자들은 [`Horst`](http://penck.de/horst/)라는 툴을 사용하면 `/etc/hosts` 파일을 GUI로 편하게 관리할 수 있다.

제대로 설정이 되었다면 아래의 명령을 실행했을 때 비밀번호 입력없이 결과를 확인할 수 있다.

```bash
$ ssh deployer@ubuntu.vm 'hostname; uptime'
ubuntu
 10:26:54 up  3:49,  1 user,  load average: 0.02, 0.02, 0.05
```

> #### Caution::주의
> 
> 복수개의 서버를 설치 중이라면 모든 서버에서 위의 작업을 해 주어야 한다.


### database.yml 설정

운영 데이터베이스 서버의 셋팅을 변경하기 위해서, `config/database.yml` 파일을 열고 아래와 같이 수정한다.

```ruby
default: &default
  adapter: sqlite3
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db/development.sqlite3

test:
  <<: *default
  database: db/test.sqlite3

production:
  adapter: mysql2
  encoding: utf8
  pool: 5
  database: rcafe_production
  username: deployer
  password: <%= ENV["RCAFE_DATABASE_PASSWORD"] %>
  host: localhost
```

### Gemfile의 추가

프로젝트의 `Gemfile`을 열고 파일 하단에 코멘트 처리되어 있는 두개의 젬을 활성화하고, `Capistrano` 관련 젬을 추가한 후 데이터베이스 젬을 환경에 맞게 추가한다.

```ruby
# 추가할 젬
# Use Capistrano for deployment
gem 'capistrano-rbenv', '~> 2.0'
gem 'capistrano-rbenv-install', '~> 1.2.0'
gem 'capistrano-rails', group: :development
gem 'capistrano3-puma' , group: :development
gem 'capistrano-figaro-yml', '~> 1.0.2'
gem 'capistrano-upload-config'
gem 'capistrano3-nginx', '~> 2.0'

프로젝트에 적용하기 위해서 번들 인스톨한다.

```bash
$ bin/bundle install
```

### Capistrano의 초기화
그리고 프로젝트에 사용하기 위해서 `Capistrano`를 초기화한다.

```bash
$ cap install
mkdir -p config/deploy
create config/deploy.rb
create config/deploy/staging.rb
create config/deploy/production.rb
mkdir -p lib/capistrano/tasks
Capified
```

### Capistrano의 배포 환경설정

위에서 생성된 `config/deploy.rb` 파일을 열고 아래와 같이 변경한다.

```ruby
# config valid only for current version of Capistrano
lock '3.6.1'

set :application, 'rcafe2'
set :repo_url, 'git@github.com:rorlab/rcafe2.git'

set :migration_role, :app

set :rbenv_type, :user # or :system, depends on your rbenv setup
set :rbenv_ruby, File.read('.ruby-version').strip

set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
set :rbenv_map_bins, %w{rake gem bundle ruby rails}
set :rbenv_roles, :all # default value

# Default branch is :master
# ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

# Default deploy_to directory is /var/www/my_app_name
set :deploy_to, '/home/deployer/apps/rcafe2'

# Default value for :scm is :git
# set :scm, :git

# Default value for :format is :airbrussh.
# set :format, :airbrussh

# You can configure the Airbrussh format using :format_options.
# These are the defaults.
# set :format_options, command_output: true, log_file: 'log/capistrano.log', color: :auto, truncate: :auto

# Default value for :pty is false
set :pty, true

# Default value for :linked_files is []
set :linked_files, fetch(:linked_files, []).push('config/database.yml', 'config/secrets.yml')

# Default value for linked_dirs is []
set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'public/system')

# Default value for default_env is {}
# set :default_env, { path: "/opt/ruby/bin:$PATH" }

# Default value for keep_releases is 5
# set :keep_releases, 5

before 'deploy:check:linked_files', 'config:push'
before 'deploy:starting', 'figaro_yml:setup'
after 'figaro_yml:setup', 'puma:nginx_config'

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

#### 운영서버 배포 환경

다음 `config/deploy/production.rb` 파일을 열고 아래와 같이 변경한다.

```ruby
role :app, %w{deployer@ubuntu.vm}
role :web, %w{deployer@ubuntu.vm}
role :db,  %w{deployer@ubuntu.vm}

set :nginx_server_name, 'ubuntu.vm'
set :unicorn_workers, 4

server 'ubuntu.vm', user: 'deployer', roles: %w{web app}
```

최적의 `:unicorn_workers` 수를 정하기 위해서 참고할 만한 글을 [여기](https://www.digitalocean.com/community/tutorials/how-to-optimize-unicorn-workers-in-a-ruby-on-rails-app)를 참고하기 바란다. 여기서는 `4`로 지정했다. 각자의 서버 환경에 따라 적절하게 조절할 필요가 있다.


#### secrets.yml 파일의 옵션 변경

`config/secrets.yml` 파일을 열고 아래와 같이 변경한다.

```ruby
production:
  secret_key_base: <%= ENV["RCAFE_SECRET_KEY_BASE"] %>
```

#### Capfile 설정


위에서 `RCAFE_`를 환경변수 이름 앞에 추가한다.


`Capfile`을 열고 아래와 같이 변경한다.

```ruby
# Load DSL and Setup Up Stages
require 'capistrano/setup'

# Includes default deployment tasks
require 'capistrano/deploy'

# Includes tasks from other gems included in your Gemfile
require 'capistrano/rbenv'
require 'capistrano/rbenv_install'
require 'capistrano/bundler'
require 'capistrano/rails/assets'
require 'capistrano/rails/migrations'
require 'capistrano/rails/console'
require 'capistrano/rails/collection'
require 'capistrano/unicorn_nginx'
require 'capistrano/rails_tail_log'

# Loads custom tasks from `lib/capistrano/tasks' if you have any defined.
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```

#### 서버 시스템에 환경변수 지정

서버에 접속한 후 `/etc/environment` 파일을 열고 아래와 같이 추가한다.

```bash
RCAFE_SECRET_KEY_BASE='xxxxxxxxxxxxxx'
RCAFE_DATABASE_PASSWORD='xxxxxxx'
```

> **Note** 로컬 프로젝트 디렉토리에서 아래와 같이 명령을 실행하면 `secret` 키를 생성할 수 있다
```bash
$ bin/rake secret
```
이 때 생성된 키를 위의 `RCAFE_SECRET_KEY_BASE` 값으로 할당하면 된다.

#### 데이터베이스의 생성

서버에 접속하여 아래와 같이 데이터베이스를 생성하고 권한을 부여한다.

```bash
deployer@ubuntu $ mysql -u root -p
mysql> create database rcafe_production;
mysql> grant usage on *.* to deployer@localhost identified by 'password';
mysql> grant all privileges on rcafe_production.* to deployer@localhost;
```

### 운영서버의 셋업

배포 전에 아래와 같은 명령으로 서버의 셋팅 작업을 한다.

```bash
$ cap production setup
```

아래와 같이 루비 2.2.0 설치 중 에러가 발생하면,

```bash
INFO [b3b4b5bb] Running /usr/bin/env ~/.rbenv/bin/rbenv install 2.2.0 as deployer@ubuntu14.vm
DEBUG [b3b4b5bb] Command: /usr/bin/env ~/.rbenv/bin/rbenv install 2.2.0
DEBUG [b3b4b5bb] 	Downloading ruby-2.2.0.tar.gz...
DEBUG [b3b4b5bb] 	-> http://dqw8nmjcqpjn7.cloudfront.net/7671e394abfb5d262fbcd3b27a71bf78737c7e9347fa21c39e58b0bb9c4840fc
DEBUG [b3b4b5bb] 	Installing ruby-2.2.0...
DEBUG [b3b4b5bb]
DEBUG [b3b4b5bb] 	BUILD FAILED
DEBUG [b3b4b5bb] 	 (Ubuntu 14.10 using ruby-build 20150130)
DEBUG [b3b4b5bb]
DEBUG [b3b4b5bb] 	Inspect or clean up the working tree at /tmp/ruby-build.20150202122321.28969
DEBUG [b3b4b5bb] 	Results logged to /tmp/ruby-build.20150202122321.28969.log
DEBUG [b3b4b5bb]
DEBUG [b3b4b5bb] 	Last 10 log lines:
DEBUG [b3b4b5bb] 	./libffi-3.2.1/.libs/libffi.a: error adding symbols: Bad value
DEBUG [b3b4b5bb] 	collect2: error: ld returned 1 exit status
DEBUG [b3b4b5bb] 	Makefile:325: recipe for target '../../.ext/x86_64-linux/fiddle.so' failed
DEBUG [b3b4b5bb] 	make[2]: *** [../../.ext/x86_64-linux/fiddle.so] Error 1
DEBUG [b3b4b5bb] 	make[2]: Leaving directory '/tmp/ruby-build.20150202122321.28969/ruby-2.2.0/ext/fiddle'
DEBUG [b3b4b5bb] 	exts.mk:177: recipe for target 'ext/fiddle/all' failed
DEBUG [b3b4b5bb] 	make[1]: *** [ext/fiddle/all] Error 2
DEBUG [b3b4b5bb] 	make[1]: Leaving directory '/tmp/ruby-build.20150202122321.28969/ruby-2.2.0'
DEBUG [b3b4b5bb] 	uncommon.mk:187: recipe for target 'build-ext' failed
DEBUG [b3b4b5bb] 	make: *** [build-ext] Error 2
```

서버에 접속하여 아래와 같이 `libffi-dev` 라이브러리를 설치하고 다시 시도한다.  

```bash
deployer@ubuntu $ sudo apt-get libffi-dev
```

### 배포하기

이제 실제 배포 명령을 실행한다.

```bash
$ cap production deploy
```

> #### Danger::버그
> 
> 이 때 아래와 같은 에러가 발생하고 중단할 경우에는 배포서버로 접속한 후 `'...'` 내용을 복사해서 실행하고 한번더 **deploy**하면 해결된다. 아직 이런 현상의 이유를 알 수 없다.
  ```
  Couldn't reload, starting 'cd /home/deployer/apps/blog/current && ( RBENV_ROOT=~/.rbenv RBENV_VERSION=2.1.2 RBENV_ROOT=~/.rbenv RBENV_VERSION=2.1.2 ~/.rbenv/bin/rbenv exec bundle exec unicorn -D -c /home/deployer/apps/blog/shared/config/unicorn.rb -E production )' instead
  ```

마지막으로 한가지 추가할 것은 서버에서 `rake db:seed`가 실행되도록 하여 기본 게시판을 생성하도록 한다. 

```bash
$ cap production rails:rake:db:seed
```

에러 없이 배포가 완료되면 브라우저에서 확인한다.

### 소스코드의 관리

* 이후 소스변경이 필요한 경우 커밋 후 `git push`한 다음, `cap production deploy` 명령을 실행하면 된다.
* 잘 못 배포된 경우에는 `cap production deploy:rollback` 명령으로 취소할 수 있다.


지금까지 설명한 배포과정이 다소 길고 복잡한 감이 있다. 또한 각자의 환경에 따라 예상치 못했던 여러가지 에러가 발생할 것으로 생각한다. 각자 따라해 보고 문제가 발생하면 아래에 코멘트를 달아 함께 공유하고 고민해 보자.


---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_16

---

_**References:**_

1. [SSH Key 생성하기](http://webdir.tistory.com/200)
2. [Sudo without password on Ubuntu](http://jeromejaglale.com/doc/unix/ubuntu_sudo_without_password)
3. [NginX 주요 설정 (nginx.conf)](http://sarc.io/index.php/nginx/61-nginx-nginx-conf)
