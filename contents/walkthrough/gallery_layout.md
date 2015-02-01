# 갤러리형 레이아웃 작성하기

이전에 언급한 바와 같이 게시판의 종류를 세가지로 분류한 바 있다. 일반형, 블로그형, 갤러리형 세가지 중에 일반형과 블로그형은 이미 전용 레이아웃을 구현하였고, 이제 남은 건 `갤러리형` 레이아웃을 만드는 것이다.

`갤러리형` 게시판은 이미지를 업로드할 수 있으면 업로드된 이미지들은 쎔네일 형태로 보이도록 하고자 한다.

이미지를 업로드를 구현하기 위해서는 두가지 젬을 사용할 수 있다. 하나는 [`paperclip`](https://github.com/thoughtbot/paperclip)이고 다른 하나는 [`carrierwave`](https://github.com/carrierwaveuploader/carrierwave)라는 젬이다.

이 두 젬은 모두 시스템에 `ImageMagick`이라는 툴이 설치되어 있어야 한다. 시스템에서 아래와 같은 명령으로 설치 여부를 알 수 있다. [`ghostscript`](http://www.ghostscript.com)도 함께 설치하면 PDF 파일 업로드시 첫페이지 이미지를 썸네일로 만들 수 있다. 

```bash
$ convert

Version: ImageMagick 6.8.7-7 Q16 x86_64 2013-11-27 http://www.imagemagick.org
Copyright: Copyright (C) 1999-2014 ImageMagick Studio LLC
Features: DPC Modules
Delegates: bzlib freetype jng jpeg ltdl png xml zlib
~중략~
```

만약 설치되어 있지 않으면 [`ImageMagick 설치하기`](../../appendices/imagemagick.html)를 참고하여 설치하면 된다.

### Carrierwave 젬 설치하기

여기서는 [`carrierwave`](https://github.com/carrierwaveuploader/carrierwave) 젬을 사용하기로 한다.

Gemfile에 아래와 같이 젬을 추가한다.

```ruby
gem 'carrierwave'
```

젬을 설치한다.

```bash
$ bundle install
```

### 한글 파일명의 인코딩 문제

`carreirwave` 젬을 사용할 때에는 한글파일명을 가진 파일을 업로드할 때 문제가 있다. 즉, 한글 파일명을 가진 파일이 업로드되면 `sanitization` 과정에서 한글이 모두 "_" 문자로 대체되어 파일명을 알수 없게 된다.

`github`에 [해결책](https://github.com/jnicklas/carrierwave#securing-uploads)이 기술되어 있어 소개한다.

`config/initializers/carrierwave.rb` 파일을 생성한 후 아래의 코드를 추가해 주기만 하면 한글파일명이 깨지지 않고 그대로 업로드되는 것을 확인할 수 있다.

```ruby
# Allow non-ascii letters in uploaded filenames
CarrierWave::SanitizedFile.sanitize_regexp = /[^[:word:]\.\-\+]/
```

그러나 이렇게 하면 한글파일명 중에 스페이스가 포함되면 이 또한 `"_"` 문자로 보이게 되는데, 이것 마저도 스페이스 그대로 보이게 하려면 `"\s"`를 추가해 주면 된다.

```ruby
# Allow non-ascii letters in uploaded filenames
CarrierWave::SanitizedFile.sanitize_regexp = /[^[:word:]\s\.\-\+]/
```

> **Caution** : 젬을 추가하거나 설정파일을 변경한 경우에는 반드시 웹서버를 다시 시작해야 한다.

### 업로드 클래스의 생성

이미지 업로드를 위한 `Picture`라는 이름을 가지는 업로더를 생성한다.

```bash
$ bin/rails g uploader Picture
      create  app/uploaders/picture_uploader.rb
```

생성된 `PictureUploader` 클래스 파일의 내용은 아래와 같다


```ruby
# encoding: utf-8

class PictureUploader < CarrierWave::Uploader::Base

  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  # include CarrierWave::MiniMagick

  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog

  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # Provide a default URL as a default if there hasn't been a file uploaded:
  # def default_url
  #   # For Rails 3.1+ asset pipeline compatibility:
  #   # ActionController::Base.helpers.asset_path("fallback/" + [version_name, "default.png"].compact.join('_'))
  #
  #   "/images/fallback/" + [version_name, "default.png"].compact.join('_')
  # end

  # Process files as they are uploaded:
  # process :scale => [200, 300]
  #
  # def scale(width, height)
  #   # do something
  # end

  # Create different versions of your uploaded files:
  # version :thumb do
  #   process :resize_to_fit => [50, 50]
  # end

  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
  # def extension_white_list
  #   %w(jpg jpeg gif png)
  # end

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end

end
```

사용법에 대한 자세한 안내가 코멘트로 처리되어 있다. 코멘트를 제거한 클래스를 보면 아래와 같다.

```ruby
# encoding: utf-8

class PictureUploader < CarrierWave::Uploader::Base

  storage :file

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

end
```

### 빈 디렉토리 자동으로 삭제하기

업로드한 이미지를 삭제하면 해당 폴더가 남아 있게 된다. 아래와 같이 업로더 클래스(`PictureUploader`)에 추가하면 자동으로 빈 폴더가 삭제된다. 자세한 내용은 [여기](https://github.com/carrierwaveuploader/carrierwave/wiki/How-to%3A-Make-a-fast-lookup-able-storage-directory-structure)를 참고하라.

```ruby
# 업로드 상단에 아래의 after 매크로를 추가한다.
after :remove, :delete_empty_upstream_dirs

def delete_empty_upstream_dirs
  path = ::File.expand_path(store_dir, root)
  Dir.delete(path) # fails if path not empty dir

  path = ::File.expand_path(base_store_dir, root)
  Dir.delete(path) # fails if path not empty dir
rescue SystemCallError
  true # nothing, the dir is not empty
end
```

### Picture 속성 추가하기

`Post` 클래스에 `picture` 속성을 추가하기 위해서 아래와 같이 마이그레이션 클래스 파일을 생성하고 DB 마이그레이션 한다.

```bash
$ bin/rails g migration add_picture_to_posts picture
      invoke  active_record
      create    db/migrate/20150201000532_add_picture_to_posts.rb

$ bin/rake db:migrate
== 20150201000532 AddPictureToPosts: migrating ================================
-- add_column(:posts, :picture, :string)
   -> 0.0007s
== 20150201000532 AddPictureToPosts: migrated (0.0007s) =======================
```

### 업로더 마운트하기

`Post` 클래스 파일(`app/models/post.rb`)을 열어 `PictureUploader`  업로더 클래스를 `picture` 속성으로 마우트한다.

```ruby
class Post < ActiveRecord::Base
  belongs_to :bulletin
  mount_uploader :picture, PictureUploader
end
```

또한 `posts` 컨트롤러의 params 해시에 `picture`와 `picture_cahe` 속성을 추가한다. 

```ruby
def post_params
  params.require(:post).permit(:title, :content, :picture, :picture_cache)
end
```


### MiniMagick 젬 추가하기

이미지 크기를 조절하기 위해서 `Rmagick`이나 `MiniMagick` 젬을 추가한다. `carrierwave` 문서에 따르면 `MiniMagick` 젬을 추천하므로 아래와 같이 `Gemfile`에 `minimagick` 젬을 추가하고,

```
gem 'mini_magick'
```

젬을 설치한다.

```bash
$ bundle install
```

### 업로더에 이미지 크기 옵션 추가하기

`app/uploaders/picture_uploader.rb` 파일을 열고 아래의 내용을 추가한다.

```ruby
# encoding: utf-8

class PictureUploader < CarrierWave::Uploader::Base

  include CarrierWave::MiniMagick

  ...

  process :resize_to_fit => [800, 800]

  version :thumb do
    process :resize_to_fill => [200,200]
  end

end
```

### Post 폼에 파일업로드 추가하기

폼 `partial` 템플릿 파일(`app/views/posts/_form.html.erb`)을 아래와 같이 변경한다.

```erb
<%= simple_form_for([@bulletin, @post]) do |f| %>
  <%= f.error_notification %>

  <div class="form-inputs">
    <%= f.input :title %>
    <%= f.input :content, input_html: { rows: 10 } %>

    <% if @post.bulletin.post_type == "gallery" %>
      <%= f.input :picture, as: :file %>
      <%= f.hidden_field :picture_cache %>
    <% end %>
  </div>

  <div class="form-actions">
    <%= f.button :submit %>
  </div>
<% end %>

```

위에서 `erb` 코드 부분을 보면, `gallery`형 게시판에서만 이미지를 업로드할 수 있도록 조건을 추가한 것을 주목하자.

그리고 파일 업로드 `input`에 대한 스타일을 `app/assets/stylesheets/posts.scss` 파일에 추가한다. 

```css
input[type='file'] {
  border: 1px solid #d1d1d1;
  padding: .5em;
  border-radius: .3em;
  width: 100%;
}
```

### 갤러리 게시판을 생성

이미지를 업로드하는 게시판을 생성하기 위해서 `http://localhost:3000/bulletins`로 접속한 후 아래와 같이 "갤러리"라는 게시판을 추가한다. 이 때 `Post_type`에서 `갤러리`로 선택하고 저장한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_09-15-25_zps70fcf00e.png)

이렇게 해서 게시판은 총 4개가 되었다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_09-18-38_zps3ceb0979.png)

이제 어플리케이션 레이아웃 파일(`app/views/layouts/application.html.erb`)을 열고, 상단 메뉴항목에 `갤러리`를 추가한다.

```erb
...
<li class="<%= params[:bulletin_id] == '4' ? 'active' : '' %>"><%= link_to '갤러리', bulletin_posts_path(4) %></li>
...
```

다음은, `awesome` 폰트를 사용하기 위해서 아래와 같이 `Gemfile`에 추가하고 

```ruby
gem "font-awesome-rails"
```

`bundle install` 후 웹서버를 다시 시작한다.

그리고 `app/assets/stylesheets/application.scss` 파일을 열고 아래와 같이 추가한다. 

```ruby
...
@import 'bootstrap';
@import "font-awesome";  <<< 추가
@import 'posts';
...
```

> **Info** : 이 젬은 `fa_icon`이라는 헬퍼메소드를 지원해 준다. 사용법은 [여기](https://github.com/bokmann/font-awesome-rails#helpers)를 참고한다. 

다음에는 `갤러리`용 `partial` 템플릿 파일을 생성하기 위해  `app/views/posts/post_types/` 디렉토리에 `_gallery.html.erb` 파일을 추가하고 아래와 같이 작성한다.

```erb
<h2><%= bulletin_name params[:bulletin_id] %></h2>

<div class='gallery'>
<% @posts.each do | post | %>
    <div class='col-lg-3 col-md-3 col-xs-4  picture'>
      <div class='image'><%= link_to(image_tag(post.picture_url(:thumb)), [post.bulletin, post]) if post.picture? %></div>
      <div class='title'><%= post.title %></div>
      <div class='content'><%= post.content %></div>
      <div class='actions'>
          <%= link_to fa_icon('eye'), [post.bulletin, post] %>
          <%= link_to fa_icon('edit'), edit_bulletin_post_path(post.bulletin, post) %>
          <%= link_to fa_icon('trash'), [post.bulletin, post], method: :delete, data: { confirm: 'Are you sure?' } %>
      </div>
    </div>
<% end %>
</div>
<br>

<%= link_to 'New Post', new_bulletin_post_path, class: 'btn btn-default' %>
```

`app/assets/stylesheets/posts.scss` 파일에 아래와 같이 추가한다.

```css
.gallery {
  overflow:auto;
  .picture {
    position:relative;
    margin:1.5em 0;
    .image img {
      border: 1px solid #928c75;
    }
    .title {}
    .content {}
    .actions {
      position:absolute;
    }
  }
}
```

갤러리 게시판에서 이미지를 추가하는 화면은 아래와 같다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_13-03-17_zps11cb5681.png)


또한, 업로더 클래스에 아래와 같이 업로드할 수 있는 파일 포맷을 지정할 수 있다.

```ruby
...
def extension_white_list
  %w(jpg jpeg gif png pdf)
end
...
```

이제 `.jpg`, `.jpeg`, `.gif`, `.png`, `.pdf` 확장자를 가진 파일만 업로드할 수 있게 된다.

이러한 파일확장자 이외의 파일을 업로드하면 아래와 같은 에러 메시지가 표시된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_13-32-37_zps404bdb11.png)



`.pdf` 파일을 업로드할 경우에는 `pdf` 파일의 첫페이지가 쎔네일 이미지로 만들어진다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_16-46-07_zps8d9481b4.png)

각 게시판형에 따른 레이아웃용 `partial` 템플릿 파일에서 헤더 부분에 아래와 같이 추가하여 게시판 제목 옆에 `설정` 링크를 두면 좋겠다. (`_bulletin.html.erb`, `_blog.html.erb`, `_gallery.html.erb`)

```erb
<h2><%= bulletin_name params[:bulletin_id] %> <small><%= link_to '설정', edit_bulletin_path(params[:bulletin_id])%></small></h2>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-02-01_14-21-30_zpsbd72d3bc.png)


> **Info** : 업로드된 파일은 `public/uploads/` 디렉토리에 저장된다. 이 파일들은 소스관리를 할 필요가 없기 때문에, `.gitignore` 파일을 열어 하단에 아래와 같이 추가해 준다.

```
...
public/uploads/*
```

---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_11
