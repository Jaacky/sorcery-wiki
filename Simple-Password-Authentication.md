In this tutorial we will add Sorcery as the registration/authentication engine to **Rails 4 app**.
At the end of the tutorial we will be able to register a user, and then login and logout with a username and a password.

Let's start from adding the sorcery gem into the Gemfile, and 'bundle install'. In the app's Gemfile:

```ruby
# Gemfile
gem 'sorcery'
```


We'll start building the app by running this generator added by sorcery:

    rails g sorcery:install

It will create the initializer file, the User model, unit test stubs, and the default migration, which we want to run now:

    rake db:migrate



To get some views and controllers fast, we'll run rails scaffold generator, and skip the files we already created (we'll need to delete the new migration manually though):

    rails g scaffold user email:string crypted_password:string salt:string --migration false


We don't want users to edit/view their crypted password or salt, so we'll remove these from all templates in app/views/users/.

We also need to allow UsersController receive form attributes:

    class UsersController < ApplicationController
      # ...
      private

      def user_params
        params.require(:user).permit(:email, :password, :password_confirmation)
      end
    end

We'll need to add a password 'virtual' field instead, that will hold the password before it is encrypted into the database:

```rhtml
# app/views/users/_form.html.erb
<div class="field">
   <%= f.label :password %><br />
   <%= f.password_field :password %>
</div>
<div class="field">
   <%= f.label :password_confirmation %><br />
   <%= f.password_field :password_confirmation %>
</div>
```

The virtual attributes will be added via `validates :password, confirmation: true` as seen below. You may also want to allow the user to change only some of his attributes. Also we need to let the User model know it is using sorcery. We do that like this:

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  authenticates_with_sorcery!

  validates :password, length: { minimum: 3 }
  validates :password, confirmation: true
  validates :password_confirmation, presence: true

  validates :email, uniqueness: true
end
```

Now run the app and create a new user at [http://0.0.0.0:3000/users](http://0.0.0.0:3000/users).
Voila! The password was automatically encrypted, and a salt was also auto-created!
By default the encryption algorithm used is BCrypt (using the bcrypt-ruby gem) but that can be configured, as well as the salt, and the database field names.

Now we need a way to login after registering, so we'll generate a controller for that:


    rails g controller UserSessions new create destroy


Make it look like this:

```ruby
# app/controllers/user_sessions_controller.rb
class UserSessionsController < ApplicationController
  def new
    @user = User.new
  end

  def create
    if @user = login(params[:email], params[:password])
      redirect_back_or_to(:users, notice: 'Login successful')
    else
      flash.now[:alert] = 'Login failed'
      render action: 'new'
    end
  end

  def destroy
    logout
    redirect_to(:users, notice: 'Logged out!')
  end
end
```

'login' takes care of login, 'logout' takes care of logout.
'redirect_back_or_to' takes care of redirecting the user to the page he asked for before reaching the login form, if such a page exists.

Let's create the login form:

```rhtml
# app/views/user_sessions/new.html.erb
<h1>Login</h1>

<%= render 'form' %>

<%= link_to 'Back', user_sessions_path %>
```

```rhtml
# app/views/user_sessions/_form.html.erb
<%= form_tag user_sessions_path, :method => :post do %>
  <div class="field">
    <%= label_tag :email %><br />
    <%= text_field_tag :email %>
  </div>
  <div class="field">
    <%= label_tag :password %><br />
    <%= password_field_tag :password %>
  </div>
  <div class="actions">
    <%= submit_tag "Login" %>
  </div>
<% end %>
```

And define some routes. At this point make sure your routes.rb file includes these routes and remove others:

```ruby
# config/routes.rb
root :to => 'users#index'
resources :user_sessions
resources :users

get 'login' => 'user_sessions#new', :as => :login
post 'logout' => 'user_sessions#destroy', :as => :logout
```

One Last thing, we need some navigation links, and a way to display flash messages:

```rhtml
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title>Tutorial</title>
      <%= stylesheet_link_tag    "application" %>
      <%= javascript_include_tag "application" %>
      <%= csrf_meta_tags %>
  </head>
  <body>

  <div id="nav">
    <% if current_user %>
      <%= link_to "Edit Profile", edit_user_path(current_user.id) %>
      <%= link_to "Logout", :logout, method: :post %>
    <% else %>
      <%= link_to "Register", new_user_path %> |
      <%= link_to "Login", :login %>
    <% end %>
  </div>
  <div>
    <p id="notice"><%= flash[:notice] %></p>
    <p id="alert"><%= flash[:alert] %></p>
  </div>
  <%= yield %>

  </body>
</html>
```

Now you can click on the "Login" link, enter your username and password, and get redirected to the users list with a nice flash message. Try the "Logout" link as well.

OK, that's nice, but our logged-out users can do anything a logged-in user can, like editing users for example.
Let's fix that by protecting our sensitive controller actions:

```ruby
# app/controllers/application_controller.rb
before_action :require_login

# app/controllers/users_controller.rb
skip_before_filter :require_login, only: [:index, :new, :create]

# app/controllers/user_sessions_controller.rb
skip_before_filter :require_login, except: [:destroy]
```

The default controller code redirects you to "show" action after "create" completes.
This means when you register a user, you will be redirected to a protected action, so let's fix it by redirecting to the users index action:

```ruby
# app/controllers/users_controller.rb
...
if @user.save
  redirect_to(:users, notice: 'User was successfully created')
  ...
```

```ruby
# app/controllers/application_controller.rb
private
def not_authenticated
  redirect_to login_path, alert: "Please login first"
end
```

Now some parts of the application are protected from logged-out users. Also, if you try to edit a user while logged out, get denied and then login at the login form, you'll see you are automatically taken to the edit page. If 'not_authenticated' is a problematic name for you, it is possible to supply a different method at configuration time, that will be called when a user is denied due to being logged-out.