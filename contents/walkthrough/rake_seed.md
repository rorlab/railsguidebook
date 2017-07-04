# 기본 데이터 추가하기

`rcafe2` 프로젝트는 4개의 메뉴를 가지게 되었다.

* 공지사항
* 새소식
* 가입인사
* 갤러리

처음 이 프로젝트를 실행하면 게시판이 없기 때문에 메뉴 링크시 에러가 발생한다. 따라서 이를 위해서 기본 게시판을 미리 생성해야 할 필요가 있다.

레일스에서는 이를 위해서 데이터 `db:seed`라는 `rails` 작업을 할 수 있다.

```bash
$ bin/rake db:seed
```

이와 같은 명령은 `db/seeds.rb` 파일에 있는 명령을 실행하여 데이터를 생성하게 된다.

이제, 여기서는 4개의 게시판을 생성하는 명령을 추가하도록 한다.

```ruby
# 디폴트 게시판 생성
Bulletin.create! title: '공지사항'
Bulletin.create! title: '새소식'
Bulletin.create! title: '가입인사', post_type: :blog
Bulletin.create! title: '갤러리', post_type: :gallery
```

`공지사항`과 `새소식`은 `post_type` 속성을 지정하지 않았는데 이것은 `post_type` 속성의 디폴트 값이 `:bulletin`이기 때문에 생략해도 테이블에 저장될 때는 `:bulletin` 값이 할당된다.

이를 테스트 하기 위해서 현재 테이블에 저장된 모든 데이터를 초기화 한다.

```bash
$ bin/rake db:reset
```

`as_enum`이라는 젬을 사용하기 때문에 실제 값은 아래와 같이 보이게 될 것이다. 

![](/assets/2016-12-18_12-31-17_zpshhykpu4g.png)

아래의 그림과 같이 `db:reset`은 DB를 삭제한 후 다시 `db:setup`하는 작업을 하는데, 이 때 `db:seed` 작업도 함께 수행된다.

![](/assets/2016-12-18_12-47-35_zpszfnsbrsz.png)

이제 브라우저에서 확인하여 메뉴항목을 클릭하면 해당 게시판이 에러 없이 보이게 된다.

> #### Note::노트
> 
> `rcafe2` 프로젝트에서는 데이터베이스로 [`postgresql`](https://www.postgresql.org/)를 사용한다. 그러나 레일스의 디폴트 데이터베이스는 [`sqlite`](http://www.sqlite.org)다. 이 데이터베이스는 서버 설정이 필요없는 `serverless` 데이터베이스로 트랜잭션이 가능한 관계형데이터베이스(RDBMS)이다. 따라서 데이터베이스를 생성하는 과정이 필요없지만, `PostgreSQL`이나 `MySQL`과 같은 기타 다른 데이터베이스의 경우는 처음에 `db:create` 명령으로 데이터베이스 생성 작업을 해 주어야 한다.

---
> **Git소스** https://github.com/rorlab/rcafe2/tree/chapter_05_13
