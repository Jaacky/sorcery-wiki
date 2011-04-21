**NOTE: 'authenticates_with_sorcery!' used to be 'activate_sorcery!' in v0.3.1 and before.**


**NOTE: 'redirect_back_or_to' used to be 'return_or_redirect_to' in v0.3.0 and before.**

In this tutorial we will generate a new Rails 3 app and add sorcery as the authentication engine.
At the end of the tutorial we will be able to register a user, and then login and logout with a username and a password.

I'm using rvm so i'll first create a separate gemset:
```
    rvm gemset create tutorial
    rvm use 1.9.2@tutorial
```


Now I'll get rails and bundler:
```
    gem install rails mysql2 bundler
```


Now I'll create a new app with mysql database:
```
    rails new tutorial -d mysql
    cd tutorial
    rake db:create
```


At this point we'll add the sorcery gem into the Gemfile, and 'bundle install'.
Then we'll run:
```
    rake sorcery:bootstrap
```

To create an initializer with the default configuration for sorcery.
This will actually only be useful in the next tutorials, as the default config is good for this one.

We'll start building the app by adding the User resource so that we'll be able to register new users:
```
    rails g scaffold User username:string email:string crypted_password:string salt:string
    rake db:migrate
```


There is also a generator for migrations which you can use if your app doesn't need the whole scaffold. To generate the basic User migration use it like so:
```
    rails g sorcery_migration core
```


We don't want users to edit/view their crypted password or salt, so we'll remove these from all templates in app/views/users/.
We'll need to add a password 'virtual' field instead, that will hold the password before it is encrypted into the database:
```ruby
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

The virtual attributes will be added via 'validates_confirmation_of' as seen below. You may also want to allow the user to change only some of his attributes like so:
```ruby
    # app/models/user.rb
    class User < ActiveRecord::Base
      attr_accessible :email, :password, :password_confirmation
  
      validates_confirmation_of :password, :on => :create, :message => "should match confirmation"
    end
```

It's time to add Sorcery in, so we'll get a crypted password when registering a user.
    # Gemfile
    gem 'sorcery'
    bundle install

We need to let the User model know it is using sorcery. We do that like this:
```ruby
    # app/models/user.rb
    class User < ActiveRecord::Base
      attr_accessible :email, :password, :password_confirmation
      authenticates_with_sorcery!

      validates_confirmation_of :password, :on => :create, :message => "should match confirmation"
    end
```

Now run the app and create a new user. 
Voila! The password was automatically encrypted, and a salt was also auto-created!
By default the encryption algorithm used is BCrypt (using the bcrypt-ruby gem) but that can be configured, as well as the salt, and the database field names.

Now we need a way to login after registering.
    rails g controller UserSessions new create destroy

Make it look like this:
```ruby
    # app/controllers/user_sessions_controller.rb
    class UserSessionsController < ApplicationController
      def new
        @user = User.new
      end
  
      def create
        respond_to do |format|
          if @user = login(params[:username],params[:password])
            format.html { redirect_back_or_to(:users, :notice => 'Login successfull.') }
            format.xml { render :xml => @user, :status => :created, :location => @user }
          else
            format.html { flash.now[:alert] = "Login failed."; render :action => "new" }
            format.xml { render :xml => @user.errors, :status => :unprocessable_entity }
          end
        end
      end
    
      def destroy
        logout
        redirect_to(:users, :notice => 'Logged out!')
      end
    end
```

'login' takes care of login, 'logout' takes care of logout.
'redirect_back_or_to' takes care of redirecting the user to the page he asked for before reaching the login form, if such a page exists.

Let's create the login form:
```ruby
    # app/views/user_sessions/new.html.erb
    <h1>Login</h1>

    <%= render 'form' %>

    <%= link_to 'Back', user_sessions_path %>
```
```ruby
    # app/views/user_sessions/_form.html.erb
    <%= form_tag user_sessions_path, :method => :post do %>
      <div class="field">
        <%= label_tag :username %><br />
        <%= text_field_tag :username %>
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
  
    match 'login' => 'user_sessions#new', :as => :login
    match 'logout' => 'user_sessions#destroy', :as => :logout
```

One Last thing, we need some navigation links, and a way to display flash messages:
```ruby
    # app/views/layouts/application.html.erb
    <!DOCTYPE html>
    <html>
    <head>
      <title>Tutorial</title>
      <%= stylesheet_link_tag :all %>
      <%= javascript_include_tag :defaults %>
      <%= csrf_meta_tag %>
    </head>
    <body>
    
    	<div id="nav">
    		<% if current_user %>
            	<%= link_to "Edit Profile", edit_user_path(current_user.id) %>
            	<%= link_to "Logout", :logout %>
            <% else %>
            	<%= link_to "Register", new_user_path %> |
            	<%= link_to "Login", :login %>
            <% end %>
    	</div>
        <div>
    		<p id="notice"><%= notice %></p>
    		<p id="alert"><%= alert %></p>
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
    before_filter :require_login

    # app/controllers/users_controller.rb
    skip_before_filter :require_login, :only => [:index, :new, :create]

    # app/controllers/user_sessions_controller.rb
    skip_before_filter :require_login, :except => [:destroy]
```

The default controller code redirects you to "show" action after "create" completes.
This means when you register a user, you will be redirected to a protected action, so let's fix it by redirecting to the users index action:
```ruby
    # app/controllers/users_controller.rb
    ...
    if @user.save
        format.html { redirect_to(:users, :notice => 'User was successfully created.') }
    ...
```

There's only one problem. The default action of 'require_login' when blocking a logged-out user is to redirect to application root. Our root is the default Rails index.html page. Let's fix that by overriding the default behavior:

    rm public index.html

```ruby
    # app/controllers/application_controller.rb
    protected
    def not_authenticated
      redirect_to root_path, :alert => "Please login first."
    end
```

Now some parts of the application are protected from logged-out users. Also, if you try to edit a user while logged out, get denied and then login at the login form, you'll see you are automatically taken to the edit page. If 'not_authenticated' is a problematic name for you, it is possible to supply a different method at configuration time, that will be called when a user is denied due to being logged-out.