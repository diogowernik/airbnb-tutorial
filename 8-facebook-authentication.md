### 7 - transactional-email

go to developers.facebook.com, create an app
With all set on developers.facebook.com set:

Check if the Gems are already installed:

    omniauth
    omniauth-facebook

Add facebook fields to User table

    rails g migration AddFieldsToUser provider:string uid:string image:string

    rake db:migrate


**config/initializers/devise.rb**

Add in the end of file

```ruby
  config.omniauth :facebook, 'app-id', 'app-secret', scope: 'email', info_fields: 'email, name'
end
```

Example:

```ruby
  config.omniauth :facebook, '1480269288956113', '3f71398af5212ccc2e84ac1a239cd572', scope: 'email', info_fields: 'email, name'
```

To login without confirm email, for few days:

search for unconfirmed_access_for

around line 110 or Ctrl + F

```ruby
  config.allow_unconfirmed_access_for = 20.days
```





**app/models/user.rb**

```ruby
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :confirmable, :omniauthable

  validates :fullname, presence: true, length: {maximum: 50}

  def self.from_omniauth(auth)
  	where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
        user.fullname = auth.info.name
        user.provider = auth.provider
        user.uid = auth.uid
        user.email = auth.info.email
        user.image = auth.info.image
        user.password = Devise.friendly_token[0,20]
      end
  end

end
```

**app/controller**

Create file: omniauth_callbacks_controller.rb

```ruby
class OmniauthCallbacksController < Devise::OmniauthCallbacksController

	def facebook
		@user = User.from_omniauth(request.env["omniauth.auth"])	

		if @user.persisted?
			sign_in_and_redirect @user, :event => :authentication
			set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
		else
			session["devise.facebook_data"] = request.env["omniauth.auth"]
			redirect_to new_user_registration_url
		end
	end

end
```

Views:

**app/views/devise/sessions/new.html.erb**
and
**app/views/devise/registrations/new.html.erb**


Add line 4:

```ruby
    <%= link_to "Sign In with Facebook", user_omniauth_authorize_path(:facebook), class: "btn btn-primary" %>
```

**config/routes.rb**

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

Now the adress is:

localhost:3000/sign_up

because was changed in the routes.rb

#### for change avatar to facebook avatar:

**app/helpers/application_helper.rb**

```ruby
module ApplicationHelper
	def avatar_url(user)
		gravatar_id = Digest::MD5::hexdigest(user.email).downcase 
		if user.image
			user.image
		else
			"https://www.gravatar.com/avatar/#{gravatar_id}.jpg?d=identical&s=150"
		end
	end
end
```
