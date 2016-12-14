# 모델간의 관계선언

`posts` 테이블의 글을 게시판별로 분류하기 위해서는 `Bulletin` 모델과 `Post` 모델 사이에 일대다의 관계선언을 해 줄 필요가 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-13_20-47-09_zpsujhmnhu1.png)

`app/models/bulletin.rb` 파일을 열고 아래와 같이 추가한다.

```ruby
class Bulletin < ActiveRecord::Base
  has_many :posts, dependent: :destroy
end
```

`app/models/post.rb` 파일을 열고 아래와 같이 추가한다.

```ruby
class Post < ActiveRecord::Base
  belongs_to :bulletin
end
```

> #### Note::노트
> 
> 두 모델 간의 관계선언을 할 때는 단복수에 주의해야 한다. 위에서와같이 `has_many` 다음에는 항상 복수형(`posts`)을 지정해야 하고, `belongs_to` 다음에는 항상 단수형(`bulletin`)으로 지정해야 한다.

관계선언을 하는 [매크로 스타일의 클래스 메소드](http://stackoverflow.com/a/926655)인 `has_many`와 `belongs_to`는 매우 직관적이어서 별도의 설명이 필요치 않다. 단, `has_many`의 경우 `dependent` 옵션을 지정할 수 있는데, `:destroy`로 지정할 경우, 특정 `bulletin` 레코드를 삭제할 때 이 게시판에 속하는 모든 `posts`도 동시에 삭제된다.

> #### Note::노트
> 
> 단, 이와 같이 두 모델의 관계를 선언하는 것만으로 실제 DB 테이블이 자동으로 연결되지 않는다. 즉, 관계형 데이터베이스에서 두 테이블이 관계를 가지기 위해서는 자식 테이블 필드 중에 부모 테이블의 `id`를 외래키(foreign key)로 가지고 있어야 한다.

액티브레코드를 통해서 이 두 모델이  연결되는 각각의 테이블을 물리적으로 연결하기 위해서는 `posts` 테이블에 `bulletin_id`(모델명 + 'id')란 정수형(integer형)의 필드를 추가해 주어야 한다.

> #### Note::노트
> 
> 레일스의 모든 모델 클래스는 해당 테이블의 `primary key`가 `id`인 것으로 가정한다. 이것은 레일스의 규칙이다. 물론 `id` 이외의 다른 키를 `primary key`로 사용할 수 있지만, 부가적인 작업을 더 해주어야 하기 때문에 불편하다. 따라서 레일스 나라에서 살려면 레일스의 법을 준수할 필요가 있는 것이다.

따라서 `posts` 테이블에 `bulletin_id` 필드를 추가하기 위해 마이그레이션 파일을 작성한다.

```bash
$ bin/rails g migration add_bulletin_id_to_posts bulletin:references
Running via Spring preloader in process 34806
      invoke  active_record
      create    db/migrate/20161213115533_add_bulletin_id_to_posts.rb
```

생성된 마이그레이션 파일(`db/migrate/(생성된 일자가 포함된 일련의 숫자)_add_bulletin_id_to_posts.rb`)은 아래와 같다.

```ruby
class AddBulletinIdToPosts < ActiveRecord::Migration[5.0]
  def change
    add_reference :posts, :bulletin, foreign_key: true
  end
end
```

이 마이그레이션 파일에 대해서 `db:migrate` 작업을 한다.

```bash
$ bin/rails db:migrate
Running via Spring preloader in process 35094
== 20161213115533 AddBulletinIdToPosts: migrating =============================
-- add_reference(:posts, :bulletin, {:foreign_key=>true})
   -> 0.0371s
== 20161213115533 AddBulletinIdToPosts: migrated (0.0372s) ====================
```

`db/schema.rb` 파일에서 `posts` 테이블 부분을 보면 `bulletin_id` 속성이 추가되고 인덱스 파일까지 생성된 것을 확인할 수 있다.  

```sql
create_table "posts", force: :cascade do |t|
  t.string   "title",                    comment: "글 제목"
  t.text     "content",                  comment: "글 내용"
  t.datetime "created_at",  null: false
  t.datetime "updated_at",  null: false
  t.integer  "bulletin_id"
  t.index ["bulletin_id"], name: "index_posts_on_bulletin_id", using: :btree
end
```

### 레일스 콘솔에서 확인

지금까지 작업한 것이 제대로 동작하는지를 확인하기 위해서 터미널에서 아래와 같이 레일스 콘솔을 실행해 보자.

```bash
$ bin/rails console
Running via Spring preloader in process 31958
Loading development environment (Rails 5.0.0.1)
>> bulletin = Bulletin.new
=> #<Bulletin id: nil, title: nil, description: nil, created_at: nil, updated_at: nil>
```

그리고 아래와 같이 `bulletin.post`까지 입력한 후 `<tab>` 키를 두번 눌러보면 사용할 수 있는 메소드들을 볼 수 있다.

```bash
>> bulletin.post
bulletin.post_ids   bulletin.post_ids=  bulletin.posts      bulletin.posts=
```

이 메소드들은 두 모델 간의 관계선언을 할 때 자동으로 만들어진다.

* `bulletin.post_ids` : 이 메소드는 특정 `bulletin` 객체에 속하는 모든 `post` 객체들의 `id` 값들을 배열로 반환한다. 현재는 빈배열을 반환할 것이다.
* `bulltein.post_ids=` : 이 메소드는 `post` 객체들의 `id` 값들을 요소로하는 배열을 할당해 주어, 해당 `post` 객체들이 이 `bulletin` 객체의 자식 객체들로 등록되도록 한다.
* `bulletin.posts` : 이 메소드는 특정 `bulletin` 객체에 속하는 모든 `post` 객체들을 배열로 반환한다. 현재는 빈배열(`#<ActiveRecord::Associations::CollectionProxy []>`)을 반환할 것이다.
* `bulletin.posts=` : 이 메소드는 `post` 객체들를 요소로하는 배열를 할당해 주어, 해당 `post` 객체들이 이 `bulletin` 객체의 자식 객체들로 등록되도록 한다.

이상으로 두 모델의 관계선언을 완성하였다.

### 관계선언의 잇점

특정 게시판에 하나의 글을 추가한다고 가정해 보자. 우선, 레일스 콘솔을 열고 앞에서 생성했던 `공지사항`이라는 `bulletin` 객체를 불러 온다. 

```bash
$ bin/rails console
Running via Spring preloader in process 32673
Loading development environment (Rails 5.0.0.1)
>> bulletin = Bulletin.first
  Bulletin Load (0.3ms)  SELECT  "bulletins".* FROM "bulletins" ORDER BY "bulletins"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> #<Bulletin id: 1, title: "공지사항", description: "공지사항을 입력하는 게시판입니다.", created_at: "2016-12-13 10:43:57", updated_at: "2016-12-13 10:43:57">
```

그리고 `post` 객체도 하나 생성하자. 공지사항 게시판에서 글을 생성할 것이기 때문에 아래와 같이 `bulletin.posts`에 대해서 `create` 메소드를 호출하고 각 속성에 값을 할당하여 넘겨 준다. 

```bash
>> post = bulletin.posts.create title:"레일스 가이드라인 책 집필", content:"초보자를 위한 레일스"
   (0.3ms)  BEGIN
  SQL (24.3ms)  INSERT INTO "posts" ("title", "content", "created_at", "updated_at", "bulletin_id") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["title", "레일스 가이드라인 책 집필"], ["content", "초보자를 위한 레일스"], ["created_at", 2016-12-14 07:48:52 UTC], ["updated_at", 2016-12-14 07:48:52 UTC], ["bulletin_id", 1]]
   (1.0ms)  COMMIT
=> #<Post id: 2, title: "레일스 가이드라인 책 집필", content: "초보자를 위한 레일스", created_at: "2016-12-14 07:48:52", updated_at: "2016-12-14 07:48:52", bulletin_id: 1>
```

현재는 `post.bulletin_id` 값이 `1`이기 때문에,
`post` 객체와 `bulletin` 객체가 물리적으로 연결되어 있지만, 이 값을 `nil`로 지정하면 두 객체의 관계는 없어지게 된다. 

```ruby
>> post.bulletin_id = nil
=> nil
>> post.save
   (0.3ms)  BEGIN
  SQL (0.5ms)  UPDATE "posts" SET "updated_at" = $1, "bulletin_id" = $2 WHERE "posts"."id" = $3  [["updated_at", 2016-12-14 07:54:56 UTC], ["bulletin_id", nil], ["id", 2]]
   (1.1ms)  COMMIT
=> true
```

이제 `post.bulletin`와 같이 호출하면 여전히 `bulletin` 객체와 연결되어 있는 것을 알 수 있다. `nil`이 나와야 할 것 같은데, 이상하다. 이유는 `post` 객체의 `bulletin` 캐쉬값은 이전 값이 그대로 남아 있기 때문이다. 

```ruby
>> post.bulletin
=> #<Bulletin id: 1, title: "공지사항", description: "공지사항을 입력하는 게시판입니다.", created_at: "2016-12-13 10:43:57", updated_at: "2016-12-13 10:43:57">
```

따라서, 이와 같이 `reload` 메소드를 호출한 후 `bulletin`을 다시 불러오면 제대로 `nil` 값이 나타나는 것을 확인할 수 있다. 

```ruby
>> post.reload.bulletin
  Post Load (0.3ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
=> nil
```

마찬가지로 `bulletin.posts`를 호출하면,

```ruby
>> bulletin.posts
  Post Load (9.1ms)  SELECT "posts".* FROM "posts" WHERE "posts"."bulletin_id" = $1  [["bulletin_id", 1]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Post id: 2, title: "레일스 가이드라인 책 집필", content: "초보자를 위한 레일스", created_at: "2016-12-14 07:48:52", updated_at: "2016-12-14 07:54:56", bulletin_id: nil>]>
```

`bulletin` 객체를 다시 로딩한 후 호출하면,

```ruby
>> bulletin.reload.posts
  Bulletin Load (3.0ms)  SELECT  "bulletins".* FROM "bulletins" WHERE "bulletins"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Post Load (0.3ms)  SELECT "posts".* FROM "posts" WHERE "posts"."bulletin_id" = $1  [["bulletin_id", 1]]
=> #<ActiveRecord::Associations::CollectionProxy []>
```

제대로 값이 보이게 된다. 


이제 아래와 같이 두 모델의 관계선언이 제대로 설정되었는지를 확인해 보자.

```bash
irb(main):010:0> bulletin.posts
  Post Load (0.2ms)  SELECT "posts".* FROM "posts" WHERE "posts"."bulletin_id" = ?  [["bulletin_id", 1]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Post id: 2, title: "레일스 가이드라인 책 집필", content: "초보자를 위한 레일스", created_at: "2015-02-01 06:42:13", updated_at: "2015-02-01 06:42:56", bulletin_id: 1>]>
`

지금까지 `bulletin` 객체에 임의의 `post` 객체를 추가하는 과정을 보았다.

즉, `bulletin.posts.create`와 같이 `post`를 생성하면 생성되는 `post` 객체의 `bulletin_id` 속성이 `bulletin.id` 값으로 자동으로 지정되기 때문에, `post.bulletin_id=`에 `bulletin.id` 값을 할당하는 별도의 과정이 필요 없게 된다.


---
> **Git소스** https://github.com/rorlab/rcafe2/tree/chapter_05_07
