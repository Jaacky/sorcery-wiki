In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First Add some db fields:

    rails g sorcery:install brute_force_protection --only-submodules

Which will create:

```ruby
class SorceryBruteForceProtection < ActiveRecord::Migration
  def self.up
    add_column :users, :failed_logins_count, :integer, :default => 0
    add_column :users, :lock_expires_at, :datetime, :default => nil
    add_column :users, :unlock_token, :string, :default => nil
  end
    
  def self.down
    remove_column :users, :lock_expires_at
    remove_column :users, :failed_logins_count
    remove_column :users, :unlock_token
  end
end
```

    rake db:migrate

Then add the brute_force_protection submodule:

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:brute_force_protection, blabla, blablu, ...]
```
Refer to your `config/initializers/sorcery.rb` file to customize specifics that are required - it will not work properly without customization: 

```ruby
# config/initializers/sorcery.rb

user.consecutive_login_retries_amount_limit = 50 
user.login_lock_time_period = (60 * 5) # in seconds

user.unlock_token_mailer_disabled = true # I want to have full control over when and how emails are sent

# You'll also need to specify a mailer, a mailer action and a view so that password unlock instructions are sent.
user.unlock_token_mailer = UserMailer

# default mailer action is: send_unlock_token_email - but is configurable
```

You will also need to configure your `sessions_controller.rb` or equivalent file to register that an incorrect login has occurred which will increase the `failed_logins_count` on the relevant user model:

```ruby
# sessions_controller
def create
  login(params[:email], params[:password], params[:remember]) do |user, failure|      
    if user 
    else
      case failure
      when :invalid_password
        user.register_failed_login!
      when :locked
        UserMailer.activation_needed_email(user.id).deliver_later
        puts "oh no, you're locked out! Please check your email" 
      end
     end
  end
end
```

Now please configure your Mailer

```ruby
# UserMailer.rb
def send_unlock_token_email(user_id)
    @user = User.find(user_id)
    # @url  = # add unlock account url
    mail(:to => @user.email,
         :subject => "Please unlock your account",
         :from => "#{@user.email}"
         )
  end


# mailers/users/send_unlock_token_email.html.erb 
<h1>Hello, <%= @user.email %></h1>
<p>
	You've been locked out of your account, because of too many incorrect password attempts.
</p>
<p>
	To unlock, just follow this link: <%= link_to("unlock account.", @url) %>
</p>
<p>Have a great day!</p>

# mailers/users/send_unlock_token_email.text.erb 
Hello, <%= @user.email %>
===============================================

You've been locked out of your account, because of too many incorrect password attempts.

To unlock, just follow this link: <%= link_to("unlock account.", @url) %>

Have a great day!
```

We need to create the unlock url and controllers:

```ruby
  # routes.rb
  scope module: :users do
    get "unlock_accounts/:token", to: "unlock_accounts#create", as: "unlock_accounts"
  end

  # users/unlock_accounts_controller.rb
  module Users
  class UnlockAccountsController < ApplicationController
    skip_before_action :require_login, :only => [:get]
    def get
      if @user = User.load_from_unlock_token(params[:token])
        @user.login_unlock!
        redirect_to login_path, notice: "Please reset your password if you have forgotten it, or otherwise log in: #{view_context.link_to "here ", login_path}."
      else
        not_authenticated
      end
    end
  end
end
```

