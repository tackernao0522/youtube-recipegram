## Recipegram part3 Recepi Function

+ `$ rails g model recipe user_id:integer title:string body:text image_id:string`を実行<br>

+ `$ rails db:migrate`を実行<br>

+ `$ rails g controller recipes index show new edit`を実行<br>

+ `config/reoutes.rb`を編集<br>

```
Rails.application.routes.draw do
  devise_for :users
  root to: "home#index"
  resources :users
  resources :recipes
end
```

+ `app/models/recipe.rb`を編集<br>

```
class Recipe < ApplicationRecord
  belongs_to :user
  attachment :image
end
```

+ `app/models/user.rb`を編集<br>

```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  attachment :profile_image
  has_many :recipes, dependent: :destroy // userが削除されたらrecipeも一緒に削除される
end
```

+ `app/views/recipes/new.html.erb`を編集<br>

```
<h1>Recipes#new</h1>
<p>Find me in app/views/recipes/new.html.erb</p>

<%= form_with(model: @recipe, local: true) do |f| %>
  <%= f.label :title %>
  <%= f.text_field :title %>

  <%= f.label :body %>
  <%= f.text_area :body %>

  <%= f.label :image %>
  <%= f.attachment_field :image %>

  <%= f.submit %>
<% end %>
```

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  def index
  end

  def show
  end

  def new
    @recipe = Recipe.new
  end

  def edit
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

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  def index
  end

  def show
    @recipe = Recipe.find(params[:id])
  end

  def new
    @recipe = Recipe.new
  end

  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.user_id = current_user.id // 誰が投稿したのかわかるようにする
    @recipe.save
    redirect_to recipe_path(@recipe)
  end

  def edit
  end

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `app/views/recipes/show.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        レシピ詳細
      </h1>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-7">
        <div class="card">
          <div class="card-image">
            <figure class="image is-4by3">
              <%= attachment_image_tag @recipe, :image %>
            </figure>
          </div>
          <div class="card-content">
            <div class="media">
              <div class="media-content">
                <p class="title is-4"><%= @recipe.title %></p>
              </div>
            </div>
            <div class="content">
              <table class="table is-narrow">
                <tr>
                  <th>レシピ</th>
                </tr>
                <tr>
                  <td><%= simple_format @recipe.body %></td>
                </tr>
              </table>
              <% if @recipe.user.id == current_user.id %>
                <%= link_to "編集画面へ", edit_recipe_path(@recipe), class: "button is-success" %>
              <% end %>
            </div>
          </div>
        </div>
      </div>

      <div class="column is-4">
        <article class="panel is-link">
          <p class="panel-heading">
            By <%= @recipe.user.username %>
          </p>
          <div class="panel-block">
            <p class="control">
              <%= @recipe.user.profile %>
            </p>
          </div>
          <%= link_to user_path(@recipe.user), class: "panel-block" do %>
            <span class="panel-icon">
              <i class="fas fa-user" aria-hidden="true"></i>
            </span>
            <%= @recipe.user.username %> さんのページへ
          <% end %>
        </article>
      </div>
    </div>
  </div>
</section>
```

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
  def index
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

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
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

+ `app/controllers/recipes_controller.erb`を編集<br>

```
class RecipesController < ApplicationController
  def index
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

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `app/controllers/recipes_controller.rb`を再編集<br>

```
class RecipesController < ApplicationController
  def index
    @recipes = Recipe.all // 追記
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

  private
  def recipe_params
    params.require(:recipe).permit(:title, :body, :image)
  end
end
```

+ `config/application.rb`を編集<br>

```
require_relative "boot"

require "rails/all"

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module App
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.1

    # Configuration for the application, engines, and railties goes here.
    #
    # These settings can be overridden in specific environments using the files
    # in config/environments, which are processed later.
    #
    # config.time_zone = "Central Time (US & Canada)"
    # config.eager_load_paths << Rails.root.join("extras")
    config.time_zone = 'Tokyo' // 追記
  end
end
```

+ `app/views/recipes/index.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        レシピ一覧
      </h1>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-multiline">
      <% @recipes.each do |recipe| %>
        <div class="column is-4">
          <div class="card">
            <div class="card-image">
              <figure class="image is-4by3">
                <%= link_to recipe_path(recipe) do %>
                  <%= attachment_image_tag recipe, :image %>
                <% end %>
              </figure>
            </div>
            <div class="card-content">
              <div class="media">
                <div class="media-left">
                  <figure class="image is-48x48">
                    <%= link_to user_path(recipe.user) do %>
                      <%= attachment_image_tag recipe.user, :profile_image, :fill, 100, 100, fallback: "no-image.png", class: "profile_image" %>
                    <% end %>
                  </figure>
                </div>
                <div class="media-content">
                  <div class="title"><%=link_to recipe.user.username, user_path(recipe.user) %></div>
                </div>
              </div>

              <div class="content">
                <time><%= recipe.updated_at.strftime("%Y-%m-%d %H:%M") %></time>更新
              </div>
            </div>
          </div>
        </div>
      <% end %>
    </div>
  </div>
