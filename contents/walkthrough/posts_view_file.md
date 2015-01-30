# posts 뷰 변경

`index` 액션의 뷰 파일 중 해당 부분을 아래와 같이 변경하고, 인스턴스 변수(`@posts`)를 사용하는 부분만 집중해서 보자.

```html
<% @posts.each do |post| %>
  <tr>
    <td><%= post.title %></td>
    <td>
      <%= link_to 'Show', [post.bulletin, post], class:'btn btn-default' %>
      <%= link_to 'Edit', edit_bulletin_post_path(post.bulletin, post), class:'btn btn-default' %>
      <%= link_to 'Destroy', [post.bulletin, post], method: :delete, data: { confirm: 'Are you sure?' }, class:'btn btn-default' %>
    </td>
  </tr>
<% end %>
```

`@posts` 인스턴스 변수는 배열을 가진다. 루비의 배열 메소드인 `each`는 리시버(`.each` 앞에 있는 객체)의 각 요소를 하나씩 반복해서 `do` 블록 변수(여기서는 post)로 넘겨 준다. 따라서 `<%= post.title %>`는 `post` 객체의 `title` 속성값이 삽입된다.


### 기타 posts 뷰 파일의 변경 사항

* `app/views/posts/_form.html.erb`에서 아래와 같이 수정한다.

  ```
-<%= form_for(@post) do |f| %>
+<%= simple_form_for([@bulletin, @post]) do |f| %>
```


* `app/views/posts/edit.html.erb`

  ```
-<%= link_to 'Show', @post, class: 'btn btn-default' %>
-<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
+<%= link_to 'Show', [@post.bulletin, @post], class: 'btn btn-default' %>
+<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```
* `app/views/posts/index.html.erb`

  ```
-<%= link_to 'New Post', post_path, class: 'btn btn-default' %>
+<%= link_to 'New Post', new_bulletin_post_path, class: 'btn btn-default' %>
  ```

* `app/views/posts/new.html.erb`

  ```
-<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
+<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```


* `app/views/posts/show.html.erb`

  ```
-<%= link_to 'Edit', edit_post_path(@post), class: 'btn btn-default' %>
-<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
+<%= link_to 'Edit', edit_bulletin_post_path(@post.bulletin, @post), class: 'btn btn-default' %>
+<%= link_to 'Back', bulletin_posts_path, class: 'btn btn-default' %>
```

### navbar 메뉴변경

`app/views/layouts/application.html.erb` 파일에서 아래와 같이 `<ul class='nav navbar-nav'>` 부분을 아래와 같이 변경한다.

```html
<ul class="nav navbar-nav">
  <li class="<%= params[:bulletin_id] == '1' ? 'active' : '' %>"><%= link_to '공지사항', bulletin_posts_path('1') %></li>
  <li class="<%= params[:bulletin_id] == '2' ? 'active' : '' %>"><%= link_to '새소식', bulletin_posts_path('2') %></li>
  <li class="<%= params[:bulletin_id] == '3' ? 'active' : '' %>"><%= link_to '가입인사', bulletin_posts_path('3') %></li>
</ul>
```

그리고, `http://localhost:3000/bulletins` 로 접속한 후 `New Bulletin` 버튼을 클릭하여 "새소식"과 "가입인사" 게시판을 추가한다. 

우리가 의도한 바는 상단 메뉴 항목를 클릭하면 해당 게시판으로 이동하고 해당 항목이 주황색의 글씨로 표시되도록 하는 것이다.


![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2015-01-30_22-02-30_zpsb40d5eb8.png)


---
> **Git소스** https://github.com/rorlakr/rcafe/tree/chapter_05_09
