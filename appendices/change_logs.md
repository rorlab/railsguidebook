# 변경내역

### v1.0.6

"5장. 프로젝트 따라하기" 본문을 따라하면서 설명이 부족했던 부분을 보완하였고 `friendly_id` 젬 사용을 삭제하였습니다. 그리고 소소코드 저장소를 http://github.com/rorlakr/rcafe 로 변경하여 소스코드를 업데이트했습니다. 

### v1.0.5

: 브랜치 기능을 이용하여 버전 관리를 다시 시작합니다. 
2014년12월3일 웹에디터에서도 브랜치 기능을 사용할 수 있는 것을 뒤늦게 알았습니다. master 브랜치에 develop 브랜치를 추가하여 업데이트 작업을 하고 최종 버전업을 master 브랜치로 머지하는 방식으로 진행합니다. 

###**버전관리 중단합니다.** 
: 2014년11월30일 Gitbook Webeditor가 발표되면서 웹에디터에서 수정후 저장하면 바로 Gitbook에 반영이 됩니다. 따라서 더 이상 추가 업데이트 부분을 별도의 버전으로 발표하는 것이 의미가 없게 되었습니다. 

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
