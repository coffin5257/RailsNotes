**開發速度:透過CRUD 七個 action + HTTP 四個 verb + 表單 Helper 與 URL Helper 達到高速開發**
REST 之所以能簡化開發,是因為其所引入的架構約束。

## Rails 中的 REST implementation 將 controller 的 method 限制在七個:

 - index
 - show
 - new
 - edit
 - create
 - update
 - destroy
 
實際上就是整個 CRUD。而在實務上,web services 的需要的操作行為,其實也不脫這七
種。

## Rails 透過 HTTP 作為 generic connector interface,使用 HTTP 的四種 verb :GET、POST、PUT、DELETE 對資源進行操作。

 - GET
```
<%=link_to("List", posts_path) %>
<%=link_to("Show", post_path(post)) %>
<%=link_to("New", new_post_path) %>
<%=link_to("Edit", edit_post_path(post)) %>
```
 - POST
```
<%= form_for @post , :url => posts_path , :html => {:method => :post} do |f| %>
```
 - PUT
 
```
<%= form_for @post , :url => post_path(@post) , :html => {:method => :put} do |f| %>
```
- Destroy
```
<%= link_to("Destroy", post_path(@post), :method => :delete )
```

## Form 綁定 Model Attribute 的設計
Rails 的表單欄位,是對應 Model Attribute 的:

```
<h1>New post</h1>
    <%= form_for @post , :url => posts_path do |f| %>
        <%= f.error_messages %>
        <div><label>subject</label><%= f.text_field :subject %></div>
        <div><label>content</label><%= f.text_area :content %> </div>
        <%= f.submit "Submit", :disable_with => 'Submiting...' %>
<% end -%>

<%= link_to 'Back', posts_path %>
```

透過 form_for 傳送出來的表單,會被壓縮包裝成一個 parameter: params[:post]:
```Ruby
def create
    @post = Post.new(params[:post])
    if @post.save
        flash[:notice] = 'Post was successfully created.'
        redirect_to post_path(@post)
    else
        render :action => "new"
    end
end
```
如此一來,原本創造新資源的一連串繁複動作就可以大幅的被簡化,開發速度達到令人驚
艷的地步。

## More about RESTful on Rails

### Collection & Member

#### Member
宣告這個動作是屬於單個資源的 URI。可以透過 /photos/1/preview 進行 GET。有 `preview_photo_path(photo)` 或 `preview_photo_url(photo)` 這樣的 `Url Helper` 可以用。
```
resources :photos do
    member do
        get 'preview'
    end
end
```
也可以使用這種寫法:
```
resources :photos do
    get 'preview', :on => :member
end
```
#### Collection
宣告這個動作是屬於一組資源的 URI。可以透過 /photos/search 進行 GET。有 `search_photos_path` 或 `search_photos_url` 這樣的 `Url Helper` 可以用。
```
resources :photos do
    collection do
        get 'search'
    end
end
```
也可以使用這種寫法:
```
resources :photos do
    get 'search', :on => :collection
end
```

### Nested Resources

有時候我們會需要使用雙重 resources 來表示一組資源。比如說 Post 與 Comment :
```
resources :posts do
    resources :comments
end
```
可以透過 /posts/123/comments 進行 GET。有 post_comments_path(post) 或 post_comments_-url(post) 這樣的 Url Helper 可以用。
而這樣的 URL 也很 清 楚 的 表 明, 這 組 資 源 就 是 用 來 存 取 編 號 123 的 post 下 所 有 的 comments。
而要拿取 Post 的 id 可以透過這樣的方式取得:
```Ruby
class CommentsController < ApplicationController
    before_filter :find_post
    
    def index
        @comments = @post.comments
    end
        
    protected
        
    def find_post
        @post = Post.find(params[:post_id])
    end
end
```

### Namespace Resources
但是像 /admin/posts/1/edit 這種 URL,用 Nested Resources 應該造不出來吧?沒錯,這樣的 URL 應該要使用 Namespace 去建構:
```Ruby
namespace :admin do
    # Directs /admin/products/* to Admin::PostsController
    # (app/controllers/admin/posts_controller.rb)
    resources :posts
end
```
可以透過 `/admin/posts/123/edit` 進行 `GET`。有 `edit_admin_post_path(post)` 或 `edit_admin_post_url(post)` 這樣的 `Url Helper` 可以用。