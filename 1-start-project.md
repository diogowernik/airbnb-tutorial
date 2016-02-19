## 1 - Start project and set bootstrap

    rails new AirAlien
    cd AirAlien

Edit Gemfile

Delete comments and add this gems:

    gem 'bootstrap-sass', '~> 3.3.5'
    gem 'sass-rails', '>= 3.2'
    gem 'devise'
    gem 'omniauth'
    gem 'omniauth-facebook'
    gem 'toastr-rails'

Delete the gem:

    gem 'sass-rails', '~> 5.0'

Command Line

    bundle install


### Bootstrap

**app/assets/stylesheets/application.css**

rename to **application.scss** and add

    @import "bootstrap-sprockets";
    @import "bootstrap";
    @import "toastr"

**app/assets/javascripts/application.js**

after //= require jquery

    //= require bootstrap-sprockets

after //= require jquery_ujs

    //= require toastr

start server

    rails s
    Ctrl + C (to stop the server)