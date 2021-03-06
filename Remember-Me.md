In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First run the generator below, or include the code yourself:

    rails g sorcery:install remember_me --only-submodules


```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:remember_me, blabla, blablu, ...]
```

Now you'll need some new db fields so add a migration that looks like this:

```ruby
class AddRememberMeTokenToUsers < ActiveRecord::Migration
  def self.up
    add_column :users, :remember_me_token, :string, :default => nil
    add_column :users, :remember_me_token_expires_at, :datetime, :default => nil

    add_index :users, :remember_me_token
  end

  def self.down
    remove_index :users, :remember_me_token

    remove_column :users, :remember_me_token_expires_at
    remove_column :users, :remember_me_token
  end
end
```



And then run

    rake db:migrate

Adding remember me to the app is very simple - just use the good old `login` method with a third parameter.
This is usually the result of a "Remember me" check box from a form.
If it's anything that evals to true, it will 'remember' the user.

```ruby
# app/views/user_sessions/_form.html.erb
<%= check_box_tag :remember, params[:remember], true %>
<%= label_tag :remember %>

# app/controllers/user_sessions_controller.rb
@user = login(params[:email], params[:password], params[:remember])
```

To "forget me" just **logout**.

If you ever need finer control you can use the controller methods:

```ruby
remember_me!
forget_me!
```

The default session length for remember me is 1 week. To change this, use the following config option. 

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.configure do |config|
  ...
  config.user_config do |user|
    ...
    user.remember_me_for = 604800 # A week in seconds
    ...
  end
end
```

See the gem docs for a full list of all configuration options.