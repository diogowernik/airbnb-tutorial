### 11 - Rooms


1) Create Models

```
rails g model Room home_type:string room_type:string accommodate:integer bed_room:integer bath_room:integer listing_name:string summary:text address:string is_tv:boolean is_kitchen:boolean is_air:boolean is_heating:boolean is_internet:boolean price:decimal active:boolean user:references
```

rake

    rake db:migrate


**models/user.rb**

```ruby
has_many :rooms
```

**models/room.rb**

```ruby
	belongs_to :user
```

2) Create Controller

		rails g controller Rooms index show new create edit update

**controllers/rooms_controller.rb**

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


**config/routes.rb**

```ruby
 resources :rooms
```

3) Views

**views/rooms**

**_form.html.erb**

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

**new.erb**

```ruby
<%= render 'form' %>

```

**sylesheets/application.scss**

```css
label {
  color: #565a5c;
  font-size: 1.1em;
  font-weight: 500;
  margin-bottom: 10px;
}

select {
  color: #565a5c;
  background-color: #fff;
  width: 100%;
  border: 1px solid #ccc;
  border-radius: 2px;
  padding: 10px;
  appearance: none;
  -moz-appearance: none; /* Firefox */
  -webkit-appearance: none; /* Safari and Chrome */
}

.select:before {
  content: '\25bc';
  font-size: 1.2em;
  position: absolute;  
  color: #82888A;
  top: 40px;  
  right: 30px;
  transform: scale(0.84, 0.42);
}

input[type="checkbox"] {
  height: 1.25em;
  width: 1.25em;
  margin-bottom: -0.25em;
  margin-right: 5px;
  vertical-align: top;
  border: 1px solid #c4c4c4;
  border-radius: 2px;
  appearance: none;
  -moz-appearance: none; /* Firefox */
  -webkit-appearance: none; /* Safari and Chrome */
}

input[type="checkbox"]:checked:before {
  content: "\2713";
  position: absolute;
  font-size: 0.95em;
  text-align: center;
  width: 1.25em;
  color: #ff5a5f;
}


```

