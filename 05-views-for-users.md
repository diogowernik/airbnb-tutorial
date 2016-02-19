### 5 - Vies for users


#### Login Page

**app/views/devise/session/new.html.erb**

```ruby
<div class="row">
  <div class="col-md-6 col-md-offset-3 center">
   
    <h2>Log In</h2>

    <%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
      <%= render 'shared/devisemes' %>

      <div class="form-group">        
        <%= f.email_field :email, :class => 'form-control', :placeholder => 'Email' %>
      </div>

      <div class="form-group">
        <%= f.password_field :password, :class => 'form-control', :placeholder => 'Password',  autocomplete: "off" %>
      </div>

      <% if devise_mapping.rememberable? %>
        <div class="form-group">
          <%= f.check_box :remember_me %> Remember Me
        </div>
      <% end %>

      <div class="actions">
        <%= f.submit "Log In", :class => "btn btn-primary" %>
        <%= link_to "Forgot Password", new_user_password_path, :class => "btn btn-primary" %>
      </div>
    <% end %>

  </div>
</div>

```

**app/views/devise/registratios/new.html.erb**

```ruby
<div class="row">
  <div class="col-md-6 col-md-offset-3 text-center">
    
    <h2>Sign up</h2>

    <%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
      <%= render 'shared/devisemes' %>

      <div class="form-group">        
        <%= f.text_field :fullname, :class => 'form-control', :placeholder => 'Full Name', autofocus: true %>
      </div>

      <div class="form-group">        
        <%= f.email_field :email, :class => 'form-control', :placeholder => 'Email' %>
      </div>

      <div class="form-group">        
        <% if @validatable %>
          <em>(<%= @minimum_password_length %> characters minimum)</em>
        <% end %><br />
        <%= f.password_field :password, :class => 'form-control', :placeholder => 'Password',  autocomplete: "off" %>
      </div>

      <div class="actions">
        <%= f.submit "Sign up", :class => "btn btn-primary" %>
      </div>
    <% end %>

  </div>
</div>
```

**app/views/devise/passwords/new.html.erb**

```ruby
<div class="row">
  <div class="col-md-6 col-md-offset-3 text-center">

	<h2>Forgot your password?</h2>

	<%= form_for(resource, as: resource_name, url: password_path(resource_name), html: { method: :post }) do |f| %>
	  <%= render 'shared/devisemes' %>

	  <div class="form-group">
	    <%= f.email_field :email, autofocus: true, :class => "form-control", :placeholder => "Email" %>
	  </div>

	  <%= f.submit "Send me reset password instructions", :class => "btn btn-danger" %>
	<% end %>

  </div>
</div>
```

**app/views/devise/passwords/edit.html.erb**

```ruby
<div class="row">
  <div class="col-md-6 col-md-offset-3 text-center">

  <h2>Change your password</h2>

  <%= form_for(resource, as: resource_name, url: password_path(resource_name), html: { method: :put }) do |f| %>
    <%= render 'shared/devisemes' %>
    <%= f.hidden_field :reset_password_token %>

    <div class="form-group">
      <%= f.password_field :password, autofocus: true, autocomplete: "off", :class => "form-control", :placeholder => "New Password" %>
    </div>

    <div class="form-group">
      <%= f.password_field :password_confirmation, autocomplete: "off", :class => "form-control", :placeholder => "Confirm New Password" %>
    </div>

    <%= f.submit "Change my password", :class => "btn btn-danger" %>
  <% end %>

  </div>
</div>
```

**app/views/devise/registratios/edit.html.erb**

```ruby
<div class="row">
  <div class="col-md-6 col-md-offset-3 text-center">
  
    <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
      <%= render 'shared/devisemes' %>

      <div class="form-group">
        <%= f.text_field :fullname, autofocus: true, :placeholder => "Full Name", :class => 'form-control' %>
      </div>

      <div class="form-group">
        <%= f.email_field :email, :placeholder => "Email", :class => 'form-control' %>
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

      <div class="form-group">
        <%= f.password_field :current_password, autocomplete: "off", :placeholder => "Type current password to confirm your change", :class => 'form-control' %>
      </div>
      
      <div class="actions">
        <%= f.submit "Save", :class => "btn btn-primary" %>
      </div>
    <% end %>

  </div>
</div>  
```

**app/views/shared/_navbar.html.erb**

```ruby
<%= current_user.email %>
```

change

```ruby
<%= current_user.fullname %>
```