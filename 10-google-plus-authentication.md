### 10 - Google Plus Authentication


Step 1: Create Google app

- Create a new Google app, go to: https://console.developers.google.com
- Click on "Create Project" => Name it to whatever you want.
- On the left menu, click APIs => Enable "Contact API" and "Google+ API"
- On the left menu, click Credentials => Add credentials => OAuth 2.0 client ID => Web application => Set "Authorized redirect URIs" to "http://localhost:3000/auth/google_oauth2/callback"
- It will create "Client ID" and "Client secret" for you. We will be using these values later.

Step 2: Add new gem to our Gemfile

**Gemfile**

    gem 'omniauth-google-oauth2'

Step 3: Create new method in OmniautCallbacksController

**app/controller/omniauth_callbacks_controller.rb**

```ruby
def google_oauth2
  @user = User.from_omniauth(request.env["omniauth.auth"])    

  if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication
      set_flash_message(:notice, :success, :kind => "Google") if is_navigational_format?
  else
      session["devise.google_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
  end
end
```

Step 4: Config API keys for Google app

**config/initializers/devise.rb**

Add in the end of file

```ruby
config.omniauth :google_oauth2, 'YOUR_GOOGLE_APP_CLIENT_ID', 'YOUR_GOOGLE_APP_CLIENT_SECRET'
```

maybe try

```ruby
config.omniauth :google_oauth2, 'YOUR_GOOGLE_APP_CLIENT_ID', 'YOUR_GOOGLE_APP_CLIENT_SECRET', scope: 'email', info_fields: 'email, name'
```

Step 5: Add new link to sign in with Google

Views:

**app/views/devise/sessions/new.html.erb**
and
**app/views/devise/registrations/new.html.erb**


```ruby
<%= link_to "Sign In with Google", user_omniauth_authorize_path(:google_oauth2), class: "btn btn-primary" %>
```
