### 4 - Add Fullname to User

    g migration AddFullnameToUser
    rake db:migrate

**app/model/user.rb**

```ruby
  validates :fullname, presence: true, length: {maximum:50}
```

**app/controllers/application_controller.rb**

```ruby
  before_action :configure_permitted_paramters, if: :devise_controller?

  protected
  	def configure_permitted_paramters
  		devise_parameter_sanitizer.for(:sign_up) << :fullname
  		devise_parameter_sanitizer.for(:account_update) << :fullname
  	end
```