### 12 - Photos


1) install paperclip and imagemagick (linux)

```
brew install imagemagick (mac)
```

**Gemfile**

```ruby
gem 'paperclip'
```

**Terminal**

```ruby
bundle install
```

2) Models

**models/photo.rb**

```ruby
class Photo < ActiveRecord::Base
  belongs_to :room

  has_attached_file :image, styles: { medium: "300x300>", thumb: "100x100>" }
  validates_attachment_content_type :image, content_type: /\Aimage\/.*\Z/
end
```


**models/rooms.rb**

```ruby
	has_many :photos
```

3) routes

**config/routes.rb**

```ruby
	resources :photos
```

4) controllers

**rooms controller.rb**

```ruby
class RoomsController < ApplicationController
  before_action :set_room, only: [:show, :edit, :update]
  before_action :authenticate_user!, except: [:show]

  def index
    @rooms = current_user.rooms
  end

  def show
    @photos = @room.photos

    @booked = Reservation.where("room_id = ? AND user_id = ?", @room.id, current_user.id).present? if current_user

    @reviews = @room.reviews
    @hasReview = @reviews.find_by(user_id: current_user.id) if current_user
  end

  def new
    @room = current_user.rooms.build
  end

  def create
    @room = current_user.rooms.build(room_params)

    if @room.save

      if params[:images] 
        params[:images].each do |image|
          @room.photos.create(image: image)
        end
      end

      @photos = @room.photos
      redirect_to edit_room_path(@room), notice: "Saved..."
    else
      render :new
    end
  end

  def edit
    if current_user.id == @room.user.id
      @photos = @room.photos
    else
      redirect_to root_path, notice: "You don't have permission."
    end
  end

  def update
    if @room.update(room_params)

      if params[:images] 
        params[:images].each do |image|
          @room.photos.create(image: image)
        end
      end

      redirect_to edit_room_path(@room), notice: "Updated..."
    else
      render :edit
    end
  end

  private
    def set_room
      @room = Room.find(params[:id]) 
    end

    def room_params
      params.require(:room).permit(:home_type, :room_type, :accommodate, :bed_room, :bath_room, :listing_name, :summary, :address, :is_tv, :is_kitchen, :is_air, :is_heating, :is_internet, :price, :active)
    end
end





```

5) views

**rooms/_form**

