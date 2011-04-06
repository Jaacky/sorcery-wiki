In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

**Known issues: there are some bugs with this module in v0.2.0 that were fixed in v0.2.1, so be sure to use it! Tutorial was updated for v0.2.1**

First Add some db fields:
```
    rails g sorcery_migration oauth
```

Which will create:
```ruby
    class SorceryOauth < ActiveRecord::Migration
      def self.up
        create_table :authentications do |t|
          t.integer :user_id, :null => false
          t.string :provider, :uid, :null => false
    
          t.timestamps
        end
      end
    
      def self.down
        drop_table :authentications
      end
    end
```
```
    rake db:migrate
```

Let's add the submodule too:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:oauth, blabla, blablu, ...]
```

Now we need to associate User with Authentication:
```ruby
    # app/models/user.rb
    class User < ActiveRecord::Base
      attr_accessible :email, :password, :password_confirmation, :authentications_attributes
      activate_sorcery! do |config|
        config.authentications_class = Authentication
      end

      has_many :authentications, :dependent => :destroy
      accepts_nested_attributes_for :authentications
    end
```
```ruby
    # app/models/authentication.rb
    class Authentication < ActiveRecord::Base
      belongs_to :user
    end
```

Let's add links to connect to Twitter and Facebook (you would probably use images in a real app):
```ruby
    # app/views/layouts/application.html.erb
    ...
    <%= link_to 'Login with Twitter', auth_at_provider_path(:provider => :twitter) %> | 
    <%= link_to 'Login with Facebook', auth_at_provider_path(:provider => :facebook) %>
```

We'll need a controller to handle the authentications:
    rails g controller Oauths oauth callback
```ruby
    # app/controllers/oauths_controller.rb
    class OauthsController < ApplicationController
      skip_before_filter :require_login
      
      # sends the user on a trip to the provider,
      # and after authorizing there back to the callback url.
      def oauth
        auth_at_provider(params[:provider])
      end
      
      def callback
        provider = params[:provider]
        if @user = login_from_access_token(provider)
          redirect_to root_path, :notice => "Logged in from #{provider.titleize}!"
        else
          begin
            @user = create_from_provider!(provider)
            # NOTE: this is the place to add '@user.activate!' if you are using user_activation submodule

            reset_session # protect from session fixation attack
            login_user(@user)
            redirect_to root_path, :notice => "Logged in from #{provider.titleize}!"
          rescue
            redirect_to root_path, :alert => "Failed to login from #{provider.titleize}!"
          end
        end
      end
    end
```

Let's add routes for this controller:
```ruby
    # config/routes.rb
    resource :oauth do
      get :callback
    end
    match "oauth/:provider" => "oauths#oauth", :as => :auth_at_provider
```

We've got the plumbing installed, but no specific Twitter/Facebook configuration, so let's add the meat:
```ruby
    # app/controllers/application_controller.rb
    activate_sorcery! do |config|
      ...
      config.oauth_providers = [:twitter, :facebook]
      
      config.twitter.key = "<your key here>"
      config.twitter.secret = "<your key here>"
      config.twitter.callback_url = "http://0.0.0.0:3000/oauth/callback?provider=twitter"
      config.twitter.user_info_mapping = {:username => "screen_name"}
      
      config.facebook.key = "<your key here>"
      config.facebook.secret = "<your key here>"
      config.facebook.callback_url = "http://0.0.0.0:3000/oauth/callback?provider=facebook"
      config.facebook.user_info_mapping = {:username => "name"}
    end
```
You will need to register your app with Twitter/Facebook to get your keys of course.

The 'user_info_mapping' takes care of converting the user info from the provider (Twitter/Facebook) into the attributes that your user has, in this case we only used it to have a username.

Basically how this works is like this:
The user asks to login using a provider. We send the user to authorize at the provider's site, and he is then redirected back. If he doesn't exist in our db, he is auto-created and logged in. If he already exists in our db, he just gets logged in.