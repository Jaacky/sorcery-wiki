In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First add some db fields:

### ActiveRecord

```
rails g sorcery:install user_activation  --only-submodules
```


Which will create:

```ruby
class SorceryUserActivation < ActiveRecord::Migration
  def change
    add_column :users, :activation_state, :string, :default => nil
    add_column :users, :activation_token, :string, :default => nil
    add_column :users, :activation_token_expires_at, :datetime, :default => nil

    add_index :users, :activation_token
  end
end
```

    rake db:migrate


### Mongoid

For mongoid just add these three fields to your User model :

```ruby
field :activation_state,            type: String
field :activation_token,            type: String
field :activation_token_expires_at, type: DateTime
```

_Note that the token_expires_at field is only needed if you have setup user.activation_token_expiration_period in the sorcery initializer._

In addition you can add an index on the token field :

```ruby
index({ activation_token: 1 }, { unique: true, background: true })
```

    $ rake db:mongoid:create_indexes


***


Now our database is ready, we need a mailer with two actions:

    $ rails g mailer UserMailer activation_needed_email activation_success_email

You really don't have to use ActionMailer. You can use any ruby object that responds to the above actions. Actually these method names are configurable too! This is useful in case you want to send emails in the background, with gems such as 'delayed_job'. Simply pass your own 'mailer' and make it create the background jobs as you normally would, when triggered by Sorcery.

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
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:user_activation, bla, bla, ...]
```

And configure the User model with our mailer defined:

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.configure do |config|
  ...
  config.user_config do |user|
    ...
    user.user_activation_mailer = UserMailer
    ...
  end
end
```

Add the routes accordingly:

```ruby
# config/routes.rb
resources :users do
  member do
    get :activate
  end
end
```

Now when a user is created, an email will be sent to him. Currently it's not a very informative email, but we will fix that:

```ruby
# app/mailers/user_mailer.rb
def activation_needed_email(user)
  @user = user
  @url  = activate_user_url(@user.activation_token)
  mail(to: user.email, subject: 'Welcome to My Awesome Site')
end

def activation_success_email(user)
  @user = user
  @url  = login_url
  mail(to: user.email, subject: 'Your account is now activated')
end
```

```ruby
# app/views/user_mailer/activation_needed_email.text.erb
Welcome to example.com, <%= @user.email %>
===============================================

You have successfully signed up to example.com,
your username is: <%= @user.email %>.

To login to the site, just follow this link: <%= @url %> .

Thanks for joining and have a great day!
```

Here is the equivalent in html:

```ruby
# app/views/user_mailer/activation_needed_email.html.erb
<!DOCTYPE html>
<html>
  <head>
    <meta content="text/html; charset=UTF-8" http-equiv="Content-Type" />
  </head>
  <body>
    <h1> Welcome <%= @user.email %> </h1>
    <p>
      You have successfully signed up to www.example.com, and your username is: <%= @user.email %>.
    </p>
    To login to the site, just follow this link: <%= @url %> .
    <p>
    </p>
    <p>Thanks for joining and have a great day!</p>    
  </body>
</html>
```

```ruby
# app/views/user_mailer/activation_success_email.text.erb
Congratulations, <%= @user.email %>!

You have successfully activated your example.com account,
your username is: <%= @user.email %>.

To login to the site, just follow this link: <%= @url %>.

Thanks for joining and have a great day!
```

Nice. The user will get an activation URL, that won't work because we didn't add the code for it. Let's add a controller action that will activate users:

```ruby
# app/controllers/users_controller.rb
skip_before_action :require_login, :only => [:index, :new, :create, :activate]

def activate
  if @user = User.load_from_activation_token(params[:id])
    @user.activate!
    redirect_to(login_path, :notice => 'User was successfully activated.')
  else
    not_authenticated
  end
end
```


**@user.activate!** will make the user active, send a success email and clear the activation code.

If you don't want a success email, in the sorcery configuration of the model, set `activation_success_email_method_name` to *nil*.

You can set various other options for this submodule. See the docs for details.

To resend the activation email after user changes email address, add following callbacks to the user model:
```ruby
# app/models/user.rb
before_update :setup_activation, if: -> { email_changed? }
after_update :send_activation_needed_email!, if: -> { previous_changes["email"].present? }
```