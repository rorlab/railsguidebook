# 액티브레코드란?

액티브레코드(`ActiveRecord`)란, 관계형데이터베이스(`RDBMS`)의 테이블을 객체로 연결(`ORM : Object Relational Mapping`)해서 네이티브 데이터베이스 SQL을 사용하지 않고도 데이터를 조작할 수 있도록 다양한 메소드를 제공해 준다.

`MVC` 디자인 패턴으로 개발하는 레일스에서 액티브레코드는 `M(Model)`에 해당한다. 이 책의 [프로젝트 따라하기](../walkthrough/start.html)에서 사용하는 `Post` 모델 클래스의 소스를 보면 아래와 같다.

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :bulletin
  mount_uploader :picture, PictureUploader
end
```

위의 소스코드에서 `Post` 모델 클래스는 `ActiveRecord` 모듈의 `Base` 클래스([`ActiveRecord::Base`](http://api.rubyonrails.org/files/activerecord/README_rdoc.html))로부터 상속을 받는다. 이 말은, `ORM`을 통해서 데이터베이스 테이블을 쉽게 조작할 수 있도록, `Base` 클래스에  정의된 메소드를 `Post` 모델에서도 모두 그대로 사용할 수 있다는 의미이다.

`Post` 모델 클래스가 정의되고 실제로 데이터베이스에 `posts` 테이블이 존재하면 액티브레코드의 다양한 메소드를 바로 사용해서 데이터를 조작할 수 있게 되는 것이다.

즉, `Post` 모델의 빈 객체를 생성하기 위해서는 아래와 같이 코드를 작성할 수 있다.

```ruby
@post = Post.new
```

레일스 콘솔을 통해서 확인하면 아래와 같다.

```bash
$ bin/rails console
Loading development environment (Rails 4.1.1)
irb(main):001:0> post = Post.new
=> #<Post id: nil, title: nil, content: nil, created_at: nil, updated_at: nil, bulletin_id: nil, picture: nil>
```

출력 결과에서 알 수 있듯이, `post` 변수는 `Post` 모델 클래스의 `new`라는 클래스 메소드를 이용하여 데이터가 없는 빈 인스턴스 객체를 할당받게 된다. `=>` 다음에 보이는 반환값은 `post` 객체에 포함되는 속성 목록을 보여주고 있다. 이러한 각 속성들의 값은 객체지향적 프로그래밍의 기본적인 문법을 이용하여 쉽게 얻을 수 있다.

```ruby
irb(main):002:0> post.title
=> nil
```









