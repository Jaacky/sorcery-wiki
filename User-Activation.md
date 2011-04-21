In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First add some db fields:
```
    rails g sorcery_migration user_activation
```


Which will create:
```ruby
    class SorceryUserActivation < ActiveRecord::Migration
      def self.up
        add_column :users, :activation_state, :string, :default => nil
        add_column :users, :activation_token, :string, :default => nil
        add_column :users, :activation_token_expires_at, :datetime, :default => nil
        
        add_index :users, :activation_token
      end
    
      def self.down
        remove_index :users, :activation_token
        
        remove_column :users, :activation_token_expires_at
        remove_column :users, :activation_token
        remove_column :users, :activation_state
      end
    end
```

    rake db:migrate

And a mailer with two actions:

    rails g mailer UserMailer activation_needed_email activation_success_email



You really don't have to use ActionMailer. You can use any ruby object that responds to the above actions. Actually these method names are configurable too! This is useful in case you want to send emails in the background, with gems such as 'delayed_job'. Simply pass your own 'mailer' and make it create the background jobs as you normally would, when triggered by Sorcery.


We need to edit the mailer and add a 'user' parameter to each action because sorcery will send each action the new user as a parameter:
```ruby
    # app/mailers/user_mailer.rb
    def activation_needed_email(user)
    ...

    def activation_success_email(user)
    ...
```

Then add the user_activation submodule:
```ruby
    # config/initializers/sorcery.rb
    Rails.application.config.sorcery.submodules = [:user_activation, blabla, blablu, ...]
```

And configure the User model with our mailer defined:
```ruby
    # config/initializers/sorcery.rb
    Rails.application.config.sorcery.configure do |config|
      ...
      config.user_config do |user|
        ...
        user.user_activation_mailer = UserMailer
        ...
      end
    end
```

Now when a user is created, an email will be sent to him. Currently it's not a very informative email, but we will fix that:
```ruby
    # app/mailers/user_mailer.rb
    def activation_needed_email(user)
      @user = user
      @url  = "http://0.0.0.0:3000/users/#{user.activation_code}/activate"
      mail(:to => user.email,
           :subject => "Welcome to My Awesome Site")
    end
    
    def activation_success_email(user)
      @user = user
      @url  = "http://0.0.0.0:3000/login"
      mail(:to => user.email,
           :subject => "Your account is now activated")
    end
```
```ruby
    # app/views/user_mailer/activation_needed_email.txt.erb
    Welcome to example.com, <%= @user.username %>
    ===============================================
     
    You have successfully signed up to example.com,
    your username is: <%= @user.username %>.
     
    To login to the site, just follow this link: <%= @url %>.
     
    Thanks for joining and have a great day!
```
```ruby
    # app/views/user_mailer/activation_success_email.txt.erb
    Congratz, <%= @user.username %>
    ===============================================
     
    You have successfully activated your example.com account,
    your username is: <%= @user.username %>.
     
    To login to the site, just follow this link: <%= @url %>.
     
    Thanks for joining and have a great day!
```

Nice. The user will get an activation URL, that won't work because we didn't add the code for it. Let's add a controller action that will activate users:
```ruby
    # app/controllers/users_controller.rb
    skip_before_filter :require_login, :only => [:index, :new, :create, :activate]

    def activate
      if @user = User.load_from_activation_token(params[:id])
        @user.activate!
        redirect_to(login_path, :notice => 'User was successfully activated.')
      else
        not_authenticated
      end
    end
```

And fix the routes accordingly:
```ruby
    # config/routes.rb
    resources :users do
      member do
        get :activate
      end
    end
```

**@user.activate!** will make the user active, send a success email and clear the activation code.

If you don't want a success email, in the sorcery configuration of the model, set 'activation_success_email_method_name' to nil.

You can set various other options for this submodule. See the docs for details.