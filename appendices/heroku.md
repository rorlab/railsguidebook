# Heroku(허로쿠)에 배포하기

공식웹사이트 : https://www.heroku.com

허로쿠 서비스를 이용하면 `Capistrano`를 사용하지 않은 상태에서 레일스 프로젝트를 손쉽게 배포할 수 있다. 허로쿠 서비스는 개발자가 프로젝트 개발에만 집중할 수 있도록 해 주고 배포 부분은 허로쿠 서비스가 알아서 대신해 준다.


### 허로쿠 준비부터 배포까지

* [허로쿠 공식웹사이트](https://www.heroku.com)를 방문하여 회원가입한다.
* 본인 계정의 `Dashboard`의 하단에 있는 `Create a new app` 버튼을 클릭하여 새로운 어플리케이션을 생성한다. `App` 이름은 본인이 원하는 것으로 해도 무방하다.
* 로컬머신의 커맨드라인 쉘에서 [`허로쿠 툴벨트`](https://toolbelt.heroku.com)(아래 참고)를 설치한다.
* `Gemfile`에 허로쿠 배포에 필요한 두개의 젬을 추가하고,
  ```ruby
  gem 'pg', group: :production
  gem 'rails_12factor', group: :production
  ```
  레일스 5버전부터는 `rails_12factor` 젬이 필요없게 되었다. 

  기존의 `sqlite3` 젬은 아래와 같이 `:group` 옵션을 추가한다
  ```ruby
  gem 'sqlite3', group: :development
  ```

  그리고 번들 인스톨한다. 
  ```bash
  $ bundle install
  ```


* `rails s` 명령으로 개발모드에서 이상없이 작동하는 것이 확인되면 `git`의 원격 저장소 `heroku`를 생성한다.
  ```bash
  $ git remote add heroku git@heroku.com:<heroku-app-name>.git
  ```


* 이제 아래와 같이 허로쿠로 배포작업을 시작한다.

  ```bash
  $ git push heroku master
  ```


* 브라우저에서 `http://<heroku-app-name>.heroku.com`으로 접속해서 확인한다.

* 자세한 내용은 [여기](https://devcenter.heroku.com/articles/quickstart)를 참고하기 바란다.


### 허로쿠에서 레일스 4 프로젝트 배포하기

자세한 내용은 [여기](https://devcenter.heroku.com/articles/getting-started-with-rails4)를 참고하기 바란다.


#### 허로쿠 툴벨트 설치하기

로컬 운영체제에 맞는 [`허로쿠 툴벨트`](https://toolbelt.heroku.com)를 설치한다. 이후에는 터미널에서 헤로쿠에서 `heroku` 명령을 사용할 수 있게 된다

```bash
$ heroku help
Usage: heroku COMMAND [--app APP] [command-specific-options]

Help topics, type "heroku help TOPIC" for more details:

  heroku access        # CLI to manage access in Heroku Applications
  heroku addons        # manage add-ons
  heroku apps          # manage apps
  heroku auth          # authentication (login/logout)
  heroku buildpacks    # manage the buildpacks for an app
  heroku certs         # a topic for the ssl plugin
  heroku config        # manage app config vars
  heroku domains       # manage the domains for an app
  heroku drains        # list all log drains
  heroku features      # manage optional features
  heroku git           # manage local git repository for app
  heroku labs          # experimental features
  heroku local         # run heroku app locally

(생락...)
```

#### 허로쿠에서 App 생성하기

![허로쿠 웹사이트](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-23_17-42-46_zpsdjoors0o.png)

어플리케이션 이름을 입력하고 `Create App` 버튼을 클릭한다. 

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-23_17-44-37_zps1iccwovq.png)

생성 후 아래와 같은 결과를 보게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-23_17-46-49_zps05sq7hcz.png)


#### heroku 원격 브랜치 생성하기

```bash
$ git remote add heroku git@heroku.com:rcafe2.git
```

#### 배포 준비

```bash
$ git add .
$ git commit -m "작업을 내용을 커밋함"
$ git push origin  # 선택사항 : github에 푸시하기
```


#### 허로쿠에 실제 배포하기

```bash
$ git push heroku master  # 허로쿠에 푸시하기
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-23_17-53-40_zpsekl6jinp.png)

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-23_17-55-54_zpscluhwezj.png)


#### 브라우저에서 확인하기

```bash
$ open https://rcafe2.herokuapp.com/
```





