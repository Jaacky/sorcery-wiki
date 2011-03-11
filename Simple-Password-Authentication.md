In this tutorial we will generate a new Rails 3 app and add sorcery as the authentication engine.
At the end of the tutorial we will be able to register a user, and then login and logout with a username and a password.

I'm using rvm so i'll first create a separate gemset:
    rvm gemset create tutorial
    rvm use 1.9.2@tutorial

Now I'll get rails:
    gem install rails mysql2

Now I'll create a new app with mysql database:
    rails new tutorial -d mysql
    cd tutorial
    rake db:create

We'll start by adding the User resource so that we'll be able to register new users:
    rails g scaffold User username:string email:string crypted_password:string salt:string
    rake db:migrate

We don't want users to edit/view their crypted password or salt, so we'll remove these from all templates in app/views/users/.
We'll need to add a password 'virtual' field instead, that will hold the password before it is encrypted into the database. 
    # app/views/users/_form.html.erb
    <div class="field">
      <%= f.label :password %><br />
      <%= f.password_field :password %>
    </div>
    <div class="field">
      <%= f.label :password_confirmation %><br />
      <%= f.password_field :password_confirmation %>
    </div>

The virtual attributes will be added via 'validates_confirmation_of' as seen below. You may also want to allow the user to change only some of his attributes like so:
    # app/models/user.rb
    class User < ActiveRecord::Base
      attr_accessible :email, :password, :password_confirmation
  
      validates_confirmation_of :password, :on => :create, :message => "should match confirmation"
    end

It's time to add Sorcery in, so we'll get a crypted password when registering a user.
    # Gemfile
    gem 'sorcery'
    bundle install

We need to let the User model know it is using sorcery. We do that like this:
    # app/models/user.rb
    class User < ActiveRecord::Base
      attr_accessible :email, :password, :password_confirmation
      activate_sorcery!

      validates_confirmation_of :password, :on => :create, :message => "should match confirmation"
    end

Now run the app and create a new user.
Voila! The password was automatically encrypted, and a salt was also auto-created!
By default the encryption algorithm used is BCrypt (using the bcrypt-ruby gem).



