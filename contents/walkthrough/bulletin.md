# Bulletin 모델의 생성

때로는 글의 성격에 따라 별도로 관리할 필요가 있다. `게시판`의 개념을 도입하면 글을 게시판별로 묶을 수 있다. 이를 위해서 `Bulletin`이란 모델을 작성하기로 한다.

```bash
$ bin/rails g scaffold Bulletin title description:text
Running via Spring preloader in process 9842
      invoke  active_record
      create    db/migrate/20161213103856_create_bulletins.rb
      create    app/models/bulletin.rb
      invoke    test_unit
      create      test/models/bulletin_test.rb
      create      test/fixtures/bulletins.yml
      invoke  resource_route
       route    resources :bulletins
      invoke  scaffold_controller
      create    app/controllers/bulletins_controller.rb
      invoke    erb
      create      app/views/bulletins
      create      app/views/bulletins/index.html.erb
      create      app/views/bulletins/edit.html.erb
      create      app/views/bulletins/show.html.erb
      create      app/views/bulletins/new.html.erb
      create      app/views/bulletins/_form.html.erb
      invoke    test_unit
      create      test/controllers/bulletins_controller_test.rb
      invoke    helper
      create      app/helpers/bulletins_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/bulletins/index.json.jbuilder
      create      app/views/bulletins/show.json.jbuilder
      create      app/views/bulletins/_bulletin.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/bulletins.coffee
      invoke    scss
      create      app/assets/stylesheets/bulletins.scss
      invoke  scss
   identical    app/assets/stylesheets/scaffolds.scss

```

DB 마이그레이션 후 브라우저에서 확인해 보자.

```bash
$ bin/rails db:migrate
Running via Spring preloader in process 10428
== 20161213103856 CreateBulletins: migrating ==================================
-- create_table(:bulletins)
   -> 0.0372s
== 20161213103856 CreateBulletins: migrated (0.0373s) =========================
```

이전의 `posts` 뷰 페이지들과 같이 아래의 뷰 파일들을 `bootstrap` 클래스로 스타일을 수정한 후 브라우저로 확인한다.

```
$ open http://localhost:3000/bulletins
```

### index 액션과 뷰 템플릿 파일

`app/controllers/bulletins_controller.rb` :

```ruby
def index
  @bulletins = Bulletin.all
end
```
`app/views/bulletins/index.html.erb` :

```ERB
<h2>Bulletins</h2>

<table class="table">
  <thead>
    <tr>
      <th>Title</th>
      <th>Description</th>
      <th>Data actions</th>
    </tr>
  </thead>

  <tbody>
    <% @bulletins.each do |bulletin| %>
      <tr>
        <td><%= bulletin.title %></td>
        <td><%= bulletin.description %></td>
        <td>
          <%= link_to 'Show', bulletin, class: 'btn btn-default' %>
          <%= link_to 'Edit', edit_bulletin_path(bulletin), class: 'btn btn-default' %>
          <%= link_to 'Destroy', bulletin, method: :delete, data: { confirm: 'Are you sure?' }, class: 'btn btn-default' %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Bulletin', new_bulletin_path, class: 'btn btn-default' %>
```

테스트용 데이터를 추가하면 아래와 같이 보인다. 

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-13_19-44-23_zps8rsxvd40.png)

### show 액션과 뷰 템플릿 파일

`show` 액션 뷰 템플릿에서는 **Title**과 **Description**을 테이블 형식으로 표시하고 **Created at**이라는 항목을 추가하여 생성한 시각을 보여준다.

```ruby
def show
end
```

```ERB
<h2>Preview Bulletin</h2>

<table class='table table-bordered'>
<tr>
  <th>Title</th>
  <td><%= @bulletin.title %></td>
</tr>
<tr>
  <th>Description</th>
  <td><%= @bulletin.description %></td>
</tr>
<tr>
  <th>Created at</th>
  <td><%= @bulletin.created_at %></td>
</tr>
</table>

<%= link_to 'Edit', edit_bulletin_path(@bulletin), class: 'btn btn-default' %>
<%= link_to 'Back', bulletins_path, class: 'btn btn-default' %>
```

모든 뷰 템플릿을 수정해서 브라우저로 확인한 결과, 게시판을 생성한 시각이 `UTC 타임존`으로 표시된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_19-59-12_zps467a56c9.png)


### form 파셜 템플릿 파일

```ruby
<%= simple_form_for(@bulletin) do |f| %>
  <%= f.error_notification %>

  <div class="form-inputs">
    <%= f.input :title %>
    <%= f.input :description, input_html: { rows: 5 } %>
  </div>

  <div class="form-actions">
    <%= f.button :submit %>
  </div>
<% end %>

```

### new 액션 뷰 템플릿 파일

```ruby
<h2>New bulletin</h2>

<%= render 'form' %>

<hr>

<%= link_to 'Back', bulletins_path, class: 'btn btn-default' %>
```

### edit 액션 뷰 템플릿 파일

```ruby
<h2>Editing bulletin</h2>

<%= render 'form' %>

<hr>

<%= link_to 'Show', @bulletin, class: 'btn btn-default' %>
<%= link_to 'Back', bulletins_path, class: 'btn btn-default' %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_20-01-07_zps31c8eab4.png)


### 타임존(Timezone)

위의 `show` 뷰 템플릿 화면캡쳐에서 `Created at`(생성일) 값을 보면 `2014-05-03 08:59:18 UTC`와 같다. 레일스의 디폴트 타임존 변경은  `config/application.rb` 파일에서 할 수 있다.

```ruby
... 중략~
    # Set Time.zone default to the specified zone and make Active Record auto-convert to this zone.
    # Run "rake -D time" for a list of tasks for finding time zone names. Default is UTC.
    # config.time_zone = 'Central Time (US & Canada)'
... 중략~
```

레일스의 디폴트 타임존은 `UTC`이고, 로컬 타임존 목록을 보기 위해서는 터미널에서 아래와 같이 명령을 실행한다.

```bash
$ bin/rake -D time
rake time:zones:all
    Displays all time zones, also available: time:zones:us, time:zones:local -- filter with OFFSET parameter, e.g., OFFSET=-6

$ bin/rake time:zones:local

* UTC +09:00 *
Irkutsk
Osaka
Sapporo
Seoul
Tokyo
```

타임존을 `Seoul`로 설정하기 위해서는 아래와 같이 값을 변경하고 **로컬 웹서버 다시 시작한다**. (config/application.rb)

```ruby
config.time_zone = 'Seoul'
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_20-04-37_zpsf9d968b0.png)

이제 `Created at` 값이 '2014-05-03 17:59:18 **+0900**' 와 같이 변경된 타임존에 맞게 나타나는 것을 볼 수 있다.

> #### Note::노트
> 
> 이와 같이 타임존을 변경하여 시간을 해당 타임존에 맞게 표시할 수 있지만, 데이터베이스는 값을 항상 UTC 타임존으로 저장한다는 사실을 주목하자. DB로부터 `UTC`로 저장된 시간을 불러와 표시할 때는 레일스가 config/application.rb 에 저장된 time_zone 값에 맞게 자동으로 변경한 후에 표시한다. 그러나, DB에 저장할 때도 로컬 타임존에 맞게 저장하려면 `config.time_zone = 'Seoul'` 아래에  `config.active_record.default_timezone = :local`와 같이 추가해 주면 된다.


---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_06


