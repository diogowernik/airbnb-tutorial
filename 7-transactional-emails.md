### 7 - transactional-email

Using Mandrill - go to mandrill.com , create an api key and config your account.

**config/enviromments/development.rb**

find and change to true

```ruby
config.action_mailer.raise_delivery_erros = true
```
add

in the end of file add:

```ruby
config.action_mailer.raise_delivery_erros = true

  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = {
    address: 'smtp.mandrillapp.com',
    port: 587,
    enable_statrttls_auto: true,
    user_name: 'your-email@gmail.com',
    password: 'api key from mandrill.com',
    authentication: 'login'
  }
```
**config/initializers/devise.rb**

find and change:

around line 15

```ruby
config.mailer_sender = 'Your Name @ Company <no-reply@yoursite.com>'
```

around line 124

```ruby
config.reconfirmable = false
```

**app/models/users.rb**

check if is on:

```ruby
:confirmable
```

On terminal

```
rails g migration AddConfirmableToDevise
```

**db/migrate/_add__confirmable_to_devise.rb**

check if is on:

```ruby
class AddConfirmableToDevise < ActiveRecord::Migration
  def up
  	add_column :users, :confirmation_token, :string
  	add_column :users, :confirmed_at, :datetime
  	add_column :users, :confirmation_sent_at, :datetime

 		add_index :users, :confirmation_token, unique: true
  end
  def down
  	remove_column :users, :confirmation_token, :confirmed_at, :confirmation_sent_at
  end
end
```

for the messages:

**app/views/devise/mailer/confimation_instructions.html.erb**

```ruby
<p>Welcome <%= @email %>!</p>

<p>You can confirm your account email through the link below:</p>

<p><%= link_to 'Confirm my account', confirmation_url(@resource, confirmation_token: @token) %></p>
```

