In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

Let's add the db stuff first:

```
    rails g sorcery:install activity_logging --only-submodules
```


Which will create:

```ruby
    class SorceryActivityLogging < ActiveRecord::Migration
      def self.up
        add_column :users, :last_login_at,     :datetime, :default => nil
        add_column :users, :last_logout_at,    :datetime, :default => nil
        add_column :users, :last_activity_at,  :datetime, :default => nil
        add_column :users, :last_login_from_ip_address, :string, :default => nil

        add_index :users, [:last_logout_at, :last_activity_at]
      end

      def self.down
        remove_index :users, [:last_logout_at, :last_activity_at]

        remove_column :users, :last_activity_at
        remove_column :users, :last_logout_at
        remove_column :users, :last_login_at
      end
    end
```

run the migration

```
    rake db:migrate
```

And add the submodule:

```ruby
    # config/initializers/sorcery.rb
    Rails.application.config.sorcery.submodules = [:activity_logging, blabla, blublu, ...]
```

That's it, you have activity logging on! All recent activity times will be logged for each user.

Now for the fun stuff - Let's say you want to display on your site the usernames of the users currently logged in:

```ruby
    # app/controllers/application_controller.rb
    ...
    helper_method :current_users_list
    protected
    def current_users_list
      current_users.map {|u| u.username}.join(", ")
    end
    ...
```

```rhtml
    # app/views/layouts/application.html.erb
    ...
    <% if current_user %>
      <div id="current_users"> Currently active users: <%= current_users_list %></div>
    <% end %>
    <%= yield %>
    ...
```

There you have it! Try logging in with two users from different browsers and they should see each other's name on the front page.