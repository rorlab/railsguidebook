# Post 모델 CRUD 살펴보기

`scaffold` 제너레이터를 이용하여 특정 모델 리소스를 생성하면 기본적으로 `index`, `show`, `new`, `edit`, `create`, `update`, `destroy` 등 7개의 컨트롤러 액션이 생성된다.

> **Info** 제너레이터의 종류를 보고 싶은 경우에는 콘솔에서 bin/rails generate 명령을 실행하면 된다. 결과에서 Rails: scaffold 를 찾아 볼 수 있을 것이다.

데이터의 생성, 읽기, 업데이트, 삭제가 위의 5개의 액션(`create`, `index`, `show`, `update`, `destroy`)으로 구현된다.

| 모델작업 | 액션 |
| -- | -- |
| **C**(reate) | create |
| **R**(ead) | index, show |
| **U**(pdate) | update |
| **D**(elete) | destroy |


### 기본 액션들

7개의 액션 중 나머지 두개, `new`와 `edit` 액션은 데이터 조작을 하지 않고 단지 뷰를 렌더링하는 기능만을 가진다.

#### create 액션

특정 모델의 한 객체를 생성하여 DB 테이블로 저장한다.
액션 종료시 `show` 액션으로 리디렉트된다.

#### index 액션

DB 쿼리후, 특정 모델(들)의 모든 객체를 불러와 보여 준다.

```html
<h2>Listing posts</h2>

<table class="table">
  <thead>
    <tr>
      <th>Title</th>
      <th>Data actions</th>
    </tr>
  </thead>

  <tbody>
    <% @posts.each do |post| %>
      <tr>
        <td><%= post.title %></td>
        <td>
          <%= link_to 'Show', post, class: 'btn btn-default' %>
          <%= link_to 'Edit', edit_post_path(post), class: 'btn btn-default' %>
          <%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' }, class: 'btn btn-default' %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>


<%= link_to 'New Post', new_post_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-05-03_12-17-22_zpsd8874ad6.png)

#### show 액션

DB 쿼리후, 특정 모델의 특정 객체만을 불러와 보여 준다.

```html
<h2>Post Preview</h2>

<p>
  <strong>Title:</strong>
  <%= @post.title %>
</p>

<p>
  <strong>Content:</strong>
  <%= @post.content %>
</p>

<hr>

<%= link_to 'Edit', edit_post_path(@post), class: 'btn btn-default' %>
<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-05-03_12-19-12_zps1a56f407.png)

#### update 액션

DB 쿼리후, 특정 모델의 속성을 변경한 후 DB 테이블로 저장한다.
액션 종료시 `show` 액션으로 리디렉트된다.

#### destroy 액션

DB 쿼리후, 특정 모델의 특정 객체(들)를 삭제한다.
액선 종류시 `index` 액션으로 리디렉트된다.

#### form 파셜 템플릿 파일

`new`와 `edit` 뷰 템플릿 파일에서 사용하는 `form` 파셜 템플릿 파일을 아래와 같이 수정한다.

```html
<%= simple_form_for(@post) do |f| %>
  <%= f.error_notification %>

  <div class="form-group">
    <%= f.input :title,input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :content,input_html: { class: 'form-control', rows: 10 } %>
  </div>

  <%= f.button :submit, class: 'btn btn-default' %>

<% end %>
```

#### new 액션

새로운 데이터를 입력 받는 폼을 응답으로 보낸다.

```html
<h2>New post</h2>

<%= render 'form' %>

<hr>

<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-05-03_12-20-43_zpse826f549.png)

#### edit 액션

기존 데이터를 수정하기 위한 폼을 응답으로 보낸다.

```html
<h2>Editing post</h2>

<%= render 'form' %>

<hr>

<%= link_to 'Show', @post, class: 'btn btn-default' %>
<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/2014-05-03_12-21-51_zps869f6832.png)


### posts 컨트롤러

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  # GET /posts
  # GET /posts.json
  def index
    @posts = Post.all
  end

  # GET /posts/1
  # GET /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

  # GET /posts/1/edit
  def edit
  end

  # POST /posts
  # POST /posts.json
  def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /posts/1
  # PATCH/PUT /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to @post, notice: 'Post was successfully updated.' }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /posts/1
  # DELETE /posts/1.json
  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to posts_url }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def post_params
      params.require(:post).permit(:title, :content)
    end
end
```

### before_action

레일스 4부터 `_filter`가 `_action`으로 변경되었다.

따라서 `before_filter`, `after_filter`, `around_filter`는 각각 `before_action`, `after_action`, `around_action`으로 변경되었다.

컨트롤러의 상단에서 아래와 같은 `before_action` 필터를 볼 수 있다.

```ruby
before_action :set_post, only: [:show, :edit, :update, :destroy]
```

이것은 `posts` 컨트롤러의 액션 중에서 `show`, `edit`,`update`, `destroy` 액션이 실행되기 전에 반드시 `set_post` 메소드를 실행하라는 필터인 것이다. 이와 같은 필터 메소드는 해당 컨트롤러에서 `private`으로 선언되어 있다.

```ruby
private
  def set_post
    @post = Post.find(params[:id])
  end
```

즉, 파라미터로 넘겨 받은 `id` 값을 이용하여 특정 post를 조회한 후 `@post` 인스턴스 변수에 할당한다.

### Strong Parameters

레일스 3에서는 mass assignment에 대한 화이트리스트를 해당 모델 클래스에서 `attr_accesible` 매크로로 작성하였다.

```
class User < ActiveRecord::Base
  attr_accessible :first, :last, :email
end
```

즉, `User` 모델의 `first`, `last`, `email` 속성만을 [mass assignment](http://code.tutsplus.com/tutorials/mass-assignment-rails-and-you--net-31695)로 저장할 수 있다는 것이다.

그러나 레일스 4로 업그레이드되면서 이러한 속성 보안관련 기능이 모델로부터 컨트롤러로 이동하여 [Strong Parameters](http://richonrails.com/articles/rails-4-preview-strong-parameters)의 개념으로 재구성되었다.

`posts` 컨트롤러 클래스 파일의 하단에는 아래와 같이 정의되어 있다.

```ruby
private
  def post_params
    params.require(:post).permit(:title, :content)
  end
```

즉, 파라미터로 넘겨 받은 속성 중에 `title`과 `content`만을 화이트리스트로 인정하겠다는 뜻이다. 따라서 다른 속성은 `save` 또는 `udpate` 되지 않게 된다.


---
> **Git소스** https://github.com/rorlab/rcafe/tree/제5.5장


