# posts 컨트롤러 변경

### 중첩 라우팅

URI로부터 `bulletin_id` 파라미터 값과 `post`의 `id` 파라미터 값을 받아 특정 `bulletin` 객체의 `post` 개체들을 불러올 수 있다면 매우 편리할 것이다. 레일스에서는 이러한 목적으로 `중첩 라우팅`(nested routing)을 지원한다.

`config/routes.rb` 파일을 열어 아래와 같이 수정한다.

``` ruby
Rails.application.routes.draw do
  resources :posts
  resources :bulletins do
    resources :posts
  end

  root 'welcome#index'
end
```

위와 같이 리소스 라우트를 중첩하여 사용하면 아래와 같은 라우팅을 사용할 수 있게 된다. 이것을 콘솔에서 확인해 보자.

``` bash
$ bin/rails routes
            Prefix Verb   URI Pattern                                      Controller#Action
             posts GET    /posts(.:format)                                 posts#index
                   POST   /posts(.:format)                                 posts#create
          new_post GET    /posts/new(.:format)                             posts#new
         edit_post GET    /posts/:id/edit(.:format)                        posts#edit
              post GET    /posts/:id(.:format)                             posts#show
                   PATCH  /posts/:id(.:format)                             posts#update
                   PUT    /posts/:id(.:format)                             posts#update
                   DELETE /posts/:id(.:format)                             posts#destroy
    bulletin_posts GET    /bulletins/:bulletin_id/posts(.:format)          posts#index
                   POST   /bulletins/:bulletin_id/posts(.:format)          posts#create
 new_bulletin_post GET    /bulletins/:bulletin_id/posts/new(.:format)      posts#new
edit_bulletin_post GET    /bulletins/:bulletin_id/posts/:id/edit(.:format) posts#edit
     bulletin_post GET    /bulletins/:bulletin_id/posts/:id(.:format)      posts#show
                   PATCH  /bulletins/:bulletin_id/posts/:id(.:format)      posts#update
                   PUT    /bulletins/:bulletin_id/posts/:id(.:format)      posts#update
                   DELETE /bulletins/:bulletin_id/posts/:id(.:format)      posts#destroy
         bulletins GET    /bulletins(.:format)                             bulletins#index
                   POST   /bulletins(.:format)                             bulletins#create
      new_bulletin GET    /bulletins/new(.:format)                         bulletins#new
     edit_bulletin GET    /bulletins/:id/edit(.:format)                    bulletins#edit
          bulletin GET    /bulletins/:id(.:format)                         bulletins#show
                   PATCH  /bulletins/:id(.:format)                         bulletins#update
                   PUT    /bulletins/:id(.:format)                         bulletins#update
                   DELETE /bulletins/:id(.:format)                         bulletins#destroy
              root GET    /                                                welcome#index
```

위와 같은 라우팅 테이블에서 `URI Pattern`을 주목하자. 외부로부터 들어오는 요청이 이 테이블의 `URI Pattern`과 일치할 경우 매핑되는 컨트롤러의 액션이 호출된다. 이 때 컨트롤러에서는 `URI Pattern` 중 심볼에 매칭되는 부분(URI의 동적 세그먼트)은 `params` 해쉬의 키로 사용되어 해당 파라미터의 값을 불러올 수 있게 된다. `params[:bulletin_id]`, `params[:id]`와 같이 사용하여 컨트롤러 액션에서 사용할 수 있게 된다.

특정 게시판의 게시물 목록을 불러오는 예를 들어 보자.

```bash
Prefix : bulletin_posts
Verb : GET
URI Pattern : /bulletins/:bulletin_id/posts(.:format)
Controller#Action : posts#index
```

`Prefix` 끝에 `_path` 또는 `_url`을 붙여 헬퍼메소드로 사용할 수 있는데, 뷰 템플릿에서 정적 URL을 일일이 입력할 필요가 없게 된다. 즉, 뷰 템플릿 파일에서 `<%= bulletin_posts_path('공지사항') %>`와 같이 사용하면 뷰 파일이 렌더링될 때 `http://localhost:3000/bulletins/공지사항/posts`으로 변환된다.

또한 이 `URI Pattern`에 매핑되는 해당 컨트롤러의 액션에서는 `params[:bulletin_id]` 해시키를 이용하여 `'공지사항'` 값을 얻을 수 있게 된다.

### posts 컨트롤러의 변경

`:bulletins` 와 `:posts` 리소스의 중첩 라우팅을 사용하기 위해서는 `posts` 컨트롤러도 수정해야 한다. 먼저 변경된 `posts` 컨트롤러 전체를 살펴보고 바뀐 부분을 분석해보자. `posts` 리소스는 두 가지 라우팅을 가지도록 했다. 하나는 기존과 같이 단독으로 사용할 때이고 다른 하나는 `bulletins` 리소스와 중첩해서 사용할 때이다. 이제 이 두 가지 라우팅을 모두 충족할 수 있도록 컨트롤러와 뷰 파일을 변경할 것이다. 수정된 `posts` 컨트롤러는 아래와 같다.  

