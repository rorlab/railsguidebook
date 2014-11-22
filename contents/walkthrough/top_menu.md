# friendly_id 젬 사용하기

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2014-05-09_10-17-19_zps94af68c8.png)

`bulletin` 테이블에 위와 같이 3개의 게시판을 추가해 놓은 상태라고 가정하자.

우선 `공지사항` 게시판에 글을 작성하는 작업을 해 보기로 한다.

`http://localhost:3000/bulletins/공지사항`과 같이 `id` 대신에 게시판 이름으로 해당 게시판을 불러오도록 해 보자.

이를 위해서 `Gemfile`에 `friendly_id` 젬을 추가하고,

```ruby
gem 'friendly_id', '~> 5.0.0'
```

설치한다.

```bash
$ bin/bundle install
```

그리고 `Bulletin` 모델 클래스 파일(`app/models/bulletin.rb`)을 아래와 같이 변경한다.

```ruby
class Bulletin < ActiveRecord::Base
  extend FriendlyId
  friendly_id :title

  has_many :posts, dependent: :destroy
end
```

여기서는 `id` 대신에 `title` 속성을 이용해서 게시판을 불러오기 우해서 `friendly_id :title`와 같이 설정한다.

추가로, `app/controllers/bulletins_controller.rb` 파일에서 아래와 같이 수정한다. (`friendly`를 추가한다)

```ruby
private
    def set_bulletin
      @bulletin = Bulletin.friendly.find(params[:id])
    end
```

이제, 로컬 서버를 다시 시작하면, 아래의 브라우저 화면에서 `http://localhost:3000/bulletins/공지사항`을 확인할 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2014-05-09_10-34-25_zps25373419.png)



---
> **Git소스** https://github.com/rorlab/rcafe/tree/제5.8장


