# 첫번째 모델의 생성

`Post`라는 모델을 작성해 보자. 이 모델은 우선 `title`(글제목)과 `content`(글내용) 두개의 속성으로 구성하자. 나중에 필요한 속성을 추가할 것이다. 커맨드라인에서 아래와 같이 명령을 실행하자.

```bash
$ bin/rails generate scaffold Post title content:text
Running via Spring preloader in process 18827
      invoke  active_record
      create    db/migrate/20161210103346_create_posts.rb
      create    app/models/post.rb
      invoke    test_unit
      create      test/models/post_test.rb
      create      test/fixtures/posts.yml
      invoke  resource_route
       route    resources :posts
      invoke  scaffold_controller
      create    app/controllers/posts_controller.rb
      invoke    erb
      create      app/views/posts
      create      app/views/posts/index.html.erb
      create      app/views/posts/edit.html.erb
      create      app/views/posts/show.html.erb
      create      app/views/posts/new.html.erb
      create      app/views/posts/_form.html.erb
      invoke    test_unit
      create      test/controllers/posts_controller_test.rb
      invoke    helper
      create      app/helpers/posts_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/posts/index.json.jbuilder
      create      app/views/posts/show.json.jbuilder
      create      app/views/posts/_post.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/posts.coffee
      invoke    scss
      create      app/assets/stylesheets/posts.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
```

위에서 보는 바와 같이, 간단한 커맨드라인 명령으로 다양한 리소스 모듈들이 호출되고 연관 템플릿 파일들이 생성되었다.

> #### Note::노트
> 
>  하단에 있는 `scaffold.css.scss` 파일은 불필요하기 때문에 파일을 에디터로 열고 모든 스타일을 코맨트 처리하거나 삭제한다. 또는 `application.scss` 파일에서 `import` 하지 않는다.

`MVC(Model-View-Controller)` 디자인 패턴에 따라 생성된 파일들을 분류해 볼 수 있는데, 우선 위의 실행 결과물의 세번째 줄에 있는 마이그레이션 파일 `20140501054730_create_posts.rb`에 주목하자. 이 파일의 내용은 아래와 같다. (`20140501054730` 값은 상황에 따라 다를 수 있다.)

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.text :content

      t.timestamps
    end
  end
end
```

`rake` 명령의 여러가지 `task` 중 `db:migrate` 작업을 실행할 때 `rake`가 이 파일은 사용하게 되며 실행결과 데이터베이스 테이블을 생성하게 된다. 이 때 테이블의 이름은 레일스의 `C.O.C`에 따라 모델명(Post)의 복수형(posts)으로 자동 지정된다.

```bash
$ bin/rake db:migrate
== 20150130063424 CreatePosts: migrating ======================================
-- create_table(:posts)
   -> 0.0010s
== 20150130063424 CreatePosts: migrated (0.0011s) =============================
```

### 마이그레이션 상태 확인

이와 같은 마이그레이션 작업의 수행결과는 히스토리로 관리되는데 아래와 같이 커맨드라인 명령으로 확인할 수 있다.

```bash
$ in/rake db:migrate:status

database: /Users/rorlakr/rcafe/db/development.sqlite3

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20150130063424  Create posts
```

#### 상태(status)

각 컬럼에 대해서 간략하게 설명하고 넘어가자.
`Status`는 마이그레이션의 상태를 나타낸다. `up`은 마이그레이션이 실행된 상태이고, `down`으로 표시된 경우는 아직 마이그레이션이 수행되지 않는 상태를 의미한다. 아래와 같이 방금 전에 수행했던 마이그레이션을 취소(롤백, rollback)해 보자.

```bash
$ bin/rake db:rollback
== 20150130063424 CreatePosts: reverting ======================================
-- drop_table(:posts)
   -> 0.0006s
== 20150130063424 CreatePosts: reverted (0.0045s) =============================
```

이 결과로 데이터베이스에서 `posts` 테이블이 `drop`된다. 그리고 마이그레이션 상태를 확인하면,

```bash
$ bin/rake db:migrate:status

database: /Users/rorlakr/rcafe/db/development.sqlite3

 Status   Migration ID    Migration Name
--------------------------------------------------
  down    20150130063424  Create posts
