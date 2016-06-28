The particular setup you describe mixes several types of associations.

### A) User and Card

First we have a User model and second a Card model. Each card belonging to one user, hence we have a User :has_many cards, Card belongs_to :user association. This relationship is stored in the card's user_id field.

    rails g model Card user_id:integer ...
    rails g model User ...


class Card < ActiveRecord::Base
  belongs_to :user
end

class User < ActiveRecord::Base
  has_many :cards
end
B) FavoriteCard

Next we need to decide on how to implement the story that a user should be able to mark favorite cards.

This can be done by using a join model - let's call it FavoriteCard - with the columns :user_id and :card_id. The association we're building here is a has_many :through association.

A User  
  - has_many :favorite_cards  
  - has_many :favorites, through: :favorite_cards, source: :card

A Card
  - has_many :favorite_cards  
  - has_many :favorited_by, through: :favorite_cards, source: :user 
      # returns the users that favorite a card
Adding this favorites has_many :through association to the models, we get our final results.

$ rails g model FavoriteCard card_id:integer user_id:integer

# Join model connecting user and favorites
class FavoriteCard < ActiveRecord::Base
  belongs_to :card
  belongs_to :user
end

---

class User < ActiveRecord::Base
  has_many :cards

  # Favorite cards of user
  has_many :favorite_cards # just the 'relationships'
  has_many :favorites, through: :favorite_cards, source: :card # the actual cards a user favorites
end

class Card < ActiveRecord::Base
  belongs_to :user

  # Favorited by users
  has_many :favorite_cards # just the 'relationships'
  has_many :favorited_by, through: :favorite_cards, source: :user # the actual users favoriting a card
end
C) Interacting with the associations

##
# Association "A"

# Find cards the current_user created
current_user.cards

# Create card for current_user
current_user.cards.create!(...)

# Load user that created a card
@card = Card.find(1)
@card.user

##
#  Association "B"

# Find favorites for current_user
current_user.favorites

# Find which users favorite @card
@card = Card.find(1)
@card.favorited_by # Retrieves users that have favorited this card

# Add an existing card to current_user's favorites
@card = Card.find(1)
current_user.favorites << @card

# Remove a card from current_user's favorites
@card = Card.find(1)
current_user.favorites.delete(@card)  # (Validate)
D) Controller Actions

There may be several approaches on how to implement Controller actions and routing. I quite like the one by Ryan Bates shown in Railscast #364 on the ActiveRecord Reputation System. The part of a solution described below is structured along the lines of the voting up and down mechanism there.

In our Routes file we add a member route on cards called favorite. It should respond to post requests. This will add a favorite_card_path(@card) url helper for our view.

# config/routes.rb
resources :cards do
  put :favorite, on: :member
end
In our CardsController we can now add the corresponding favorite action. In there we need to determine what the user wants to do, favoriting or unfavoriting. For this a request parameter called e.g. type can be introduced, that we'll have to pass into our link helper later too.

class CardsController < ...

  # Add and remove favorite cards
  # for current_user
  def favorite
    type = params[:type]
    if type == "favorite"
      current_user.favorites << @card
      redirect_to :back, notice: 'You favorited #{@card.name}'

    elsif type == "unfavorite"
      current_user.favorites.delete(@card)
      redirect_to :back, notice: 'Unfavorited #{@card.name}'

    else
      # Type missing, nothing happens
      redirect_to :back, notice: 'Nothing happened.'
    end
  end

end
In your view you can then add the respective links to favoriting and unfavoriting cards.

<% if current_user %>
  <%= link_to "favorite",   favorite_card_path(@card, type: "favorite"), method: :put %>
  <%= link_to "unfavorite", favorite_card_path(@card, type: "unfavorite"), method: :put %>
<% end %>
That's it. If a user clicks on the "favorite" link next to a card, this card is added to the current_user's favorites.