</section>
```

+ `app/views/layouts/application.html.erb`を編集<br>

```
<!DOCTYPE html>
<html>
  <head>
    <title>RecipegramDemo</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.9.0/css/all.min.css" integrity="sha256-UzFD2WYH2U1dQpKDjjZK72VtPeWP50NoJjd26rnAdUI=" crossorigin="anonymous" /> // 追記
    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <% if flash[:notice] %>
      <div class="notification is-info">
        <p class="notice"><%= notice %></p>
      </div>
    <% end %>

    <% if flash[:alert] %>
      <div class="notification is-danger">
        <p class="alert"><%= alert %></p>
      </div>
    <% end %>

    <nav class="navbar is-warning">
      <div class="navbar-brand">
        <%= link_to root_path, class: "navbar-item" do %>
          <h1 class="title is-4" style="font-family: cursive;">recipegram</h1>
        <% end %>
        <div class="navbar-burger burger" data-target="navbarExampleTransparentExample">
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>

      <% if user_signed_in? %>
        <div id="navbarExampleTransparentExample" class="navbar-menu">
          <div class="navbar-item">
            <%= link_to "レシピ一覧", recipes_path, class: "navbar-item button is-warning is-light" %>
          </div>
          <div class="navbar-item">
            <%= link_to "レシピ投稿", new_recipe_path, class: "navbar-item button is-warning is-light" %>
          </div>

          <div class="navbar-end">
            <div class="navbar-item">
              <div class="field is-grouped">
                <p class="control">
                  <%= link_to "レシピグラマー一覧", users_path, class: " button is-warning is-light" %>
                </p>
                <p class="control">
                  <%= link_to "マイページ", user_path(current_user), class: "button is-warning is-light" %>
                </p>
                <p class="control">
                  <%= link_to "ログアウト", destroy_user_session_path, method: :delete, class: "button is-warning is-light" %>
                </p>
              </div>
            </div>
          </div>
        </div>
      <% else %>
        <div id="navbarExampleTransparentExample" class="navbar-menu">
          <div class="navbar-item">
            <%= link_to "レシピ一覧", recipes_path, class: "navbar-item button is-warning is-light" %>
          </div>
          <div class="navbar-item">
            <%= link_to "レシピグラマー一覧", users_path, class: "navbar-item button is-warning is-light" %>
          </div>
          <div class="navbar-end">
            <div class="navbar-item">
              <div class="field is-grouped">
                <p class="control">
                  <%= link_to "ログイン", new_user_session_path, class: "button is-warning is-light" %>
                </p>
                <p class="control">
                  <%= link_to "新規登録", new_user_registration_path, class: "button is-warning is-light" %>
                </p>
              </div>
            </div>
          </div>
        </div>
      <% end %>
    </nav>
    <%= yield %>
  </body>
</html>
```

+ `app/controllers/recipes_controller.rb`を編集<br>

```
class RecipesController < ApplicationController
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

+ `app/views/users/show.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        マイページ
      </h1>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-8">
        <div class="columns is-centered">
          <div class="column is-4">
            <figure class="image is-128x128"  style="margin-left: auto; margin-right: auto;">
              <%= attachment_image_tag @user, :profile_image, :fill, 100, 100, fallback: "no-image.png", class: "profile_image is-rounded" %>
            </figure>
          </div>
          <div class="column is-8">
            <table class="table is-fullwidth">
              <tr>
                <td class="is-size-4">
                  <strong><%= @user.username %></strong>
                </td>
                <% if @user.id == current_user.id %>
                <td class="is-size-4">
                  <%= @user.email %>
                </td>
                <td>
                    <%= link_to "編集", edit_user_path(@user), class: "button is-primary" %>
                </td>
                <% end %>
              </tr>
            </table>
            <table class="table is-fullwidth">
              <tr>
                <td>
                  <%= @user.profile %>
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-multiline">
      <% @user.recipes.each do |recipe| %>
        <div class="column is-4">
          <div class="card">
            <div class="card-image">
              <figure class="image is-4by3">
                <%= link_to recipe_path(recipe) do %>
                  <%= attachment_image_tag recipe, :image, fallback: "no-image.png" %>
                <% end %>
              </figure>
            </div>
            <div class="card-content">
              <div class="media">
                <div class="media-content">
                  <%= link_to recipe_path(recipe), class: "panel-block" do %>
                    <span class="panel-icon">
                      <i class="fas fa-book" aria-hidden="true"></i>
                    </span>
                    このレシピを見る
                  <% end %>
                  <% if @user.id == current_user.id %>
                    <%= link_to edit_recipe_path(recipe), class: "panel-block" do %>
                      <span class="panel-icon">
                        <i class="fas fa-edit"></i>
                      </span>
                      このレシピを編集する
                    <% end %>
                    <%= link_to recipe_path(recipe), method: :delete, data: {confirm: "削除しますか？"}, class: "panel-block" do %>
                      <span class="panel-icon">
                        <i class="fas fa-trash"></i>
                      </span>
                      このレシピを削除する
                    <% end %>
                  <% end %>
                </div>
              </div>
              <div class="content">
                <time><%= recipe.updated_at.strftime("%Y-%m-%d %H:%M") %></time>更新
              </div>
            </div>
          </div>
        </div>
      <% end %>
    </div>
  </div>
</section>
```

+ `app/views/users/index.html.erb`を編集<br>

```
<section class="hero is-success">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        レシピグラマー一覧
      </h1>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-multiline">
      <% @users.each do |user| %>
        <div class="column is-3">
          <div class="card">
            <div class="card-content">
              <div class="card-image">
                <figure class="image">
                  <%= link_to user_path(user) do %>
                      <%= attachment_image_tag user, :profile_image, :fill, 100, 100, fallback: "no-image.png" %>
                  <% end %>
                </figure>
              </div>
            </div>
            <div class="card-content">
              <div class="media">
                <div class="media-content">
                  <div class="title"><%= link_to user.username, user_path(user) %></div>
                </div>
              </div>
              <div class="content">
                投稿数：<%= user.recipes.count %> // 編集
              </div>
            </div>
          </div>
        </div>
      <% end %>
    </div>
  </div>
</section>
```
