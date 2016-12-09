# 프로젝트 따라하기

이번 장에서는 실제로 레일스 프로젝트를 작성해 가면서 레일스 프레임워크의 다양한 기능을 알아보도록 하겠다.

프로젝트명은 `Rcafe`라고 정하자. 이 프로젝트는 레일스를 이용하여 카페를 구현하는 것이다.

### 개발환경

* [ruby 2.3.3p222](https://www.ruby-lang.org/ko/news/2016/11/21/ruby-2-3-3-released/) : 2016년 11월 21일 릴리스됨.
* [rails v5.0.0.1](http://weblog.rubyonrails.org/2016/8/11/Rails-5-0-0-1-4-2-7-2-and-3-2-22-3-have-been-released/) : 2016년 8월 11일 릴리스됨. 

```bash
$ ruby -v
ruby 2.3.3p222 (2016-11-21 revision 56859)

$ rails -v
Rails 5.0.0.1
```

### 소스 코드

로컬 머신에 `git`이 설치되어 있는 경우 아래와 같이 소스를 받을 수 있다.

```bash
$ git clone https://github.com/rorlakr/rcafe.git
```

### 주요기능

* 회원가입(회원인증 및 권한설정)
* 공지사항/새소식/가입인사 게시판 기능
* 댓글기능
* 태그기능
* 파일 업로드기능
* 관리자기능
  * 게시판관리
  * 회원관리


### 프로젝트의 생성

터미널을 열어 아래와 같이 명령을 실행한다.

```bash
$ rails new rcafe2
Using -d postgresql from /Users/[user-account]/.railsrc
      create
      create  README.md
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
      create  app
      create  app/assets/config/manifest.js
      create  app/assets/javascripts/application.js
      create  app/assets/javascripts/cable.js
      create  app/assets/stylesheets/application.css
      create  app/channels/application_cable/channel.rb
      create  app/channels/application_cable/connection.rb
      create  app/controllers/application_controller.rb
      create  app/helpers/application_helper.rb
      create  app/jobs/application_job.rb
      create  app/mailers/application_mailer.rb
      create  app/models/application_record.rb
      create  app/views/layouts/application.html.erb
      create  app/views/layouts/mailer.html.erb
      create  app/views/layouts/mailer.text.erb
      create  app/assets/images/.keep
      create  app/assets/javascripts/channels
      create  app/assets/javascripts/channels/.keep
      create  app/controllers/concerns/.keep
      create  app/models/concerns/.keep
      create  bin
      create  bin/bundle
      create  bin/rails
      create  bin/rake
      create  bin/setup
      create  bin/update
      create  config
      create  config/routes.rb
      create  config/application.rb
      create  config/environment.rb
      create  config/secrets.yml
      create  config/cable.yml
      create  config/puma.rb
      create  config/spring.rb
      create  config/environments
      create  config/environments/development.rb
      create  config/environments/production.rb
      create  config/environments/test.rb
      create  config/initializers
      create  config/initializers/application_controller_renderer.rb
      create  config/initializers/assets.rb
      create  config/initializers/backtrace_silencers.rb
      create  config/initializers/cookies_serializer.rb
      create  config/initializers/cors.rb
      create  config/initializers/filter_parameter_logging.rb
      create  config/initializers/inflections.rb
      create  config/initializers/mime_types.rb
      create  config/initializers/new_framework_defaults.rb
      create  config/initializers/session_store.rb
      create  config/initializers/wrap_parameters.rb
      create  config/locales
      create  config/locales/en.yml
      create  config/boot.rb
      create  config/database.yml
      create  db
      create  db/seeds.rb
      create  lib
      create  lib/tasks
      create  lib/tasks/.keep
      create  lib/assets
      create  lib/assets/.keep
      create  log
      create  log/.keep
      create  public
      create  public/404.html
      create  public/422.html
      create  public/500.html
      create  public/apple-touch-icon-precomposed.png
      create  public/apple-touch-icon.png
      create  public/favicon.ico
      create  public/robots.txt
      create  test/fixtures
      create  test/fixtures/.keep
      create  test/fixtures/files
      create  test/fixtures/files/.keep
      create  test/controllers
      create  test/controllers/.keep
      create  test/mailers
      create  test/mailers/.keep
      create  test/models
      create  test/models/.keep
      create  test/helpers
      create  test/helpers/.keep
      create  test/integration
      create  test/integration/.keep
      create  test/test_helper.rb
      create  tmp
      create  tmp/.keep
      create  tmp/cache
      create  tmp/cache/assets
      create  vendor/assets/javascripts
      create  vendor/assets/javascripts/.keep
      create  vendor/assets/stylesheets
      create  vendor/assets/stylesheets/.keep
      remove  config/initializers/cors.rb
         run  bundle install
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies......
Installing rake 12.0.0
Using concurrent-ruby 1.0.2
Using i18n 0.7.0
Installing minitest 5.10.1
Using thread_safe 0.3.5
Using builder 3.2.2
Using erubis 2.7.0
Using mini_portile2 2.1.0
Using rack 2.0.1
Using nio4r 1.2.1
Using websocket-extensions 0.1.2
Using mime-types-data 3.2016.0521
Using arel 7.1.4
Using bundler 1.11.2
Using byebug 9.0.6
Installing coffee-script-source 1.11.1
Using execjs 2.7.0
Using method_source 0.8.2
Installing thor 0.19.4
Using debug_inspector 0.0.2
Using ffi 1.9.14
Using multi_json 1.12.1
Using rb-fsevent 0.9.8
Using pg 0.19.0
Installing puma 3.6.2 with native extensions
Using sass 3.4.22
Using tilt 2.0.5
Using turbolinks-source 5.0.0
Using tzinfo 1.2.2
Using nokogiri 1.6.8.1
Using rack-test 0.6.3
Using sprockets 3.7.0
Using websocket-driver 0.6.4
Using mime-types 3.1
Using coffee-script 2.4.1
Installing uglifier 3.0.4
Using rb-inotify 0.9.7
Using turbolinks 5.0.1
Using activesupport 5.0.0.1
Using loofah 2.0.3
Using mail 2.6.4
Using listen 3.0.8
Using rails-dom-testing 2.0.1
Using globalid 0.3.7
Using activemodel 5.0.0.1
Installing jbuilder 2.6.1
Using spring 2.0.0
Using rails-html-sanitizer 1.0.3
Using activejob 5.0.0.1
Using activerecord 5.0.0.1
Using spring-watcher-listen 2.0.1
Using actionview 5.0.0.1
Using actionpack 5.0.0.1
Using actioncable 5.0.0.1
Using actionmailer 5.0.0.1
Using railties 5.0.0.1
Using sprockets-rails 3.2.0
Using coffee-rails 4.2.1
Using jquery-rails 4.2.1
Using web-console 3.4.0
Using rails 5.0.0.1
Using sass-rails 5.0.6
```

이어서 rails 젬과 관련 의존성 젬들이 설치되고, 특히, **레일스 4.1 버전**부터는 어플리케이션 프리로더(preloader)인 [`spring`](https://github.com/rails/spring)이 기본적으로 설치되어 있어 커맨드라인 명령어인 `rake`와 `rails` 명령의 실행속도를 빠르게 해 주는데, 실행 로그 마지막에 아래와 같은 간단한 안내문이 나타난다.

```
.
.
.
Bundle complete! 15 Gemfile dependencies, 62 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
         run  bundle exec spring binstub --all
* bin/rake: spring inserted
* bin/rails: spring inserted
```

> **팁:** 계정 루트 디렉토리에 `.railsrc` 파일을 생성하고 `rails new` 명령의 옵션을 추가해 두면, 이 명령이 실행될 때마다 디폴트 옵션으로 사용하게 된다. 
  ```
  $ cat ~/.railsrc
  -d postgresql
  ```

이제 프로젝트 디렉토리로 이동하여 git 초기화한다.

```bash
$ cd rcafe2
$ git init
$ git add .
$ git commit -m "최초커밋"
```

> 소스코드 관리 : 공개 프로젝인 경우에는 github.com 클라우드 저장소를 무료로 사요할 수 있다. 

```
$ bin/rails server
=> Booting WEBrick
=> Rails 4.2.6 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2016-06-10 15:14:37] INFO  WEBrick 1.3.1
[2016-06-10 15:14:37] INFO  ruby 2.3.1 (2016-04-26) [x86_64-darwin15]
[2016-06-10 15:14:37] INFO  WEBrick::HTTPServer#start: pid=91093 port=3000
```

`Booting WEBrick` : 레일스 프로젝트를 실행하기 위해 로컬 웹서버(WEBrick)를 부팅한다는 것을 표시한다. `WEBrick`은 루비 라이브러리로 간단한 **HTTP 웹서버 서비스**를 제공한다.  

`starting in development on http://localhost:3000` : 레일스 프로젝트는 **3가지 모드**에서 실행할 수 있다. **개발모드**(development), **운영모드**(production), **테스트모드**(test). 따라서 현재 개발모드에서 실행되는 프로젝트를 HTTP 프로토콜을 이용하여 `localhost`의 3000포트에서 시작한다는 것을 의미한다.

`rails server -h` 와 같이 `-h` 옵션을 사용하여 서버를 구동하면 여러가지 시작 옵션을 볼 수 있다.

```bash
$ bin/rails s -h
Usage: rails server [mongrel, thin, etc] [options]
    -p, --port=port                  Runs Rails on the specified port.
                                     Default: 3000
    -b, --binding=ip                 Binds Rails to the specified ip.
                                     Default: 0.0.0.0
    -c, --config=file                Use custom rackup configuration file
    -d, --daemon                     Make server run as a Daemon.
    -u, --debugger                   Enable the debugger
    -e, --environment=name           Specifies the environment to run this server under (test/development/production).
                                     Default: development
    -P, --pid=pid                    Specifies the PID file.
                                     Default: tmp/pids/server.pid

    -h, --help                       Show this help message.
```

자주 사용하는 옵션에 대해서 간단히 설명한다. 

* `-p` : 레일스 서버를 특정 포트에서 실행할 때 사용한다. 디폴트로는 3000 포트를 사용한다. 로컬에서 하나의 이상의 프로젝트를 실행하고자 할 때 각기 다른 포트를 사용하면 편리하다. 예, `-p 4000`
* `-b` : 로컬호스트 외에 특정 ip로 연결하고자 할 때 사용한다. 예를 들어, 가상머신에서 레일스 서버를 시작하고 호스트 머신에서 브라우저로 접근하고자 할 때 이 옵션에 가상머신의 ip을 지정하여 연결할 수 있다. 예, `-b 192.168.56.101`
* `-d` : 레일스 서버를 데몬으로 실행할 때 사용한다.
* `-e` : 서버 실행 환경을 지정할 때 사용한다. 디폴트는 `development`이다. 운영환경에서 실행할 때는 `-e production`과 같이 옵션을 지정하면 된다. 

이제 브라우저에서 http://localhost:3000 주소로 확인할 수 있다.

![localhost:3000](http://i1373.photobucket.com/albums/ag392/rorlab/localhost_3000_zps48e7d0ba.png)

이제 터미널에서 소스관리를 위해 `git`을 초기화한 후 커밋한다.

```bash
$ git init
$ git add .
$ git commit -m "최초 커밋"
```

이로써, `master` 브랜치에 지금까지 작업한 내용이 커밋된 것이다.


지금까지 `rcafe`라는 레일스 프로젝트를 생성하여 로컬 웹서버를 구동하고 웹브라우저에서 확인하는 작업까지 진행했고, 또한 작업 내용을 git을 초기화한 후 최초 커밋을 하였다.

다음은 `rcafe` 프로젝트에서 사용할 젬들을 추가하고 설치하도록 하자.



---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05



