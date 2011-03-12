Under construction - incomplete!

In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First And some db fields:
    rails g migration AddActivationToUsers

```ruby
    class AddActivationToUsers < ActiveRecord::Migration
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
    # config/application.rb
    config.sorcery.submodules = [:user_activation, blabla, blablu, ...]
```

And activate the User model with our mailer defined:
```ruby
    # app/models/user.rb
    activate_sorcery! do |config|
      config.user_activation_mailer = UserMailer
    end
```

Now when a user is created, an email will be sent to him. Currently it's not a very informative email, but we will fix that.

**@user.activate!** will make the user active, send success email if configured and clear the activation code.