# 게시판 레이아웃 작성하기

게시판의 형태를 세가지로 분류해 보자.

| 형태 | 설명 |
|------|------|
|일반형(bulletin)|일반적인 게시판의 형태. 디폴트 형태|
|블로그형(blog)|블로그처럼 게시물의 내용이 일렬로 보이도록 하는 형태|
|갤러리형(gallery)|이미지 갤러리처럼 한 줄에 여러개의 썸네일 이미지가 보이도록 하는 형태|

### Bulletin 모델에 post_type 속성 추가하기

게시판을 새로 추가할 때, 레이아웃를 지정하기 위해서 `post_type`이라는 속성을 추가하기로 한다. 이 속성은 `string` 속성을 가지는 것으로 한다.
이를 위해서 `Bulletin` 모델에 속성을 추가하는 마이그레이션 파일을 생성한다.

```bash
$ bin/rails g migration add_post_type_to_bulletins post_type
Running via Spring preloader in process 21023
      invoke  active_record
      create    db/migrate/20161216120650_add_post_type_to_bulletins.rb
```

그리고 `db:migrate` 작업을 한다.

```bash
$ bin/rake db:migrate
Running via Spring preloader in process 21600
== 20161216120650 AddPostTypeToBulletins: migrating ===========================
-- add_column(:bulletins, :post_type, :string)
   -> 0.0469s
== 20161216120650 AddPostTypeToBulletins: migrated (0.0469s) ==================
```

`Bulletin` 모델에서 이 속성을 열거형(`enum`)으로 선언하고  `:bulletin`, `:blog`, `:gallery` 심볼을 할당한다. 이를 위해서 [`as_enum`](https://github.com/lwe/simple_enum)이라는 젬을 설치한다. `Gemfile`에 추가하고 번들 인스톨하고 로컬 서버를 재시동한다. 

`Bulletin` 모델에서 아래와 같이 `post_type` 속성을 선언한다. 

```ruby
class Bulletin < ApplicationRecord
  has_many :posts, dependent: :destroy
  as_enum :post_type, bulletin: 0, blog: 1, gallery: 2
end
```

이제 `post_type_cd` 속성을 `integer` 데이터형으로 추가한다.

```
$ rails g migration post_type_cd_to_bulletins post_type_cd:integer
```

방금 생성된 마이그레이션 파일을 열고 디폴트 값을 `0`으로 설정한다. 

```
class AddPostTypeCdToBulletins < ActiveRecord::Migration[5.0]
  def change
    add_column :bulletins, :post_type_cd, :integer, default: 0
  end
end
```

> 참고 : [ActiveRecord::Enum](http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html)

```ruby
$ bin/rails c
Running via Spring preloader in process 28065
Loading development environment (Rails 5.0.0.1)
>> bulletin = Bulletin.new
=> #<Bulletin id: nil, title: nil, description: nil, created_at: nil, updated_at: nil, post_type: nil, post_type_cd: 0>
>> bulletin.post_type
=> :bulletin
>> bulletin.post_type_cd
=> 0
>> bulletin.gallery!
=> "gallery"
>> bulletin.post_type
=> :gallery
>> bulletin.post_type_cd
=> 2
>> bulletin.blog!
=> "blog"
>> bulletin.post_type
=> :blog
>> bulletin.post_type_cd
=> 1
>> bulletin.gallery?
=> false
>> bulletin.blog?
=> true
```


### Strong Parameter 추가하기

`bulletins_controller.rb` 파일을 열어 하단에 있는 `bulletin_params` 메소드에 아래와 같이 `post_type` 속성을 추가한다.

```ruby
def bulletin_params
  params.require(:bulletin).permit(:title, :description, :post_type)
end
```

### Bulletin 뷰 템플릿 파일의 변경

`app/views/bulletins/_form.html.erb` 파일을 열어 아래의 코드를 추가해 준다. `Bulletin` 형태를 게시판, 블로그, 또는 갤러리로 선택할 수 있게 해준다.

```ruby
<%= simple_form_for(@bulletin) do |f| %>
  <%= f.error_notification %>

  <div class="form-inputs">
    <%= f.input :title %>
    <%= f.input :description, input_html: { rows: 5 } %>
    <%= f.input :post_type, collection: enum_option_pairs(Bulletin, :post_type) %>
  </div>

  <div class="form-actions">
    <%= f.button :submit %>
  </div>
<% end %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_07-47-19_zpsbb9e3bbf.png)

`app/views/bulletins/show.html.erb` 파일을 열어 아래의 코드를 추가한다. 선택한 게시판의 형태를 표시할 것이다. 

```ruby
<tr>
  <th>Post Type</th>
  <td><%= @bulletin.post_type %></td>
</tr>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_07-50-56_zps22401824.png)

이렇게 해서 `Bulletin` 모델에서 추가할 작업이 완료되었다.

게시판의 형태에 따른 뷰를 보이도록 하기 위해서는 `app/views/posts/` 디렉토리에 `post_types`라는 하위 디렉토리를 만들고 이 디렉토리에 `_bulletin.html.erb` 파일과 `_blog.html.erb`, `_gallery_html.erb` 파일을 생성한다.

`posts` 컨트롤러의 `index` 액션 뷰 파일의 모든 내용을 `_bulletin.html.erb` 파일로 이동하고 상단의 부분을 아래와 같이 변경한다. 

