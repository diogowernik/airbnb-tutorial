### 2 - basic authentication

Install and generate with devise.

    rails g devise:install
    rails g devise User
    rake db:migrate
    rails g devise:views

**config/environments/development.rb**

In production, :host should be set to the actual host of your application.

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

**app/views/layouts/application.html.erb**

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

git on project


