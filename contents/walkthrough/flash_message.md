# flash 메시지 표시하기

레일스에서는 컨트롤러의 액션 실행시 발생하는 [액티브레코드](/appendices/active_record.html) 관련 각종 메시지를 `flash`라는 세션의 특수한 형태를 통해서 표시할 수 있다. 카메라의 플래시를 연상해 보면 알 수 있듯이 메시지를 짧은 시간만 저장할 수 있으며 레일스 내부적으로는 `FlashHash` 클래스의 인스턴스이다. 즉, 액션간에 임시로 객체를 전달할 수 있는 수단으로 생각할 수 있다. 따라서 `flash` 객체에 어떤 것이라도 지정할 수 있고 바로 다음번 액션에서만 사용되고 사라지게 되는 것이다.

레일스에서는 대게 이러한 `flash` 메시지를 어플리케이션 레이아웃의 `body` 태그내 최상단에 위치시킨다.

```html
<!DOCTYPE html>
<html>
<head>
</head>
<body>

  <= 이 곳에 플래시 메시지를 삽입한다. <<<<<

  <%= yield %>

</body>
</html>
```

이제 `Bootstrap 3`의 CSS 클래스를 이용해서 이러한 `flash` 메시지를 실제로 표시해 보자.

레일스의 `scaffold` 제너레이터를 이용하여 생성한 `Post`모델의 컨트롤러(posts_controller.rb) 클래스 파일을 살펴보면, `create`, `update`, `destroy` 액션에서 `flash`를 사용하는 것을 확인할 수 있다.

```ruby
redirect_to ..., notice: 'Post was successfully created.'
```

`flash` 메시지를 할당할 때 `flash[:notice] = 'some message'`와 같이 코드를 작성할 수 있지만, 위에서와 같이 `redirect_to` 메소드를 사용할 때는 해시형태의 옵션으로 `flash` 메시지를 지정할 수도 있다.

> **Note** 루비에서 해시 키/값을 표현할 때 :notice => "some message"와 같이 작성할 수도 있지만 루비 1.9부터는 notice: "some message"와 같이 축약형으로도 지정할 수 있다.

위에서 언급한 바와 같이 이 메시지는 어플리케이션 레이아웃 템플릿(`application.html.erb`)에 `partial`로 삽입하기로 한다.


```html
<div class="container">

  <%= render partial: "shared/flash_messages", flash: flash %>

  <%= content_for?(:post_layout) ? content_for(:post_layout) : yield %>

</div>
```

그리고 `app/views/shared` 디렉토리에 `_flash_messages.html.erb`라는 `partial` 템플릿 파일을 추가하고 아래와 같이 작성한다.

```html
<% flash.each do |type, message| %>
  <div class="alert <%= bootstrap_class_for(type) %> alert-dismissable fade in">
    <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
    <%= message %>
  </div>
<% end %>
```

`render` 메소드는 `partial` 템플릿을 받을 때, 옵션으로 로컬 변수들을 넘겨 받을 수 있는데, 여기서는 `flash` 변수에 `flash` 해시를 넘겨 준다. 이 변수 `flash`는 `_flash_messages.html.erb` 파일에서 바로 사용할 수 있게 된다.

`flash` 해시는 `each` 메소드를 이용하여 각각의 요소를 블록으로 넘겨 주어 요소의 키는 `type`, 값은 `message` 블록변수로 받아 블록 내에서 사용할 수 있다. 이 때 `type` 변수 값에 따라 CSS 클래스를 달리 지정할 수 있도록 `bootstrap_class_for()`라는 어플리케이션 헬퍼 메소드를 `app/helpers/application_helper.rb` 파일에 추가하고 아래와 같이 작성한다. 여기서 사용하는 CSS 클래스는 [bootstrap 3](http://getbootstrap.com/components/#alerts)를 참고하여 지정한 것이다.

```ruby
module ApplicationHelper

  def bootstrap_class_for(flash_type)
    case flash_type
      when "success"
        "alert-success"   # 초록색
      when "error"
        "alert-danger"    # 빨간색
      when "alert"
        "alert-warning"   # 노랑색
      when "notice"
        "alert-info"      # 파랑색
      else
        flash_type.to_s
    end
  end

end
```

이제 새로운 게시물을 작성하거나 수정하면 페이지 상단의 메뉴 아래에 적절한 메시지가 표시될 것이다. 당연한 것이지만, 이 메시지를 페이지를 다시 로딩하면 사라질 것이다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/2014-05-18_16-17-06_zps5a67bce4.png)

지금까지 레일스의 `flash` 메시지를 표시하는 방법을 소개했다.


---
> **Git소스** https://github.com/rorlab/rcafe/tree/제5.13장




