## Recipigram app Part4

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  before_action :authenticate_user!, except: [:index] // 追記 authenticate_userはDeviseを導入すると使用できる

  def index
    @recipes = Recipe.all
  end

  def show
    @recipe = Recipe.find(params[:id])
  end

  def new
    @recipe = Recipe.new
  end

  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.user_id = current_user.id
    @recipe.save
    redirect_to recipe_path(@recipe)
  end

  def edit
    @recipe = Recipe.find(params[:id])
  end

  def update
    @recipe = Recipe.find(params[:id])
    @recipe.update(recipe_params)
    redirect_to recipe_path(@recipe)
  end

  def destroy
    @recipe = Recipe.find(params[:id])
    @recipe.destroy
    redirect_to recipes_path
  end

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `app/controller/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  before_action :authenticate_user!, except: [:index] // 追記

  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
  end

  def update
    @user = User.find(params[:id])
    @user.update(user_params)
    redirect_to user_path(@user)
  end

  private
  def user_params
    params.require(:user).permit(:username, :email, :profile, :profile_image)
  end
end
```

+ `app/views/home/index.html.erb`を編集<br>

```
<section class="section">
  <article class="media">
    <div class="media-content">
      <div class="content has-text-centered">
        <p>
          <strong class="is-size-1" style="font-family: cursive;">recipegram</strong>
        </p>
      </div>
      <div class="columns is-centered">
        <div class="column is-5">
          <br>
          <%= link_to "ログイン", new_user_session_path, class: "button is-warning is-fullwidth" %>
          <br>
          <%= link_to "新規登録", new_user_registration_path, class: "button is-warning is-fullwidth" %>
        </div>
      </div>
    </div>
  </article>
  <br>
  <div class="columns is-centered">
    <div class="column">
      <figure>
        <div class="main-image"></div>
      </figure>
    </div>
  </div>
</section>
```

+ `app/assets/image`ディレクトリに`meal.png`ファイルを入れておく<br>

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @recipes = Recipe.all
  end

  def show
    @recipe = Recipe.find(params[:id])
  end

  def new
    @recipe = Recipe.new
  end

  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.user_id = current_user.id
    @recipe.save
    redirect_to recipe_path(@recipe)
  end

  def edit
    @recipe = Recipe.find(params[:id])
    if @recipe.user != current_user // 追記
      redirect_to recipes_path, alert: '不正なアクセスです。'
    end
  end

  def update
    @recipe = Recipe.find(params[:id])
    @recipe.update(recipe_params)
    redirect_to recipe_path(@recipe)
  end

  def destroy
    @recipe = Recipe.find(params[:id])
    @recipe.destroy
    redirect_to recipes_path
  end

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
    if @user.id != current_user.id
      redirect_to users_path, alert: '不正なアクセスです。'
    end
  end

  def update
    @user = User.find(params[:id])
    @user.update(user_params)
    redirect_to user_path(@user)
  end

  private
  def user_params
    params.require(:user).permit(:username, :email, :profile, :profile_image)
  end
end
```

+ `app/models/recipe.rb`を編集<br>

```
class Recipe < ApplicationRecord
  belongs_to :user
  attachment :image

  with_options presence: true do
    validates :title
    validates :body
    validates :image
  end
end
```

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @recipes = Recipe.all
  end

  def show
    @recipe = Recipe.find(params[:id])
  end

  def new
    @recipe = Recipe.new
  end

  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.user_id = current_user.id
    if @recipe.save
      redirect_to recipe_path(@recipe)
    else
      render :new
    end
  end

  def edit
    @recipe = Recipe.find(params[:id])
    if @recipe.user != current_user
      redirect_to recipes_path, alert: '不正なアクセスです。'
    end
  end

  def update
    @recipe = Recipe.find(params[:id])
    if @recipe.update(recipe_params)
      redirect_to recipe_path(@recipe)
    else
      render :edit
    end
  end

  def destroy
    @recipe = Recipe.find(params[:id])
    @recipe.destroy
    redirect_to recipes_path
  end

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `app/views/recipes/new.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        新規レシピ作成
      </h1>
    </div>
  </div>
</section>

