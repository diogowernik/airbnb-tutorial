Comandos e dicas AirAlien

    rails new AirAlien
    cd AirAlien
    bundle install
    
Gemfile

Delete comments

    gem 'bootstrap-sass', '~> 3.3.5'
    gem 'sass-rails', '>= 3.2'
    gem 'devise'
    gem 'omniauth'
    gem 'omniauth-facebook'
    gem 'toastr-rails'
    
Gem opcional para testes (por minha conta)

    group :development, :test do
      gem 'byebug'
      gem 'rspec-rails', '~> 3.0'
    end

Gerar rpec (por minha conta)

    rails generate rspec:install

Linha de commando:

    bundle install

Instalar a gem devise e gerar estrutura de Usuários

    rails g devise:install
    rails g devise User

Criar estrutura banco de dados

    rake db:migrate
    
    rails g devise:views
    
Editar manualmente:

config/environments/development.rb

    config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

app/views/layouts/application.html.erb.

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>   
       
Gerar views:

    rails g devise:views

Rodar servidor:

    rails s

Parar servidor - Ctrl + C

Gerar pagina pages/home

    rails g controller Pages home

Menu

    rails g migration AddFullnameToUser fullname:string
    rake db:migrate







