## Recipegram App Part2

+ `src/Gemfile`を編集<br>

```
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.5'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails', branch: 'main'
gem 'rails', '~> 6.1.4', '>= 6.1.4.1'
# Use mysql as the database for Active Record
gem 'mysql2', '~> 0.5'
# Use Puma as the app server
gem 'puma', '~> 5.0'
# Use SCSS for stylesheets
gem 'sass-rails', '>= 6'
# Transpile app-like JavaScript. Read more: https://github.com/rails/webpacker
gem 'webpacker', '~> 5.0'
# Turbolinks makes navigating your web application faster. Read more: https://github.com/turbolinks/turbolinks
gem 'turbolinks', '~> 5'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.4', require: false

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :development do
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 4.1.0'
  # Display performance information such as SQL time and flame graphs for each request in your browser.
  # Can be configured to work on production as well see: https://github.com/MiniProfiler/rack-mini-profiler/blob/master/README.md
  gem 'rack-mini-profiler', '~> 2.0'
  gem 'listen', '~> 3.3'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
end

group :test do
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '>= 3.26'
  gem 'selenium-webdriver'
  # Easy installation and use of web drivers to run system tests with browsers
  gem 'webdrivers'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

gem 'devise' // 追記
gem "refile", require: "refile/rails", github: 'manfe/refile' // 追記
gem "refile-mini_magick" // 追記
gem "bulma-rails" // 追記
```

+ `$ bundle install`を実行<br>

+ `src/app/assets/stylesheets/css`を`src/app/assets/stylesheets/scss`に変更する<br>

+ ``src/app/assets/stylesheets/scss`を編集<br>

```
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, or any plugin's
 * vendor/assets/stylesheets directory can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the bottom of the
 * compiled file so the styles you add here take precedence over styles defined in any other CSS/SCSS
 * files in this directory. Styles in this file should be added after the last require_* statement.
 * It is generally better to create a new file per style scope.
 *
 *= require_tree .
 *= require_self
 */

@import "bulma"; // 追記
```

## deveseの導入

+ `$ rails generate devise:install`を実行<br>

+ `src/config/enviroments/development.rb`を編集<br>

```
require "active_support/core_ext/integer/time"

Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.

  # In the development environment your application's code is reloaded any time
  # it changes. This slows down response time but is perfect for development
  # since you don't have to restart the web server when you make code changes.
  config.cache_classes = false

  # Do not eager load code on boot.
  config.eager_load = false

  # Show full error reports.
  config.consider_all_requests_local = true

  # Enable/disable caching. By default caching is disabled.
  # Run rails dev:cache to toggle caching.
  if Rails.root.join('tmp', 'caching-dev.txt').exist?
    config.action_controller.perform_caching = true
    config.action_controller.enable_fragment_cache_logging = true

    config.cache_store = :memory_store
    config.public_file_server.headers = {
      'Cache-Control' => "public, max-age=#{2.days.to_i}"
    }
  else
    config.action_controller.perform_caching = false

    config.cache_store = :null_store
  end

  # Store uploaded files on the local file system (see config/storage.yml for options).
  config.active_storage.service = :local

  # Don't care if the mailer can't send.
  config.action_mailer.raise_delivery_errors = false

  config.action_mailer.perform_caching = false

  # Print deprecation notices to the Rails logger.
  config.active_support.deprecation = :log

  # Raise exceptions for disallowed deprecations.
  config.active_support.disallowed_deprecation = :raise

  # Tell Active Support which deprecation messages to disallow.
  config.active_support.disallowed_deprecation_warnings = []

  # Raise an error on page load if there are pending migrations.
  config.active_record.migration_error = :page_load

  # Highlight code that triggered database queries in logs.
  config.active_record.verbose_query_logs = true

  # Debug mode disables concatenation and preprocessing of assets.
  # This option may cause significant delays in view rendering with a large
  # number of complex assets.
  config.assets.debug = true

  # Suppress logger output for asset requests.
  config.assets.quiet = true

  # Raises error for missing translations.
  # config.i18n.raise_on_missing_translations = true

  # Annotate rendered view with file names.
  # config.action_view.annotate_rendered_view_with_filenames = true

  # Use an evented file watcher to asynchronously detect changes in source code,
  # routes, locales, etc. This feature depends on the listen gem.
  config.file_watcher = ActiveSupport::EventedFileUpdateChecker
  config.action_mailer.default_url_options = { host: 'localhost', port: 3000 } // 追記

  # Uncomment if you wish to allow Action Cable access from any origin.
  # config.action_cable.disable_request_forgery_protection = true
end
```

+ `src/config/routes.rb`を編集<br>

```
Rails.application.routes.draw do
  root to: "home#index"
end
```

+ `src/app/views/layouts/application.html.erb`を編集<br>

```
<!DOCTYPE html>
<html>
  <head>
    <title>App</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

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

    <%= yield %>
  </body>