<% if @recipe.errors.any? %>
  <div class="notification is-danger">
    <h2><%= pluralize(@recipe.errors.count, "error") %> prohibited this object from being saved: not successfully</h2>
    <ul>
      <% @recipe.errors.full_messages.each do |message| %>
      <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-6">
        <%= form_with(model: @recipe, local: true) do |f| %>
          <div class="field">
            <%= f.label :title, "レシピ名", class: "label" %>
            <%= f.text_field :title, class: "input" %>
          </div>

          <div class="field">
            <%= f.label :body, "作り方", class: "label" %>
            <%= f.text_area :body, class: "textarea" %>
          </div>

          <div class="field">
            <%= f.label :image, "写真", class: "label" %>
            <%= f.attachment_field :image, class: "input" %>
          </div>

          <%= f.submit '投稿', class: "button is-success" %>
        <% end %>
      </div>
    </div>
  </div>
</section>
```

+ `app/views/recipes/edit.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        レシピ編集
      </h1>
    </div>
  </div>
</section>

<% if @recipe.errors.any? %>
  <div class="notification is-danger">
    <h2><%= pluralize(@recipe.errors.count, "error") %> prohibited this object from being saved: not successfully</h2>
    <ul>
      <% @recipe.errors.full_messages.each do |message| %>
      <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-6">
        <%= form_with(model: @recipe, local: true) do |f| %>
          <div class="field">
            <%= f.label :title, "レシピ名", class: "label" %>
            <%= f.text_field :title, class: "input" %>
          </div>

          <div class="field">
            <%= f.label :body, "作り方", class: "label" %>
            <%= f.text_area :body, class: "textarea" %>
          </div>

          <div class="field">
            <%= f.label :image, "写真", class: "label" %>
            <%= f.attachment_field :image, class: "input" %>
          </div>

          <%= f.submit '更新', class: "button is-success" %>
        <% end %>
      </div>
    </div>
  </div>
</section>
```

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
    if @user.id != current_user.id
      redirect_to users_path, alert: '不正なアクセスです。'
    end
  end

  def update
    @user = User.find(params[:id])
    if @user.update(user_params)
      redirect_to user_path(@user)
    else
      render :edit
    end
  end

  private
  def user_params
    params.require(:user).permit(:username, :email, :profile, :profile_image)
  end
end
```

+ `app/views/users/edit.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        ユーザー編集
      </h1>
    </div>
  </div>
</section>

<% if @user.errors.any? %>
  <div class="notification is-danger">
    <h2><%= pluralize(@user.errors.count, "error") %> prohibited this object from being saved: not successfully</h2>
    <ul>
      <% @user.errors.full_messages.each do |message| %>
      <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-6">
        <%= form_with(model: @user, local: true) do |f| %>
          <div class="field">
            <%= f.label :username, "ユーザー名", class: "label" %>
            <%= f.text_field :username, class: "input" %>
          </div>

          <div class="field">
            <%= f.label :email, "メールアドレス", class: "label" %>
            <%= f.email_field :email, class: "input" %>
          </div>

          <div class="field">
            <%= f.label :profile, "プロフィール", class: "label" %>
            <%= f.text_area :profile, class: "textarea" %>
          </div>

          <div class="field">
            <%= f.label :profile_image, "プロフィール画像", class: "label" %>
            <%= f.attachment_field :profile_image, class: "input" %>
          </div>

          <%= f.submit "更新", class: "button is-success" %>
        <% end %>
      </div>
    </div>
  </div>
</section>
```

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @recipes = Recipe.all
  end

  def show
    @recipe = Recipe.find(params[:id])
  end

  def new
    @recipe = Recipe.new
  end

  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.user_id = current_user.id
    if @recipe.save
      redirect_to recipe_path(@recipe), notice: '投稿に成功しました。' // 編集
    else
      render :new
    end
  end

  def edit
    @recipe = Recipe.find(params[:id])
    if @recipe.user != current_user
      redirect_to recipes_path, alert: '不正なアクセスです。'
    end
  end

  def update
    @recipe = Recipe.find(params[:id])
    if @recipe.update(recipe_params)
      redirect_to recipe_path(@recipe), notice: '更新に成功しました。' // 編集
    else
      render :edit
    end
  end

  def destroy
    @recipe = Recipe.find(params[:id])
    @recipe.destroy
    redirect_to recipes_path
  end

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
    if @user.id != current_user.id
      redirect_to users_path, alert: '不正なアクセスです。'
    end
  end

  def update
    @user = User.find(params[:id])
    if @user.update(user_params)
      redirect_to user_path(@user), notice: '更新に成功しました。' // 編集<br>
    else
      render :edit
    end
  end

  private
  def user_params
    params.require(:user).permit(:username, :email, :profile, :profile_image)
  end
end
```