```erb
<h2><%= bulletin_name params[:bulletin_id] %></h2>
```

> #### Info::안내
> 
> `bulletin_name` 헬퍼 메소드는 잠시 후에 설명할 것이다.  

그리고, `index.html.erb`는 아래와 같이 수정한다. `render` 메소드는 `partial` 템플릿 파일을 인수로 받아 렌더링 결과를 삽입해 준다. 루비에서는 이중 인용부호 내의 `#{expression}`을 표현식의 결과로 대체해 준다. 따라서 `@bulletin.post_type` 값이 `'bulletin'`일 경우 `"posts/post_types/bulletin"`로 평가되어 `app/views/posts/post_types/` 디렉토리의 `_bulletin.html.erb`이라는 `partial` 템플릿 파일을 `render` 메소드가 처리하게 된다.

```ruby
<%= render "posts/post_types/#{@bulletin.post_type}" %>
```

> #### Note::노트
> 
> `partial` 템플릿 파일에서는 부모 템플릿 파일에서 사용하는 모든 인스턴스 변수를 그대로 사용할 수 있다.

`post` 객체의 `bulletin_id` 속성값으로부터 게시판 이름을 얻기 위한 헬퍼 메소드를 작성한다. `app/helpers/posts_helper.rb` 파일을 열고 아래와 같이 추가한다. 

```ruby
module PostsHelper

  def bulletin_name(bulletin_id)
    Bulletin.find(bulletin_id).title
  end

end
```

`_blog.html.erb` 파일의 내용을 아래와 같이 작성한다. 테이블 형태로 게시물을 보여줬던 `index.html.erb`와 비교해보면 조금 달라졌다.

``` ruby
<h2><%= bulletin_name params[:bulletin_id] %></h2>

<% @posts.each do | post | %>
    <div class='post'>
      <div class='title'><%= post.title %></div>
      <div class='content'><%= simple_format post.content %></div>
      <div>
          <%= link_to 'Show', [post.bulletin, post], class:'btn btn-default' %>
          <%= link_to 'Edit', edit_bulletin_post_path(post.bulletin, post), class:'btn btn-default' %>
          <%= link_to 'Destroy', [post.bulletin, post], method: :delete, data: { confirm: 'Are you sure?' }, class:'btn btn-default' %>
      </div>
    </div>
<% end %>

<br>

<%= link_to 'New Post', new_bulletin_post_path, class: 'btn btn-default' %>
```

이제 `posts` 뷰 템플릿 파일들을 아래와 같이 수정한다.

### posts#new 뷰 템플릿 파일

"New post"라는 제목 대신 글을 올리는 게시판 이름이 나오도록 했다.

```ruby
<h2><%= bulletin_name params[:bulletin_id] %></h2>

<%= render 'form' %>

<hr>
<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```

### posts#edit 뷰 템플릿 파일

"Editing post"라는 제목 대신 글을 수정하는 게시판 이름이 나도오록 수정했다.

```ruby
<h2><%= bulletin_name params[:bulletin_id] %></h2>

<%= render 'form' %>

<hr>
<%= link_to 'Show', [@post.bulletin, @post], class: 'btn btn-default' %>
<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```

### posts#show 뷰 템플릿 파일

게시판 이름이 보이도록 수정했고 테이블 형태로 글을 보여준다.

``` ruby
<h2><%= bulletin_name params[:bulletin_id] %></h2>

<table class='table'>
<tr>
  <th>Title</th>
  <td><%= @post.title %></td>
</tr>
<tr>
  <th>Content</th>
  <td><%= @post.content %></td>
</tr>
<tr>
  <th>Created at</th>
  <td><%= @post.created_at %></td>
</tr>
</table>


<%= link_to 'Edit', edit_bulletin_post_path(@post.bulletin, @post), class: 'btn btn-default' %>
<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```

이제 브라우저에서 `http://localhost:3000/bulletins/3/edit`로 접속한 후 게시판의 종류를 `블로그`로 변경한 후,

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_08-04-11_zpsf31f98b6.png)


`CSS` 클래스 `post`, `title`, `content`를 `app/assets/stylesheets/posts.scss` 파일에 작성해 준다.

```css
.post {
  border: 1px solid $gray-light;
  border-radius: 5px;
  padding:1em;
  margin-bottom: 1em;
  .title {
    font-weight: bold;
    font-size: 1.5em;
    margin-bottom:.5em;
  }
}
```

그리고, `app/assets/stylesheets/` 디렉토리에 있는 `application.scss` 파일을 열고 아래와 같이 작성한다.(`@import 'posts';`을 추가했음)

```bash{10}
$light-orange: #ff8c00;
$navbar-default-color: $light-orange;
$navbar-default-bg: #312312;
$navbar-default-link-color: gray;
$navbar-default-link-active-color: $light-orange;
$navbar-default-link-hover-color: white;
$navbar-default-link-hover-bg: black;

@import 'bootstrap';
@import 'posts';  <<< 추가함

body { padding-top: 60px; }
```

이제 브라우저에서 상단 메뉴 중 `가입인사`를 클릭하고 가입인사를 테스트로 몇개 새로 추가하면 아래와 같이 변경되어 보이게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_08-47-08_zpsbd119099.png)




---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_10
