### Notification


    rails g model Notification recipient_id:integer actor_id:integer read_at:datetime action:string notifiable_id:integer notifiable_type:string
    
    
notification.rb

    class Notification < ActiveRecord::Base
      belongs_to :recipient, class_name: "User"
      belongs_to :actor, class_name: "User"
      belongs_to :notifiable, polymorphic: true
      
      scope :unread, ->{ where(read_at: nil) }
      scope :recent, ->{ order(created_at: :desc).limit(5) }
    end
    
users.rb

    has_many :notifications, foreign_key: :recipient_id

comments_controller.rb (to notification off comments)

    @comment.save

    (@comentable.users.uniq - [current_user]).each do |user|
      Notification.create(recipient: user, actor: current_user, action: "commented", notifiable: @comment)
    end

publication.rb

    has_many :users, through: :comments
    
routes.rb


    resources :notifications do
      collection do
        post :mark_as_read
      end
    end

notifications_controller.rb

    class NotificationsController < ApplicationController
      before_action :authenticate_user!

      def index
        @notifications = Notification.where(recipient: current_user).recent
      end

      def mark_as_read
        @notifications = Notification.where(recipient: current_user).unread
        @notifications.update_all(read_at: Time.zone.now)
        render json: {success: true}
      end
    end

view/notifications/index.json.jbuilder

```
json.array! @notifications do |notification|
  json.id notification.id
  json.unread !notification.read_at?
  json.template render partial: "notifications/#{notification.notifiable_type.underscore.pluralize}/#{notification.action}", locals: {notification: notification}, formats: [:html]
```

view/notifications/users/_follow.html.erb

```
<%= link_to "#", class: "dropdown-item #{"unread" if !notification.read_at?}" do %>
  <%= notification.actor.username %>
  <%= notification.action %>ed
  you
<% end %>
```

view/notifications/comments/_commented.html.erb

```
<%= link_to commentable_path(notification.notifiable.commentable, anchor: dom_id(notification.notifiable)), class: "dropdown-item #{"unread" if !notification.read_at?}" do %>
  <%= notification.actor.username %>
  <%= notification.action %>
  a <%= notification.notifiable_type.underscore.humanize.downcase %>
<% end %>
```

app/assets/javascripts/notifications.js.coffee

```
class Notifications
  constructor: ->
    @notifications = $("[data-behavior='notifications']")

    if @notifications.length > 0
      @handleSuccess @notifications.data("notifications")
      $("[data-behavior='notifications-link']").on "click", @handleClick

      setInterval (=>
        @getNewNotifications()
      ), 5000

  getNewNotifications: ->
    $.ajax(
      url: "/notifications.json"
      dataType: "JSON"
      method: "GET"
      success: @handleSuccess
    )

  handleClick: (e) =>
    $.ajax(
      url: "/notifications/mark_as_read"
      dataType: "JSON"
      method: "POST"
      success: ->
        $("[data-behavior='unread-count']").text(0)
    )

  handleSuccess: (data) =>
    items = $.map data, (notification) ->
      notification.template

    unread_count = 0
    $.each data, (i, notification) ->
      if notification.unread
        unread_count += 1

    $("[data-behavior='unread-count']").text(unread_count)
    $("[data-behavior='notification-items']").html(items)

jQuery ->
  new Notifications
```

navbar.html.erb

    <li class="nav-item btn-group" data-behavior="notifications" data-notifications='<%= render template: "notifications/index", formats: [:json] %>'>
      <a class="dropdown-toggle nav-link" type="button" data-behavior="notifications-link" id="dropdownMenu1" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
        <%= fa_icon "bell" %> <span data-behavior="unread-count"></span>
      </a>
      <div class="dropdown-menu" aria-labelledby="dropdownMenu1" data-behavior="notification-items">

      </div>
    </li>
