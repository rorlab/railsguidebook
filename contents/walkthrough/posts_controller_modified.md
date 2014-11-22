# posts 컨트롤러 변경

### 중첩 라우팅

RESTful URI로부터 `bulletin_id` 또는 `post`의 `id` 값을 받아 해당 `bulletin` 객체를 생성하거나 이 `bulletin` 객체의 `posts` 개체를 불러오기 위해서는 `중첩 라우팅`(nested routing) 기법을 사용하면 편리하다.

`config/routes.rb` 파일을 열어 아래와 같이 수정한다.

``` ruby
Rails.application.routes.draw do
  resources :bulletins do
    resources :posts
  end

  root 'welcome#index'
end
```

위와 같이 리소스 라우트를 중첩하면 아래와 같은 라우팅을 사용할 수 있게 된다. 이것을 콘솔에서 확인해 보자.

``` bash
$ bin/rake routes
            Prefix Verb   URI Pattern                                      Controller#Action
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

위와 같은 라우팅 테이블에서 `URI Pattern`을 주목하자. 외부로부터 들어오는 요청이 이 테이블의 `URI Pattern`과 일치할 경우 매핑되는 컨트롤러의 액션이 호출된다. 이 때 `URI Pattern` 중 심볼에 매칭되는 부분은 `params[]` 해쉬의 키로 사용되어 해당 파라미터의 값을 불러올 수 있게 된다. 위의 예에서는 `params[:bulletin_id]` 키에 해당하는 파라미터 값을 불러와 액션에서 사용할 수 있게 된다.

특정 게시판의 게시물 목록을 불러오는 예를 들어 보자.

```bash
Prefix : bulletin_posts
Verb : GET
URI Pattern : /bulletins/:bulletin_id/posts(.:format)
Controller#Action : posts#index
```

`Prefix` 끝에 `_path` 또는 `_url`을 붙여 헬퍼메소드로 사용할 수 있는데, 뷰 템플릿에서 동적 URL을 사용할 수 있도록 해 주어 정적 URL을 일일이 입력할 필요가 없게 된다. 즉, 뷰 템플릿 파일에서 `<%= bulletin_posts_path('공지사항') %>`와 같이 사용하면 뷰 파일이 렌더링될 때 `http://localhost:3000/bulletins/공지사항/posts`으로 변환된다.

또한 이 `URI Pattern`에 매핑되는 해당 컨트롤러의 액션에서는 `params[:bulletin_id]` 해시키를 이용하여 `'공지사항'` 값을 얻을 수 있게 된다.

### posts 컨트롤러의 변경

`Controller#Action`의 `posts#index`는 `posts` 컨트롤러의 `index` 액션을 호출하라는 의미이다. 여기서는  `app/controllers/posts_controller.rb` 파일의 `index` 액션 코드만을 주목하자.

``` ruby
class PostsController < ApplicationController

 before_action :set_bulletin
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def index
    @posts = @bulletin.posts.all
  end

  def show
  end

  def new
    @post = @bulletin.posts.new
  end

  def edit
  end

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

  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to bulletin_posts_url, notice: 'Post was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

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

`index` 액션의 인스턴스 변수 `@bulletin`는, 액션 실행전에 수행되는 `set_bulletin` 메소드의 결과로 만들어진다. 이 `private`  메소드는 `posts` 컨트롤러의 모든 액션이 실행되기 전에 수행된다. 반면에, `before_action`인 `set_post` 메소드는 `:show` `:edit`, `:update`, `:destroy` 액션이 실행되기 전에만 수행된다(`before_action` 매크로의 `:only` 옵션으로 지정함). 따라서 `index` 액션이 실행될 때는 `set_posts` 메소드가 사전에 실행되지 않게 된다. `index` 액션에서는 궁극적으로 인스턴스 변수 `@posts`를 생성하게 되고 이것은 해당 뷰 템플릿(`app/views/posts/index.html.erb`)에서 바로 사용할 수 있게 된다.



---
> **Git소스** https://github.com/rorlab/rcafe/tree/제5.9장
