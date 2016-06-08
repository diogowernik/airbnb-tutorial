### 11 - Review Stars


1) Create Models

    rails g model Review comment:text star:integer room:references user:references

rake

    rake db:migrate

**db/____create_reviews.rb**

```ruby
class CreateReviews < ActiveRecord::Migration
  def change
    create_table :reviews do |t|
      t.text :comment
      t.integer :star, default: 1
      t.references :room, index: true, foreign_key: true
      t.references :user, index: true, foreign_key: true

      t.timestamps null: false
    end
  end
end
```

**models/user.rb**

```ruby
has_many :reviews
```

**models/room.rb**

```ruby
	has_many :reviews




  def average_rating
    reviews.count == 0 ? 0 : reviews.average(:star).round(2)
  end

```

2) Create Controller

**controllers/reviews_controller.rb**

Create file: **reviews_controller.rb**

```ruby
class ReviewsController < ApplicationController

	def create
		@review = current_user.reviews.create(review_params)
		redirect_to @review.room
	end

	def destroy
		@review = Review.find(params[:id])
		room = @review.room
		@review.destroy

		redirect_to room
	end

	private
		def review_params
			params.require(:review).permit(:comment, :star, :room_id)
		end
end
```

**controllers/rooms_controller.rb**

Create file: **rooms_controller.rb**

```ruby

@booked = Reservation.where("room_id = ? AND user_id = ?", @room.id, current_user.id).present? if current_user

@reviews = @room.reviews
@hasReview = @reviews.find_by(user_id: current_user.id) if current_user
```

**config/routes.rb**

```ruby

resources :rooms do
  resources :reviews, only: [:create, :destroy]
end

```

3) Views

**views/reviews**

Create folder **reviews** and file **_form.html.erb**

```ruby
<%= form_for([@room, @room.reviews.new]) do |f| %>
	
	<div id="user_stars"></div>

	<div class="form-group">
		<%= f.text_area :comment, rows: 3, class: "form-control" %>	
	</div>
	

	<%= f.hidden_field :room_id, value: @room.id %>

	<div class="actions">
		<%= f.submit "Create", class: "btn btn-primary" %>
	</div>

<% end %>

<script>
	$('#user_stars').raty({
		path: '/assets',
		scoreName: 'review[star]',
		score: 1
	});
</script>
```

**views/reviews.html.erb**

Create file **_lists.html.erb**

```ruby
<% if @reviews.count == 0 %>
	<div class="text-center"><h4>There is no Review yet</h4></div>
<% else %>

	<% @reviews.order("id desc").each do |r| %>
		<hr>

		<div class="row">
			<div class="col-md-1">
				<%= image_tag avatar_url(r.user), class: "img-circle avatar-medium" %>
			</div>
			<div class="col-md-11">
				<div>
					<strong><%= r.user.fullname %> <div id="stars_<%= r.id %>"></div> </strong>
					<span class="pull-right">
						<%= link_to "Remove My Review", [@room, r], method: :delete, data: {confirm: "Are you sure?"} if current_user && current_user == r.user %>
					</span>
				</div>

				<div><%= r.created_at.strftime("%v") %></div>
				<div><%= r.comment %></div>
			</div>
		</div>

		<script>
			$('#stars_<%= r.id %>').raty({
				path: '/assets',
				readOnly: true,
				score: <%= r.star %>
			});
		</script>

	<% end %>

<% end %>
```



**views/rooms/show.html.erb**

Add

```ruby
<div class="row">
	<div class="col-md-12">
		<h3>Reviews <span id="average_rating"></span> (<%= @reviews.count %>)</h3>
		<div class="container">
			<div>
				<%= render 'reviews/form' if @booked && !@hasReview %>
			</div>
			<div>
				<%= render 'reviews/list' %>
			</div>
		</div>
	</div>
</div>
```

**Plugin to add stars**

JQuery Raty

**assets/images**

Add images 

star-half.png / star-off.png /  star-on.png

**assets/javascript**
jquery.raty.js

2 - Add the stars in the users rooms list

```ruby
<h4>Reviews</h4><br>

<% @rooms.each do |room| %>
	<% if !room.reviews.blank? %>

		<% room.reviews.each do |review| %>
			<div class="row">
				<div class="col-md-2 text-center">
					<%= image_tag avatar_url(review.user), class: "img-circle avatar-medium" %><br>
					<%= review.user.fullname %>
				</div>
				<div class="col-md-10">
					<%= link_to room.listing_name, room %><br>
					<%= review.comment %><br>
					<%= review.created_at.strftime("%v") %>
				</div>
			</div>
		<% end %>

	<% end %>
<% end %>
```