```

`$ bin/rake db:migrate:redo` 명령으로 취소했던 마이그레이션을 다시 실행할 수 있다.

```bash
$ bin/rake db:migrate:redo
== 20150130063424 CreatePosts: migrating ======================================
-- create_table(:posts)
   -> 0.0014s
== 20150130063424 CreatePosts: migrated (0.0015s) =============================
```

마이그레이션이 다시 실행된 상태('up')를 확인할 수 있다.

```bash
$ bin/rake db:migrate:status

database: /Users/rorlakr/rcafe/db/development.sqlite3

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20150130063424  Create posts
```

#### 마이그레이션 ID (Migration ID)

이것은 마이그레이션 작업의 고유 번호다. 이 값은 마이그레이션 파일이 생성될 때 자동으로 파일명 앞에 붙는 타임스탬프로 고유한 값을 가진다. 예를 들어, 위에서 사용된 `20140501054730_create_posts.rb` 파일의 파일명 시작부분에 있는 숫자가 이에 해당한다.

#### 마이그레이션 이름(Migration Name)

이것은 마이그레이션 파일명의 타임스탬프를 제외한 부분에서 발췌한 것이다.

#### schema_migrations 테이블

데이터베이스 마이그레이션 작업은 `version`이라는 하나의 속성만을 가지는 `schema_migrations`라는 테이블에 마이그레이션 ID 값이 저장된다. 위에서 언급했던 마이그레이션 상태(Status)는 해당 마이그레이션 ID 값이 `schema_migrations` 테이블에 존재할 때 `up` 상태로 표시되고 없을 경우에 `down` 상태로 표시된다. 이를 확인하기 위해서 **레일스 DB 콘솔**로 접근해서 해당 테이블의 값을 조회해 보자.

```bash
$ bin/rails db
SQLite version 3.8.5 2014-08-15 22:37:57
Enter ".help" for usage hints.
sqlite> select version from schema_migrations;
20150130063424
```

### 리소스 라우팅

이제 브라우저에서 http://localhost:3000/posts 로 접근하면 아래와 같은 화면을 볼 수 있다. 

![posts_index](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_15-47-26_zpsd0567b97.png)


위에서 `Post` 모델을 **scaffold 제너레이터**를 이용하여 생성할 때 콘솔 출력내용 중 아래와 같은 부분을 발견할 수 있다.

```bash
$ bin/rails generate scaffold Post title content:text
       ...
      invoke  resource_route
       route    resources :posts
       ...
```

즉, `resource_route` 모듈을 호출하여 `config/routes.rb` 파일에 `resources :posts` 라인을 추가한다. 이와 같이 라우팅을 선언하는 방법을 `리소스 라우팅`이라고 한다. 이로서 아래와 같은 라우팅을 사용할 수 있게 된다.

```bash
$ bin/rake routes
   Prefix Verb   URI Pattern               Controller#Action
    posts GET    /posts(.:format)          posts#index
          POST   /posts(.:format)          posts#create
 new_post GET    /posts/new(.:format)      posts#new
edit_post GET    /posts/:id/edit(.:format) posts#edit
     post GET    /posts/:id(.:format)      posts#show
          PATCH  /posts/:id(.:format)      posts#update
          PUT    /posts/:id(.:format)      posts#update
          DELETE /posts/:id(.:format)      posts#destroy
     root GET    /                         welcome#index
```

따라서, 위와 같이 브라우저에서 `http://localhost:3000/posts`와 같이 요청하게 되면 디폴트로 HTTP **GET 메소드**에 매칭되는 URI 패턴을 찾아보게 된다. 결과적으로 `posts` 컨트롤러를 호출하여 `index` 액션을 실행하게 되고 최종적으로 `app/views/posts/` 디렉토리 상의  `index.html.erb` 뷰 템플릿을 렌더링하여 응답으로 보내게 된다.

브라우저 상에서 데이터를 추가, 삭제, 변경해서 문제없이 잘 수행되는 것을 확인하자.

지금까지 **scaffold 제너레이터**를 이용하여 특별한 추가 코딩없이 최소한의 기능을 가진 게시물 작성 모듈을 작성할 수 있게 되었는데, 처음 레일스를 접하는 개발자들에게는 놀라운 일이다. 레일스는 이와 같이 기본적인 기능을 구현하기 위해서 개발자들이 추가해야 하는 코드를 대신해서 작성해 준다.



---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_03
