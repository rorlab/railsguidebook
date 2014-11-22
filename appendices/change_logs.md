# 변경내역

### v1.0.4

* 부록에 [**우분투 14.04 서버 세팅하기 (Virtual Box)**] 챕터 추가함. - [rorlab](mailto:rorlab@gmail.com) 2014년 10월 9일 한글날.
* [**5.17. 서버로 배포하기**] : 서버의 tail log 기능 추가함 - [rorlab](mailto:rorlab@gmail.com) 2014년 10월 9일
  * Gemfile : gem 'capistrano-rails-tail-log' 추가
  * Capfile : require 'capistrano/rails_tail_log' 추가
  * deploy.rb : set :rails_env, "production" 추가

### v1.0.3

#### 추가내용

* [추천 웹사이트 및 블로그]
* [**5.17 서버로 배포하기** / **Gemfile의 추가** : `rb-readline` 젬 추가함. [참고문서](http://vvv.tobiassjosten.net/ruby/readline-in-ruby-with-rbenv/)]
* [**5.17 서버로 배포하기** / **배포하기** : **버그** 해결법 추가.] - [rorlab](mailto:rorlab@gmail.com) 2014년 9월 28일

### v1.0.2

#### 포맷변경

* PDF 파일로 변환될 때 소제목 시작시에 새로운 페이지로 넘어가는 것을 수정함. [Lucius Choi](maitlto:lucius.choi@gmail.com) 2014년 9월 11일

#### 오타수정

* [코멘트 변경하기], [Dev_senna](mailto:dev.senna@gmail.com ) 2014년 8월 22일

```
1. 자바스크리트가…. => 자바스크립트가
```

### v1.0.1

#### 오타수정

* [코멘트 변경하기], [rkjun](mailto:juntai81@gmail.com) 2014년 8월 9일

```
1. .... 자동으로 comments라는 테이블명으로 테이블이 생선된다는 것이다.
생선 => 생성
2. app/controllers/comment_controller.rb 파일을 열어 보면 아래와 같다.
comment_controller.rb => comments_controller.rb
```

* [코멘트 변경하기], [효링](mailto:dev.jinak@gmail.com) 2014년 8월 15일

```
1. 컨트롤러에 def comment_params 에 params 오타있습니다. parmas 로 되어있네요.
```


#### 변경 및 추가

* [변경내역] 메뉴 추가, [rorlab](mailto:rorlab@gmail.com) 2014년 8월 9일
