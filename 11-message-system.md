### 11 - Message System


1) Create Models

    rails g model Conversation sender_id:integer recipient_id:integer
    rails g model Message content:text conversation:references user:references

rake

    rake db:migrate

**models/conversation.rb**

```ruby
class Conversation < ActiveRecord::Base
	belongs_to :sender, foreign_key: :sender_id, class_name: 'User'
	belongs_to :recipient, foreign_key: :recipient_id, class_name: 'User'

	has_many :messages, dependent: :destroy

	validates_uniqueness_of :sender_id, scope: :recipient_id

	scope :involving, -> (user) do
		where("conversations.sender_id = ? OR conversations.recipient_id = ?", user.id, user.id)
	end

	scope :between, -> (sender_id, recipient_id) do
		where("(conversations.sender_id = ? AND conversations.recipient_id = ?) OR (conversations.sender_id = ? AND conversations.recipient_id = ?)", 
					sender_id, recipient_id, recipient_id, sender_id)
	end

end
```

**models/message.rb**

```ruby
class Message < ActiveRecord::Base
  belongs_to :conversation
  belongs_to :user

  validates_presence_of :content, :conversation_id, :user_id

  def message_time
  	created_at.strftime("%v")
  end

end
```

2) Create Controller

**controllers/conversations_controller.rb**

Create file: **conversations_controller.rb**

```ruby
class ConversationsController < ApplicationController
	before_action :authenticate_user!

	def index
		@users = User.all
		@conversations = Conversation.involving(current_user)
	end

	def create
		if Conversation.between(params[:sender_id], params[:recipient_id]).present?
			@conversation = Conversation.between(params[:sender_id], params[:recipient_id]).first
		else
			@conversation = Conversation.create(conversation_params)
		end

		redirect_to conversation_messages_path(@conversation)
	end

	private

		def conversation_params
			params.permit(:sender_id, :recipient_id)
		end

end
```

**controllers/messages_controller.rb**

Create file: **messages_controller.rb**

```ruby
class MessagesController < ApplicationController
	before_action :authenticate_user!
	before_action :set_conversation

	def index
		if current_user == @conversation.sender || current_user == @conversation.recipient
			@other = current_user == @conversation.sender ? @conversation.recipient : @conversation.sender
			@messages = @conversation.messages.order("created_at DESC")
		else
			redirect_to conversations_path, alert: "You don't have permission to view this."
		end
	end

	def create
		@message = @conversation.messages.new(message_params)
		@messages = @conversation.messages.order("created_at DESC")

		if @message.save
			respond_to do |format|
				format.js
			end
		end
	end

	private

		def set_conversation
			@conversation = Conversation.find(params[:conversation_id])
		end

		def message_params
			params.require(:message).permit(:content, :user_id)
		end
end
```

**config/routes.rb**

```ruby
  resources :conversations, only: [:index, :create] do
    resources :messages, only: [:index, :create]
  end

```

3) Views

**views/conversations**

Create folder **conversations** and file **index.html.erb**

```ruby
<div class="row">
	<div class="col-md-12">
		
		<div class="panel panel-default">
			<div class="panel-heading">Your conversations</div>
			<div class="panel-body">
				<div class="container">
					<% @conversations.each do |conversation| %>
						<% other = conversation.sender == current_user ? conversation.recipient : conversation.sender %>

						<%= link_to conversation_messages_path(conversation) do %>

							<div class="row conversation">
								<div class="col-md-2">
									<%= image_tag avatar_url(other), class: "img-circle avatar-medium" %>
								</div>
								<div class="col-md-2">
									<%= other.fullname %><br>
									<%= conversation.messages.last.message_time %>
								</div>
								<div class="col-md-8">
									<%= conversation.messages.last.content %>
								</div>
							</div>

						<% end %>
					<% end %>
				</div>
			</div>
		</div>

	</div>
</div>
```

**views/messages**

Create folder **messages** and file **index.html.erb**

```ruby
<div class="row">
	
	<div class="col-md-3 text-center">
		<%= image_tag avatar_url(@other), class: "img-circle avatar-medium" %><br>
		<strong><%= @other.fullname %></strong>
		<%= link_to "View Profile", @other, class: "btn btn-default wide" %>
	</div>

	<div class="col-md-9">
		
		<div class="panel panel-default">
			<div class="panel-heading">
				Conversation with <%= @other.fullname %>
			</div>
			<div class="panel-body">
				<div class="container">
					
					<%= form_for [@conversation, @conversation.messages.new], remote: true do |f| %>
						<div class="form-group">
							<%= f.text_area :content, placeholder: "Add a personal message", class: "form-control" %>
						</div>
						<%= f.hidden_field :user_id, value: current_user.id %>

						<div class="actions">
							<%= f.submit "Send Message", class: "btn btn-primary" %>
						</div>
					<% end %>

				</div>
			</div>
		</div>

		<div id="chat">
			<%= render @messages, locals: {conversation: @conversation} %>
		</div>

	</div>

</div>

<%= subscribe_to conversation_messages_path(@conversation) %>
```


Create file **_message.html.erb**

```ruby
<div class="panel">
	<div class="panel-body">
		<%= image_tag avatar_url(message.user), class: "img-circle avatar-small" %>
		<strong><%= message.user.fullname %></strong>

		<span class="pull-right"><%= message.message_time %></span>
		<br>
		<div class="row-space-2">
			<%= message.content %>
		</div>
	</div>
</div>

```

Create file **create.js.erb**

```ruby
<% publish_to conversation_messages_path(@conversation) do %>
	$('#chat').prepend("<%= j render @message %>");
<% end %>

$('#new_message')[0].reset();
```


**application.scss**

```css
.conversation {
  color: #7f7f7f;
  padding: 10px;
}

.conversation:hover, .conversation:focus {
  background-color: #F4F4F5;
}
```

**view/navbar**

```ruby
 <%= link_to conversations_path do %>
  <i class="fa fa-envelope-o"></i>
 <% end %>
```

**card/index.html.erb**

```ruby
<% if current_user != @user %>
	<div class="row-space-2">
		<%= link_to "Send Message", conversations_path(sender_id: current_user.id, recipient_id: @user.id), method: 'post', class: "btn btn-primary wide" %>
	</div>
<% end %>
```

**real-time**

Gemfile

    gem 'private_pub'
    gem 'thin'