```ruby
class PostsController < ApplicationController
  before_action :set_bulletin
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def index
    @posts = @bulletin.present? ? @bulletin.posts.all : Post.all
  end

  def show
  end

  def new
    @post = @bulletin.present? ? @bulletin.posts.new : Post.new
  end

  def edit
  end

  def create
    @post = @bulletin.present? ? @bulletin.posts.new(post_params) : Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to (@bulletin.present? ? [@post.bulletin, @post] : @post), notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to (@bulletin.present? ? [@post.bulletin, @post] : @post), notice: 'Post was successfully updated.' }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to (@bulletin.present? ? bulletin_posts_url : posts_url), notice: 'Post was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
  def set_bulletin
    @bulletin = Bulletin.find(params[:bulletin_id]) if params[:bulletin_id].present?
  end

  def set_post
    if @bulletin.present?
      @post = @bulletin.posts.find(params[:id])
    else
      @post = Post.find(params[:id])
    end
  end

  def post_params
    params.require(:post).permit(:title, :content)
  end
end
```

`app/controllers/posts_controller.rb` 파일을 열고 위의 내용을 복사해서 붙여 넣기 한다.

먼저 `private` 메소드인 `set_bulletin`을 생성하고 `before_action` 필터로 지정한다. 이 메소드는 `posts` 컨트롤러의 모든 액션이 실행되기 전에 수행될 것이다. 또 다른 `before_action`인  `set_post` 메소드는 `only` 옵션으로 `show`, `edit`, `update`, `destroy` 액션을 추가했으며 이것은 이 메소드를 실행할 액션을 지정하게 된다.

``` ruby
class PostsController < ApplicationController
  before_action :set_bulletin
  before_action :set_post, only: [:show, :edit, :update, :destroy]
```

`index` 액션에서 인스턴스 변수 `@bulletin`이 `posts` 앞에 추가되었다. `@bulletin`은 `set_bulletin` 메소드에서 생성되는데 선택한 게시판(bulletin)에 대한 객체가 할당된다. 이렇게 해서 특정 게시판에 속하는 글을 모두 보여주거나(`@bulletin.posts.all`) 글을 저장할 때 해당 게시판에 포함되도록(`@bulletin.posts.new`) 할 수 있다.

``` ruby
  def index
    @posts = @bulletin.present? ? @bulletin.posts.all :   Post.all
  end

  def show
  end

  def new
    @post = @bulletin.present? ? @bulletin.posts.new : Post.new
  end

  def edit
  end
```

새로운 글을 생성하는 `create` 액션에서도 게시판과 글의 종속 관계를 정의한다.(`@bulletin.posts.new`) 원래는 `redirect_to @post`로 객체를 다음 `show` 액션으로 리다이렉트했지만 게시판과의 종속 관계 때문에 리다이렉트 하는 과정에서 어떤 게시판에 속하는 `post`인지 알려줘야 하므로 `redirect_to [@post.bulletin, @post]`로 변경되었다. `update` 액션도 마찬가지다. `[@post.bulletin, @post]`와 같이 종속관계의 두 객체를 배열로 표시하는 것은 `bulletin_post_path(@post.bulletin, @post)` 또는 `url_for([@post.bulletin, @post])`의 축약형이다.

``` ruby
  def create
    @post = @bulletin.posts.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to [@post.bulletin, @post], notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to [@post.bulletin, @post], notice: 'Post was successfully updated.' }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
```

`destroy` 액션에서 글을 삭제하는 과정은 변함이 없다. `Bulletin` 모델과 `Post` 모델과의 관계는 일대다 관계로 글이 삭제된다고 해당 게시판에 영향을 미치지는 않기 때문이다. 반대로 게시판이 삭제되면 해당 게시판 내의 글이 모두 삭제되어야 하는데 이는 `Bulletin` 모델을 선언할 때 `Post` 모델과의 관계를 `dependent: :destroy`로 정의했기 때문에 자동으로 처리된다. 객체를 삭제한 다음 액션이 종료될 때 `index` 액션으로 리다이렉트 되는데 중첩 라우팅에 의해 `index` 액션의 경로 헬퍼 prefix가 `posts`에서 `bulletin_posts`로 바뀌었다. 따라서 리다이렉트 url도 `bulletin_posts_url`로 수정한다.

``` ruby
  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to bulletin_posts_url, notice: 'Post was successfully destroyed.' }
      format.json { head :no_content }
    end
  end
```

마지막으로 `private` 메소드가 남았다. `private` 메소드는 클래스 안에서만 호출할 수 있는데, `before_action` 필터에 의해 먼저 실행될 `set_bulletin` 메소드는 파라미터로 넘겨 받은 게시판 id를 통해 해당 객체를 `@bulletin` 인스턴스 변수에 할당한다. 이렇게 해서 `posts` 컨트롤러에서 각 액션을 처리할 때 해당 글이 속하는 게시판 정보를 함께 처리할 수 있게 된다.

``` ruby
  private
    def set_bulletin
      @bulletin = Bulletin.friendly.find(params[:bulletin_id])
    end

    def set_post
      @post = @bulletin.posts.find(params[:id])
    end

    def post_params
      params.require(:post).permit(:title, :content)
    end
end
```

---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_08
