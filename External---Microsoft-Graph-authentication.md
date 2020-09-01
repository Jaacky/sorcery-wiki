# External Authentication with Azure AD/Office 365

## This example can be used to setup external authentication for Microsoft. 

### _this is based on version 0.15_

When creating your app you should add the external module...

    $ rails generate sorcery:install external

This will create the migration:

    class SorceryExternal < ActiveRecord::Migration
      def change
        create_table :authentications do |t|
          t.integer :user_id, :null => false
          t.string :provider, :uid, :null => false
    
          t.timestamps
        end

        add_index :authentications, [:provider, :uid]
      end
    end

run `rake db:migrate` to create the table.

You will end up with a column for `user_id`, `provider`, and `uid`. 

Edit your `user.rb` model like so:

    has_many :authentications, :dependent => :destroy
    accepts_nested_attributes_for :authentications

And the `authentication.rb` model:

    belongs_to :user

Now you will want to go to your Azure AD. You do not need to be the owner of the AD the final users will belong to, as long as the AD admin adds your app as an authorized app. You will need the app ID. In the Azure AD control panel go to manage -> app registrations and add your app. In the form give your app a name. Then choose the supported account types for this app. You may want only internal user accounts, or personal accounts or both. Finally under **Redirect URI** add a URI for your app. This is the address Azure will respond to with Authentication info.

    http://localhost:3000/oauth/callback?provider=microsoft

Unlike the other providers you do not want to use the POST route. You will need to add the provider as Sorcery will not know which service to use. 

Now you can configure the external provider info in the `app/config/initializers/sorcery.rb:

   ...

    # Here you can configure each submodule's features.
    Rails.application.config.sorcery.configure do |config|

                 ...

      config.external_providers = [:microsoft]

                 ...

      config.microsoft.key = "<Application (client) ID>" # in the Azure control panel for your app under Overview
      config.microsoft.secret = ENV['client_secret'] # in app -> Certificates and Secrets generate a key and use it here using ENV variables for security
      config.microsoft.callback_url = "http://localhost:3000/oauth/callback?provider=microsoft" #you can use this in development
      config.microsoft.user_info_mapping = {:email => "userPrincipalName", :username => "userPrincipalName"}
      config.microsoft.scope = "openid email https://graph.microsoft.com/User.Read"

Add the following to your `routes.rb`:

    post "oauth/callback" => "oauths#callback"
    get "/oauth/callback/microsoft"  => "oauths#callback" # for microsoft
    get "oauth/callback" => "oauths#callback" # for use with Github, Facebook
    get "oauth/:provider" => "oauths#oauth", :as => :auth_at_provider

Add a link to your login page for Office 365 login in `views/user_sessions/_form`:

     <div>
      <%= link_to 'Login with Office', auth_at_provider_path(:provider => :microsoft) %>
     </div>

Generate a controller for your Oauths:

    rails g controller Oauths oauth callback

And in the oauths_controller.rb add:

    class OauthsController < ApplicationController
      skip_before_action :require_login, raise: false
    
      # sends the user on a trip to the provider,
      # and after authorizing there back to the callback url.
      def oauth
        login_at(params[:provider])
      end
    
      # We are using external providers to log in pre-staged users. This means most likely just MSFT.
      # Hopefully a Google based shop would have similar access to the Object ID for users like
      # in MSFT Azure AD.
    
      def callback
        provider = auth_params[:provider]
        if @user = login_from(provider)
          redirect_to root_path, :notice => "Logged in from #{provider.titleize}!"
        else
          begin
            User.find_by(email: @user
          rescue
            redirect_to root_path, :notice => "Failed to login from #{provider.titleize}!"
          end
        end
      end
    
      #example for Rails 4: add private method below and use "auth_params[:provider]" in place of
      #"params[:provider] above.
    
      private
      def auth_params
        params.permit(:code, :provider, :session_state) #session_state is used for MSFT auths
      end
    end

We now need to add our users to the DB. In Azure AD you can go to Users and at the top of the users table is a "bulk activities" button with a dropdown option of "download users". You can use that to build your users profiles. The important columns are email and Object ID. At a minimum you would add a new user like this on the rails console:

    User.create(email: 'some_user@some_place.com') # we are using email as the username. Your user model may also have other fields.
     #<User:0x00007f88a2199240
       id: 110,
       email: 'some_user@some_place.com', ...>

then create an authentication entry in the authentications table:

    Authentications.create(user_id: 110, provider: 'microsoft', uid: '0439a075-229e-498c-a63b-be446e88e898')

The `uid:` field is the users Object ID from their Azure AD profile. 

When a user clicks on the link to log in via Microsoft they will be take to the Microsoft login page. Once they have authenticated it will make a call back to your app with the parameters for :code, :provider, and :session_state. Your web server will then make a request in to the authentications table for the user with the matching provider and Object ID. If it finds them it will log them in. If not it will fail the login and throw the error message. 

    