</html>
```

+ `$ rails g devise:views`を実行<br>

+ `$ rails generate devise user`を実行<br>

+ `src/db/migrate/xxxxx_devise_create_users.rb`を編集<br>

```
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[6.1]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # t.integer  :sign_in_count, default: 0, null: false
      # t.datetime :current_sign_in_at
      # t.datetime :last_sign_in_at
      # t.string   :current_sign_in_ip
      # t.string   :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at

      t.string :username //
      t.text :profile //
      t.string :profile_image_id //
      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

+ `$ rails db:migrate`を実行<br>

## Registration view 14:24〜

`app/views/devise/registratiosns/new.html.erb`を編集<br>

```
<section class="hero is-info">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        新規登録画面
      </h1>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-5">
        <%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
        <%= render "devise/shared/error_messages", resource: resource %>


        <div class="field">
          <%= f.label :username, class: "label" %>
          <p class="control has-icons-left">
            <%= f.text_field :username, autofocus: true, autocomplete: "email", :placeholder => "username", class: "input" %>
            <span class="icon is-small is-left">
              <i class="fas fa-user"></i>
            </span>
          </p>
        </div>

        <div class="field">
          <%= f.label :email, class: "label" %>
          <p class="control has-icons-left">
            <%= f.email_field :email, autofocus: true, autocomplete: "email", :placeholder => "email", class: "input" %>
            <span class="icon is-small is-left">
              <i class="fas fa-envelope"></i>
            </span>
          </p>
        </div>

        <div class="field">
          <%= f.label :password, class: "label" %>
          <% if @minimum_password_length %>
          <em>(<%= @minimum_password_length %> characters minimum)</em>
          <% end %>
          <p class="control has-icons-left">
          <%= f.password_field :password, autocomplete: "new-password", :placeholder => "password", class: "input" %>
            <span class="icon is-small is-left">
              <i class="fas fa-lock"></i>
            </span>
          </p>
        </div>

        <div class="field">
          <%= f.label :password_confirmation, class: "label" %>
          <p class="control has-icons-left">
            <%= f.password_field :password_confirmation, autocomplete: "new-password", :placeholder => "password confirmation", class: "input" %>
            <span class="icon is-small is-left">
              <i class="fas fa-lock"></i>
            </span>
          </p>
        </div>

        <div class="actions">
          <%= f.submit "Sign up", class: "button is-success" %>
        </div>
        <% end %>

        <%= render "devise/shared/links" %>
      </div>
    </div>
  </div>
</section>
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

    <%= yield %>
  </body>
</html>
```

## Login画面の編集

+ `app/views/devise/sessions/new.html.erb`を編集

```
<section class="hero is-info">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        ログイン画面
      </h1>
    </div>
  </div>
</section>

<section class="section">
  <div class="container">
    <div class="columns is-centered">
      <div class="column is-5">
        <%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
        <div class="field">
          <%= f.label :email, class: "label" %>
          <p class="control has-icons-left">
            <%= f.email_field :email, autofocus: true, autocomplete: "email", :placeholder => "email", class: "input" %>
            <span class="icon is-small is-left">
              <i class="fas fa-envelope"></i>
            </span>
          </p>
        </div>

        <div class="field">
          <%= f.label :password, class: "label" %>
          <p class="control has-icons-left">
          <%= f.password_field :password, autocomplete: "current-password", :placeholder => "password", class: "input" %>
            <span class="icon is-small is-left">
              <i class="fas fa-lock"></i>
            </span>
          </p>
        </div>

        <div class="actions">
          <%= f.submit "Log in", class: "button is-success" %>
        </div>
        <% end %>

        <%= render "devise/shared/links" %>
      </div>
    </div>
  </div>
</section>
```

+ `app/controllers/application_controller.rb`を編集<br>

```
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?
  private
  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:username])
  end
end
```

+ `$ rails g controller home index`を実行<br>

+ `config/routes.rb`を編集<br>

```
Rails.application.routes.draw do
  get 'home/index' // 削除する
  devise_for :users
  root to: "home#index"
end
```

+ ブラウザでrootの確認方法 `$ localhost:3000/rails/info/routes`を実行<br>

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

+ `app/assets/stylesheets/application.scss`を編集<br>

```
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, or any plugin's
 * vendor/assets/stylesheets directory can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the bottom of the
 * compiled file so the styles you add here take precedence over styles defined in any other CSS/SCSS
 * files in this directory. Styles in this file should be added after the last require_* statement.
 * It is generally better to create a new file per style scope.
 *
 *= require_tree .
 *= require_self
 */

@import 'bulma';

.notification:not(:last-child) {
  margin-bottom: 0;
}

.profile_image {
  border-radius: 50% !important;
}

.main-image {
  height: 940px;
  background-image: url(meal.png);
  background-repeat: no-repeat;
  background-size: contain;
}
```