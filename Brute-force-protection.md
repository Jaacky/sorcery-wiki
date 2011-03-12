In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First Add some db fields:
    rails g migration AddBruteForceProtectionToUsers

```ruby
    class AddBruteForceProtectionToUsers < ActiveRecord::Migration
      def self.up
        add_column :users, :failed_logins_count, :integer, :default => 0
        add_column :users, :lock_expires_at, :datetime, :default => nil
      end
    
      def self.down
        remove_column :users, :lock_expires_at
        remove_column :users, :failed_logins_count
      end
    end
```
    rake db:migrate

Then add the brute_force_protection submodule:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:brute_force_protection, blabla, blablu, ...]
```

That's it! You now only need to configure options like how many failed logins are allowed and for how long to lock an account when this number has been reached. See the docs for specifics.