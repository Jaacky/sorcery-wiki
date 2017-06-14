In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First Add some db fields:

    rails g sorcery:install external --only-submodules


Which will create:

```ruby
class SorceryExternal < ActiveRecord::Migration
  def change
    create_table :authentications do |t|
      t.integer :user_id, :null => false
      t.string :provider, :uid, :null => false
    
      t.timestamps
    end
  end
end
```

    rake db:migrate

And generate the model Authentication without migration:

    rails g model Authentication --migration=false

Let's add the submodule and configuration:

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:external, blabla, blablu, ...]

Rails.application.config.sorcery.configure do |config|
  ...
  config.external_providers = [:twitter, :facebook]

#add this file to .gitignore BEFORE putting any secret keys in here, or use a system like Figaro to abstract it!!! 
      
  config.twitter.key = "<your key here>"
  config.twitter.secret = "<your key here>"
  config.twitter.callback_url = "http://0.0.0.0:3000/oauth/callback?provider=twitter"
  config.twitter.user_info_mapping = {:email => "screen_name"}
      
  config.facebook.key = "<your key here>"
  config.facebook.secret = "<your key here>"
  config.facebook.callback_url = "http://0.0.0.0:3000/oauth/callback?provider=facebook"
  config.facebook.user_info_path = "me?fields=email,name,username" #etc
  config.facebook.user_info_mapping = {:email => "email", :name => "name", :username => "username", :hometown => "hometown/name"} #etc
  config.facebook.scope = "email,user_hometown,user_interests,user_likes" #etc
  config.facebook.display = "popup"
  ...

  # --- user config ---
  config.user_config do |user|
  ...

    # -- external --
    user.authentications_class = Authentication
    ...

  end
  ...
   
end
```

You will need to register your app with Twitter/Facebook to get your keys of course.

The 'user_info_mapping' takes care of converting the user info from the provider (Twitter/Facebook) into the attributes that your user has, in this case we only used it to have a username.

Now we need to associate User with Authentication:

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
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
  skip_before_action :require_login
      
  # sends the user on a trip to the provider,
  # and after authorizing there back to the callback url.
  def oauth
    login_at(params[:provider])
  end
      
  def callback
    provider = params[:provider]
    if @user = login_from(provider)
      redirect_to root_path, :notice => "Logged in from #{provider.titleize}!"
    else
      begin
        @user = create_from(provider)
        # NOTE: this is the place to add '@user.activate!' if you are using user_activation submodule

        reset_session # protect from session fixation attack
        auto_login(@user)
        redirect_to root_path, :notice => "Logged in from #{provider.titleize}!"
      rescue
        redirect_to root_path, :alert => "Failed to login from #{provider.titleize}!"
      end
    end
  end
  
  #example for Rails 4: add private method below and use "auth_params[:provider]" in place of 
  #"params[:provider] above.

  # private
  # def auth_params
  #   params.permit(:code, :provider)
  # end

end
```

Let's add routes for this controller:

```ruby
# config/routes.rb
post "oauth/callback" => "oauths#callback"
get "oauth/callback" => "oauths#callback" # for use with Github, Facebook
get "oauth/:provider" => "oauths#oauth", :as => :auth_at_provider
```

Basically how this works is like this:
The user asks to login using a provider. We send the user to authorize at the provider's site, and he is then redirected back. If he doesn't exist in our db, he is auto-created and logged in. If he already exists in our db, he just gets logged in.