```ruby
<div class="panel panel-default">
	<div class="panel-heading">
		Create your beautiful place
	</div>
	<div class="panel-body">
		<div class="container">
			
			<%= form_for @room, :html => { multipart: true } do |f| %>

				<div class="row">
					<div class="col-md-4 select">
						<div class="form-group">
							<label>Home Type</label>
							<%= f.select :home_type, [["Apartment","Apartment"], ["House","House"], ["Bed & Breakfast","Bed & Breakfast"]], id: "home_type", prompt: "Select...", class: "form-control" %>
						</div>
					</div>

					<div class="col-md-4 select">
						<div class="form-group">
							<label>Room Type</label>
							<%= f.select :room_type, [["Entire","Entire"], ["Private","Private"], ["Shared","Shared"]],prompt: "Select...", class: "form-control" %>
						</div>
					</div>

					<div class="col-md-4 select">
						<div class="form-group">
							<label>Accommodate</label>
							<%= f.select :accommodate, [["1",1], ["2",2], ["3",3], ["4",4], ["5",5], ["6+",6]], prompt: "Select...", class: "form-control" %>
						</div>
					</div>
				</div>

				<div class="row">
					<div class="col-md-4 select">
						<div class="form-group">
							<label>Bedrooms</label>
							<%= f.select :bed_room, [["1",1], ["2",2], ["3",3], ["4+",4]], prompt: "Select...", class: "form-control" %>
						</div>
					</div>

					<div class="col-md-4 select">
						<div class="form-group">
							<label>Bathrooms</label>
							<%= f.select :bath_room, [["1",1], ["2",2], ["3",3], ["4+",4]], prompt: "Select...", class: "form-control" %>
						</div>
					</div>
				</div>

				<div class="row">
					<div class="form-group">
						<label>Listing name</label>
						<%= f.text_field :listing_name, placeholder: "Be clear and descriptive", class: "form-control" %>
					</div>
				</div>

				<div class="row">
					<div class="form-group">
						<label>Summary</label>
						<%= f.text_area :summary, rows: 5, placeholder: "Tell about your house", class: "form-control" %>
					</div>
				</div>

				<div class="row">
					<div class="form-group">
						<label>Address</label>
						<%= f.text_field :address, placeholder: "What is your address?", class: "form-control" %>
					</div>
				</div>

				<div class="row">
					<div class="col-md-4">
						<div class="form-group">
							<%= f.check_box :is_tv %> TV
						</div>
						<div class="form-group">
							<%= f.check_box :is_kitchen %> Kitchen
						</div>
						<div class="form-group">
							<%= f.check_box :is_internet %> Internet
						</div>
					</div>

					<div class="col-md-4">
						<div class="form-group">
							<%= f.check_box :is_heating %> Heating
						</div>
						<div class="form-group">
							<%= f.check_box :is_air %> Air Conditioning
						</div>
					</div>	
				</div>	

				<div class="row">
					<div class="col-md-4">
						<div class="form-group">
							<label>Nightly Price</label>
							<div class="input-group">
								<div class="input-group-addon">$</div>
								<%= f.text_field :price, placeholder: "eg. $100", class: "form-control" %>
							</div>	
						</div>
					</div>
				</div>

				<div class="row">
					<div class="col-md-4">
						<div class="form-group">							
							<span class="btn btn-default btn-file">
							    <i class="fa fa-cloud-upload fa-lg"></i> Upload Photos 
							    <%= file_field_tag "images[]", type: :file, multiple: true %>
							</span>
						</div>		
					</div>
				</div>

				<div id="photos"><%= render 'photos/list' %></div>

				<div class="row">
					<div class="form-group">
						<%= f.check_box :active %> Active
					</div>
				</div>

				<div class="actions">
					<%= f.submit "Save", class: "btn btn-primary" %>
				</div>

			<% end %>

		</div>
	</div>
</div>

```

**photos/_list.html.erb**

```ruby
<% if @photos %>

	<div class="row">
		<% @photos.each do |photo| %>		
			<div class="col-md-4">
				<div class="panel panel-default">
				  <div class="panel-heading preview">
				  	<%= image_tag photo.image.url() %>
				  </div>
				  <div class="panel-body">
				  	<span class="pull-right">
				  		<%= link_to photo, remote: true, method: :delete, data: {confirm: "Are you sure?"} do %>
					  		<i class="fa fa-times fa-lg"></i>
				  		<% end %>
				  	</span>
				  </div>
				</div>
			</div>
		<% end %>			
	</div>

<% end %>
```

6) styles

**application.scss**

```css
.btn-file {
    position: relative;
    overflow: hidden;
}

.btn-file input[type=file] {
    position: absolute;
    top: 0;
    right: 0;
    min-width: 100%;
    min-height: 100%;
    font-size: 100px;
    text-align: right;
    filter: alpha(opacity=0);
    opacity: 0;
    outline: none;
    background: white;
    cursor: inherit;
    display: block;
}

.panel-heading.preview {
  padding: 0;
}

.panel-heading.preview img{  
  width: 100%;
}
```

7) remove with ajax

**photos_controller.rb**

```ruby
class PhotosController < ApplicationController

	def destroy
		@photo = Photo.find(params[:id])
		room = @photo.room

		@photo.destroy
		@photos = Photo.where(room_id: room.id)

		respond_to :js
	end
end
```

**_list.erb**

```ruby
<%= link_to photo, remote: true, method: :delete, data: {confirm: "Are you sure?"} do %>

```

**destroy.js.erb**

```ruby
$('#photos').html("<%= j render 'photos/list' %>")
```
