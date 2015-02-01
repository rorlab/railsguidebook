# 태그 달기

각 `post`에 태그를 붙여 보자. [ruby-toolbox.com](https://www.ruby-toolbox.com/categories/rails_tagging)에서 태그관련 젬을 검색해 보면 단연코 [`acts-as-taggable-on`](https://github.com/mbleigh/acts-as-taggable-on)이라는 젬의 사용빈도가 가장 높다.

`Gemfile` 열고 아래와 같이 젬을 추가하고,

```ruby
gem 'acts-as-taggable-on'
```

번들 인스톨한 후,

```bash
$ bin/bundle install
```

로컬 웹서버를 다시 부팅한다.

다음은 이 젬의 작동을 위해 몇가지 마이그레이션 파일을 아래와 같이 생성한다.

```bash
$ bin/rake acts_as_taggable_on_engine:install:migrations
```

그리고 마이그레이션 작업을 한다.

```bash
$ bin/rake db:migrate
```

이제 태그를 붙이고자 하는 `Post` 모델 클래스 파일(`app/models/post.rb`)을 열고 아래와 같이 코드를 추가한다.

```ruby
class Post < ActiveRecord::Base
  acts_as_taggable
  ...
end
```

이로써 `Post` 모델에 대해서 `tag_list` 속성 메소드를 사용할 수 있게 된다. 이 속성을 폼 데이터로 입력받기 위해서는 `strong parameter`로 등록해 주어야 한다. 이를 위해서  `posts_controller.rb` 파일(`app/controllers/posts_controller.rb`)을 열고 아래와 같이 `post_params` 메소드에 `:tag_list` 속성을 추가한다.

```ruby
def post_params
  params.require(:post).permit(:title, :content, :picture, :picture_cache, :tag_list)
end
```

실제로 `:tag_list` 속성을 폼으로부터 입력받기 위해서 `post` 폼 `partial` 템플릿 파일(`app/views/posts/_form.html.erb`)을 열고 아래와 같이 추가한다.

```html
<div class="form-group">
  <%= f.input :tag_list, placeholder: '하나 이상의 태그는 콤마(,)로 구분하여 입력하세요.', input_html: { class: 'form-control' } %>
</div>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-07-06_07-05-59_zpse0848285.png)

`1`번과 같이 태그는 한글도 가능하고, 태그 사이에 공백도 가능하다.

이와 같이 입력된 태그를 보여 주기위해 `posts#show` 액션 뷰 템플릿 파일(`app/views/posts/show.html.erb`)을 열고 아래와 같이 추가한다.

```html
...
<tr>
  <th>Tag list</th>
  <td><%= tag_icons @post.tag_list %></td>
</tr>
...
```

여기에서 사용한 `tag_icons` 헬퍼 메소드는 어플리케이션 헬퍼 파일(`app/helpers/application_helper.rb`)에 아래와 같이 정의한다.

```ruby
def tag_icons(tag_list)
  tag_list.map do | tag |
    "<a href='/posts?tag=#{CGI::escape(tag)}' class='tag'>#{tag}</a>"
  end.join(', ').html_safe
end
```

여기서 사용한 `CGI::escape()` 메소드는 태그에서 사용할 수 있는 특수문자를 이스케이핑하기 위한 것이다.

`posts#show` 액션 뷰 템프릿 파일에 갤러리 게시판의 경우 업로드된 이미지를 보여 줄 필요가 있다. 이를 위해서 `@post.bulletin.post_type`이 `gallery`일 경우 아래와 같이 추가해 준다.

```html
...
<% if @post.bulletin.post_type == "gallery" %>
  <tr>
    <th>Picutre</th>
    <td><%= image_tag @post.picture_url %></td>
  </tr>
<% end %>
</table>
...
```

이제는 `posts#index` 액션 뷰 템플릿 파일에서 태그를 표시하도록 하자. 이 때는 여건상 블로그형과 갤러리형 게시판에서만 태그를 표시하도록 하자.

우선 `app/views/posts/post_types/_blog.html.erb` 파일을 열고 아래와 같이 추가한다.

```html
<% if post.tag_list.size > 0 %>
  <div class='tag_list'><%= icon('tags') + " " + tag_icons(post.tag_list) %></div>
<% end %>
```

`app/views/posts/post_types/_gallery.html.erb` 파일에도 적당한 위치에 동일한 내용을 추가한다.

태그 존재할 경우만 보여주기 위해 `if` 조건문을 사용하였다. `tag_list` CSS 클래스를 정의해 주기 위해서 `app/assets/stylesheets/posts.css.scss` 파일을 열고 아래와 같이 추가해 준다.

```css
...
.tag_list {
  margin-bottom: .5em;
}
...
```

또한 이 CSS 파일에, `tag_icons()` 헬퍼 메소드에서 생성하는 태그에 테두리를 추가하기 위해서, 스타일을 아래와 같이 추가한다.

```css
...
a.tag {
  border:1px solid #eaeaea;
  border-radius:3px;
  background-color: #f4f4f4;
  padding:.1em .2em;
  &:hover {
    text-decoration: none;
    color: white;
    background-color: #265484;
    border:1px solid #265484;
  }
}
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-07-06_19-24-01_zpsa0ff25fc.png)

각 태그에는 해당 태그로 검색할 수 있도록  `<a>` 링크 태그의 `href` 속성으로 `/posts?tag=...`와 같이 지정했다. 즉, `posts#index` 액션을 호출시에 `:bulletin_id` 파라미터 없이 `:tag` 파라미터만 넘겨 주게 된다. 따라서 모든 `posts` 객체에 대해서 해당 태그를 가진 것들을 쿼리하게 된다.

이를 위해서는 `posts_controller.rb` 파일(`app/controllers/posts_controller.rb`)에서 `index`, `set_bulletin`, `set_post` 메소드를 아래와 같이 수정해야 한다.

```ruby
...
def index
  if params[:bulletin_id]
    @posts = @bulletin.posts.all
  else
    if params[:tag]
      @posts = Post.tagged_with(params[:tag])
    else
      @posts = Post.all
    end
  end
end
...
private
  def set_bulletin
    @bulletin = Bulletin.friendly.find(params[:bulletin_id]) if params[:bulletin_id]
  end

  def set_post
    if params[:bulletin_id]
      @post = @bulletin.posts.find(params[:id])
    else
      @post = Post.find(params[:id])
    end
  end
...
```

`index` 액션에서 사용한 `Post.tagged_with()` 클래스메소드는 `acts-as-taggable-on` 젬이 지원한 메소드로 임의의 태그를 넘겨 주면 해당 태그를 포함하는 `post` 객체들을 반환해 준다.

이제 태그로 검색한 결과를 보여 주기 위해서 `posts#index` 액션 뷰 템플릿 파일(`app/views/posts/index.html.erb`)을 수정해야 한다.

```html
<% if params["bulletin_id"] %>
  <%= render "posts/post_types/#{@bulletin.post_type}" %>
<% else %>
  <% if params[:tag] %>
  <h2>Posts tagged with "<%= params[:tag] %>" <small>( <%= @posts.size %> )</small></h2>
  <% else %>
  <h2>All Posts <small>( <%= @posts.size %> )</small></h2>
  <% end %>
  <ul id='posts_tagged'>
    <%= render @posts %>
  </ul>
<% end %>
```

`<%= render @posts %>`에서 사용할 `_post.html.erb` 파일을 `app/views/posts/` 디렉토리에 생성하고 아래와 같이 추가한다.

```html
<li>
  <span class='label label-default'><%= post.try(:bulletin).try(:title) %></span>
  <%= link_to post.title, [post.bulletin, post] %>
  <%= time_ago_in_words(post.created_at) %> ago
  <%= icon('tags') + ' ' + tag_icons(post.tag_list) %>
</li>
```

`app/assets/stylesheets/posts.css.scss` 파일을 열고 아래의 내용을 추가한다.

```css
...
ul#posts_tagged {
  margin-top:2em;
  li {
    margin-bottom: .5em;
  }
}
```


![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-07-06_19-50-46_zps98e2d69c.png)

이상으로 태그 달기를 마치도록 하겠다.

---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_15

