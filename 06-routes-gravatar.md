6 - Routes and gravatar

**config/routes.rb**

```ruby
root 'pages#home'
```

**app/helpers/application_helper.rb**


```ruby
def avatar_url(user)
	gravatar_id = Digest::MD5::hexdigest(user.email).downcase 
	if user.image
		user.image
	else
		"https://www.gravatar.com/avatar/#{gravatar_id}.jpg?d=identical&s=150"
	end
end
```

**app/views/shared/_navbar.html.erb**

```ruby
<%= image_tag avatar_url(current_user), class: "img-circle avatar-small" %>&nbsp;
<%= current_user.fullname %>
```

**app/assets/stylesheets/application.scss**

```css
.avatar-small {
  width: 28px;
}
.avatar-medium {
  width: 48px;
}
.avatar-large {
  width: 68px;
}
.avatar-full {
  width: 100%;
}
```

