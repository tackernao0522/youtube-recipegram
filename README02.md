## 33:33〜 ユーザー一覧、 ユーザー詳細、 ユーザー編集画面の作成<br>

+ `$ rails g controller users index show edit`を実行<br>

+ `src/config/routes.rb`を編集<br>

```
Rails.application.routes.draw do
  devise_for :users
  root to: "home#index"
  resources :users
end
```

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  def index
    @users = User.all
  end

  def show
  end

  def edit
  end
end
```

+ `app/views/users/index.html`を編集<br>

```
<h1>Users#index</h1>
<p>Find me in app/views/users/index.html.erb</p>

<% @users.each do |user| %>
  <%= user.username %>
<% end %>
```

+ `Dockerfile`を編集してイメージ作り直す<br>

```
FROM ruby:2.7

ENV RAILS_ENV=production

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
  && apt-get update -qq \
  && apt-get -y install imagemagick libmagick++-dev \ // 追記
  && apt-get install -y nodejs yarn \
  && apt-get install -y vim
WORKDIR /app
COPY ./src /app
RUN bundle config --local set path 'vendor/bundle' \
  && bundle install
```

+ `app/models/user.rb`を編集<br>

```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  attachment :profile_image // 追記
end
```

+ `app/views/users/index.html.erb`を編集<br>

```
<h1>Users#index</h1>
<p>Find me in app/views/users/index.html.erb</p>

<% @users.each do |user| %>
<%= attachment_image_tag user, :profile_image, fallback: "no-image.png" %>
  <%= user.username %>
<% end %>
```

+ `app/assets/images/ディレクトリにno-image.pngファイルを入れておく`<br>

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
                投稿数：
              </div>
            </div>
          </div>
        </div>
      <% end %>
    </div>
  </div>
</section>
```

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
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
                    <%= link_to '編集', edit_user_path(@user), class: "button is-primary" %>
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
```

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
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

+ `app/controllers/users_controller.rb`を編集<br>

```
class UsersController < ApplicationController
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

+ `config/initializers/application_controller_renderer.rb`を編集<br>

```
# Be sure to restart your server when you modify this file.

# ActiveSupport::Reloader.to_prepare do
#   ApplicationController.renderer.defaults.merge!(
#     http_host: 'example.org',
#     https: false
#   )
# end

Refile.secret_key = 'a2c5b78179478581e1cb0b70b8948705e545aec1a52629c48a32c8c51f24aba7f2befdc6dedcf511505a6e22337ac9999c27977dc8e1d57e27d40aecb0649efe'
```

+ `app/views/layouts/application.html.erb`を編集<br>

```
<!DOCTYPE html>
<html>
  <head>
    <title>App</title>
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
          <div class="navbar-end">
            <div class="navbar-item">
              <div class="field is-grouped">
                <p class="control">
                  <%= link_to "レシピグラマー一覧", users_path, class: "button is-warning is-light" %>
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
          <div class="navbar-end">
            <div class="navbar-item">
              <div class="field is-grouped">
                <p class="control">
                  <%= link_to "レシピグラマー一覧", users_path, class: "button is-warning is-light" %>
                </p>
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
