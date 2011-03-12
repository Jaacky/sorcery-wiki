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

And a mailer:
    rails g mailer UserMailer

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


Now when a user is created, an email will be sent to him with activation instructions.
**@user.activate!** will make the user active, send success email if configured and clear the activation code.