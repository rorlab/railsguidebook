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

`posts` 컨트롤러의 `index` 액션에서 DB 쿼리후, 특정 모델(들)의 모든 객체를 불러와 인스턴스 변수 `@posts`에 할당한다. `index` 액션의 뷰 템플릿 파일인 `index.html.erb(app/views/posts/index.html.erb)` 파일을 다음과 같이 ***bootstrap 형식에 맞게*** 수정한다. `<% @posts.each do | post | %>` 행의 `each` 블록에서 각 객체에 대한 정보를 블록 변수 `post`에 할당한 후 렌더링하게 된다.

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

브라우저에서 http://localhost:3000/posts 로 접속한 후 `New Post` 버튼을 클릭해서 글을 작성한 후 `index` 액션의 뷰 화면은 아래와 같다. 

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_17-25-44_zps5d737a01.png)

#### show 액션

DB 쿼리후, 특정 모델의 특정 객체만을 불러와 보여 준다. `posts` 컨트롤러의 `show` 액션 뷰 템블릿 파일인 `show.html.erb(app/views/posts/show.html.erb)` 내의 인스턴트 변수 `@post`에는 선택한 객체 정보가 할당된다. `posts` 컨트롤러를 확인해 보면 `show` 액션에는 아무 내용도 없지만 `private`으로 선언된 `set_post` 메소드에 의해 파라미터로 넘겨받은 `params[:id]`를 이용하여 `post` 객체를 인스턴스 변수 `@post`에 할당하게 된다. `show` 액션 뷰 템플릿 파일을 아래와 같이 수정한 후 브라우저로 확인한다. 

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

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_17-42-10_zpsb3249498.png)

#### update 액션

DB 쿼리후, 특정 모델의 속성을 변경한 후 DB 테이블로 저장한다.
액션 종료시 `show` 액션으로 리디렉트된다.

#### destroy 액션

DB 쿼리후, 특정 모델의 특정 객체(들)를 삭제한다.
액션 종료시 `index` 액션으로 리디렉트된다.


#### form 파셜 템플릿 파일

`new`와 `edit` 뷰 템플릿 파일에서 사용하는 `form` 파셜 템플릿 파일을 아래와 같이 수정하여 Content 열의 폭을 늘려 보기 좋게 변경했다.

```html
<%= simple_form_for(@post) do |f| %>
  <%= f.error_notification %>

  <div class="form-inputs">
    <%= f.input :title %>
    <%= f.input :content, input_html: { rows: 10 } %>
  </div>

  <div class="form-actions">
    <%= f.button :submit %>
  </div>
<% end %>
```

#### new 액션

새로운 데이터를 입력 받을 폼을 응답으로 보낸다. `new` 액션 뷰 템블릿 파일인 `new.html.erb`를 다음과 같이 수정한다. `<%= render 'form' %>`은 `_form.html.erb` 파셜 템플릿을 불러와 `render` 메소드로 삽입해 준다. 새로운 입력을 처리하는 뷰(new)와 자료 수정을 처리하는 뷰(edit) 양쪽에서 동일한 폼을 사용하기 때문에 코드 중복을 피하기 위해 파셜 템플릿이 사용된다. `new` 액션 뷰 템플릿 파일을 아래와 같이 수정하고 브라우저에서 확인한다. 

```html
<h2>New post</h2>

<%= render 'form' %>

<hr>

<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_17-50-52_zps54d3a9dd.png)

#### edit 액션

기존 데이터를 수정하기 위한 폼을 응답으로 보낸다. `edit` 액션 뷰 템플릿 파일 `edit.html.erb`를 다음과 같이 수정한다.

```html
<h2>Editing post</h2>

<%= render 'form' %>

<hr>

<%= link_to 'Show', @post, class: 'btn btn-default' %>
<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_17-52-53_zpsbe607ecf.png)


### posts 컨트롤러

레일스의 `scaffold` 제너레이터에 의해 자동으로 생성된 `posts` 컨트롤러는 다음과 같다. 앞서 언급한대로 각 액션을 처리하는 뷰 템플릿와 연결해서 생각해보면 컨트롤러가 자료를 어떻게 처리하는지 이해하는데 도움이 될 것이다.

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

즉, 파라미터로 넘겨 받은 `id` 값을 이용하여 특정 `post`를 조회한 후 `@post` 인스턴스 변수에 할당한다.

이 기능은 **필터**라고 하며 이전에는 `before_filter`, `after_filter`, `around_filter`로 사용되었지만 **레일스 4**부터 `_filter`가 `_action`으로 변경되었다. 따라서 각각 `before_action`, `after_action`, `around_action`으로 사용된다.

### Strong Parameters

**레일스 3**에서는 각 모델 속성에 대한 접근을 제한하기 위해 모델 클래스에서 접근 가능한 속성(white list)을 `attr_accessible` 매크로로 선언했다.

```
class User < ActiveRecord::Base
  attr_accessible :first, :last, :email
end
```

즉, `User` 모델의 `first`, `last`, `email` 속성만을 [mass assignment](http://code.tutsplus.com/tutorials/mass-assignment-rails-and-you--net-31695)로 저장할 수 있다는 것이다.

그러나 **레일스 4**로 업그레이드되면서 이러한 속성 보안관련 기능이 모델로부터 컨트롤러로 이동하여 [Strong Parameters](http://richonrails.com/articles/rails-4-preview-strong-parameters)의 개념으로 재구성되었다.

`posts` 컨트롤러 클래스 파일의 하단에는 아래와 같이 정의되어 있다.

```ruby
private
  def post_params
    params.require(:post).permit(:title, :content)
  end
```

즉, 파라미터로 넘겨 받은 속성 중에 `title`과 `content`만을 화이트리스트(white-list)로 인정하겠다는 뜻이다. 따라서 다른 속성은 `save` 또는 `udpate` 되지 않게 된다.


---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_05


