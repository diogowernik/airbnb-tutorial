### 09 - User Page

#### User Info Page

Add new fields to user page:

    rails g migration AddExtraFieldsToUser phone_number:string description:text
    rake db:migrate

**config/routes.rb**

add resources:

```ruby
Rails.application.routes.draw do
  
  root 'pages#home'

  devise_for 	:users, 
  						:path => '', 
  						:path_names => {:sign_in => 'login', :sign_out => 'logout', :edit => 'profile'},
  						:controllers => {:omniauth_callbacks => 'omniauth_callbacks',
  														 :registrations => 'registrations'
  														}

  resources :users, only: [:show]

end

```

#### Views

**app/views**

create **users** folder

create **show.html.erb**

```ruby
<div class="row">
	<div class="col-md-3">
		<div class="center">
			<%= image_tag avatar_url(@user), class: "avatar-full" %>
		</div>
		<div class="panel panel-default">
			<div class="panel-heading">Verification</div>	
			<div class="panel-body">
				Email Address<br>
				Phone Number
			</div>
		</div>
	</div>

	<div class="col-md-9">
		<h2><%= @user.fullname %></h2>

		<div class="description row-space-3">
			<%= @user.description %>
		</div>
	</div>
</div>
```

#### User edit page

**app/views/devise/registrations/edit.html.erb**

```ruby
<div class="row">
  <div class="col-md-3">
    <ul class="sidebar-list">
      <li class="sidebar-item"><%= link_to "Edit Profile", edit_user_registration_path, class: "sidebar-link active" %></li>
    </ul>
    <br>
    <%= link_to "View My Profile", user_path(current_user.id), class: "btn btn-default wide" %>
  </div>

  <div class="col-md-9 text-center">
    <div class="panel panel-default">
      <div class="panel-heading">Your Profile</div>  
      <div class="panel-body">
        <div class="container">
          <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
            <%= render 'shared/devisemes' %>

            <div class="form-group">
              <%= f.text_field :fullname, autofocus: true, :placeholder => "Full Name", :class => 'form-control' %>
            </div>

            <div class="form-group">
              <%= f.email_field :email, :placeholder => "Email", :class => 'form-control' %>
            </div>

            <div class="form-group">
              <%= f.text_field :phone_number, :placeholder => "Phone Number", :class => 'form-control' %>
            </div>

            <div class="form-group">
              <%= f.text_area :description, rows: 5, cols: 25,  :placeholder => "Description", :class => 'form-control' %>
            </div>

            <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
              <div>Currently waiting confirmation for: <%= resource.unconfirmed_email %></div>
            <% end %>

            <div class="form-group">
              <%= f.password_field :password, autocomplete: "off", :placeholder => "New Password (leave blank if you don't want to change it)", :class => 'form-control' %>
            </div>

            <div class="form-group">
              <%= f.password_field :password_confirmation, autocomplete: "off", :placeholder => "Confirm Password", :class => 'form-control' %>
            </div>
            
            <div class="actions">
              <%= f.submit "Save", :class => "btn btn-primary" %>
            </div>
          <% end %>
        </div>
      </div>
    </div>
  </div>
</div>  

```

#### Controllers

**app/controller**

create **users_controller.rb**

```ruby
class UsersController < ApplicationController
	def show
		@user = User.find(params[:id])
	end
end
```

create **registration_controller.rb**

```ruby
class RegistrationsController < Devise::RegistrationsController
	protected
		def update_resource(resource, params)
			resource.update_without_password(params)
		end
end
```

edit **application_controller.rb**

```ruby
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  before_action :configure_permitted_paramters, if: :devise_controller?

  protected
  	def configure_permitted_paramters
  		devise_parameter_sanitizer.for(:sign_up) << :fullname
  		devise_parameter_sanitizer.for(:account_update) << :fullname << :phone_number << :description << :email << :password
  	end
end
```
