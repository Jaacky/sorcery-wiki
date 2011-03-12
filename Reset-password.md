In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First Add some db fields:
    rails g migration AddActivationToUsers

```ruby
    class AddResetPasswordToUsers < ActiveRecord::Migration
      def self.up
        add_column :users, :reset_password_token, :string, :default => nil
        add_column :users, :reset_password_token_expires_at, :datetime, :default => nil
        add_column :users, :reset_password_email_sent_at, :datetime, :default => nil
      end
    
      def self.down
        remove_column :users, :reset_password_email_sent_at
        remove_column :users, :reset_password_token_expires_at
        remove_column :users, :reset_password_token
      end
    end
```
    rake db:migrate

And a mailer with one action:
    rails g mailer UserMailer reset_password_email

We need to edit the mailer and add a 'user' parameter to the action because sorcery will send it the new user as a parameter:
```ruby
    # app/mailers/user_mailer.rb
    def reset_password_email(user)
    ...
```

Then add the reset_password submodule:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:reset_password, blabla, blablu, ...]
```


