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
Now please refer to your `config/initializers/sorcery.rb` file to customize specifics that are required - it will not work properly without customizing it. Here are some examples.

```ruby
# config/initializers/sorcery.rb
user.consecutive_login_retries_amount_limit = 50 # 50 is the limit you can retry with a wrong password without being locked out
user.login_lock_time_period = (60 * 5) # you'll be locked out for 60 * 5 seconds

# You'll also need to specify a mailer, a mailer action and a view so that password unlock instructions are sent.
```

You will also need to configure your `sessions_controller.rb` or equivalent file to register that an incorrect login has occurred which will increase the `failed_logins_count` on the relevant user model:

```ruby
# sessions_controller
# here is a rough sample:
def create
login(params[:email], params[:password], params[:remember]) do |user, failure|      
        if user 
        else
            case failure
            when :invalid_password
              user.register_failed_login!
              flash.now[:alert] = "Wrong password provided. Please carefully type your password or #{view_context.link_to " reset it ", new_password_reset_path} if you've forgotten."
            when :locked
              flash.now[:alert] = "Too many incorrect attempts. We've locked your account - please check your email to unlock"
        end
end
end
```