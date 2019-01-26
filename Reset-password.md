In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

** Note: `reset_password!(:password => "secret")` changed into `change_password!(new_password)` in v0.5.0 **

First Add some db fields:

    rails g sorcery:install reset_password --only-submodules

Which will create:

```ruby
class SorceryResetPassword < ActiveRecord::Migration
  def change
    add_column :users, :reset_password_token, :string, :default => nil
    add_column :users, :reset_password_token_expires_at, :datetime, :default => nil
    add_column :users, :reset_password_email_sent_at, :datetime, :default => nil
  end
end
```

And then:

    rake db:migrate


Add a mailer with one action:

    rails g mailer UserMailer reset_password_email

We need to edit the mailer and add a 'user' parameter to the action because sorcery will send it the new user as a parameter:

```ruby
# app/mailers/user_mailer.rb
def reset_password_email(user)
...
```

Then add the reset_password submodule, and define what mailer to use:

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:reset_password, blabla, blablu, ...]

Rails.application.config.sorcery.configure do |config|
  config.user_config do |user|
    user.reset_password_mailer = UserMailer
  end
end
```

We need a controller to handle all the password action:

    rails g controller PasswordResets create edit update

Make it look like this:

```ruby
# app/controllers/password_resets_controller.rb
class PasswordResetsController < ApplicationController
  # In Rails 5 and above, this will raise an error if
  # before_action :require_login
  # is not declared in your ApplicationController.
  skip_before_action :require_login
    
  # request password reset.
  # you get here when the user entered his email in the reset password form and submitted it.
  def create 
    @user = User.find_by_email(params[:email])
        
    # This line sends an email to the user with instructions on how to reset their password (a url with a random token)
    @user.deliver_reset_password_instructions! if @user
        
    # Tell the user instructions have been sent whether or not email was found.
    # This is to not leak information to attackers about which emails exist in the system.
    redirect_to(root_path, :notice => 'Instructions have been sent to your email.')
  end
    
  # This is the reset password form.
  def edit
    @token = params[:id]
    @user = User.load_from_reset_password_token(params[:id])

    if @user.blank?
      not_authenticated
      return
    end
  end
      
  # This action fires when the user has sent the reset password form.
  def update
    @token = params[:id]
    @user = User.load_from_reset_password_token(params[:id])

    if @user.blank?
      not_authenticated
      return
    end

    # the next line makes the password confirmation validation work
    @user.password_confirmation = params[:user][:password_confirmation]
    # the next line clears the temporary token and updates the password
    @user.change_password!(params[:user][:password])
    redirect_to(root_path, :notice => 'Password was successfully updated.')
  rescue ActiveRecord::RecordInvalid
    render :action => "edit"
  end
end
```

Add the rest:

```ruby
# config/routes.rb
resources :password_resets, only: [:create, :edit, :update]
```

```rhtml
# app/views/layouts/mailer.html.erb
= yield
```

```rhtml
# app/views/user_mailer/reset_password_email.text.erb
Hello, <%= @user.email %>
===============================================
 
You have requested to reset your password.

To choose a new password, just follow this link: <%= @url %>.
 
Have a great day!
```

```ruby
# app/mailers/user_mailer.rb
def reset_password_email(user)
  @user = User.find user.id
  @url  = edit_password_reset_url(@user.reset_password_token)
  mail(:to => user.email,
       :subject => "Your password has been reset")
end
```

Now we need some work on the reset password views:

```rhtml
# app/views/password_resets/edit.html.erb
<h1>Choose a new password</h1>
<%= form_for @user, :url => password_reset_path(@token), :html => {:method => :put} do |f| %>
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>
    
      <ul>
      <% @user.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
    
  <div class="field">
    <%= f.label :email %><br />
    <%= @user.email %>
  </div>
  <div class="field">
    <%= f.label :password %><br />
    <%= f.password_field :password %>
  </div>
  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

I like to put the "forgot password?" form in the same page as the login form:

```rhtml
# app/views/user_sessions/new.html.erb
...
<h1>Forgot Password?</h1>
<%= render 'forgot_password_form' %>
```

```rhtml
# app/views/user_sessions/_forgot_password_form.html.erb
<%= form_tag password_resets_path, :method => :post do %>
  <div class="field">
    <%= label_tag :email %><br />
    <%= text_field_tag :email %> <%= submit_tag "Reset my password!" %>
  </div>
<% end %>
```

Alternatively you could create a "new" view and link to it from the login page:

```rhtml
# app/views/user_sessions/new.html.erb
...
<%= link_to 'Forgot Password?', new_password_reset_path %>
```

```rhtml
# app/views/password_resets/new.html.erb
<%= form_tag password_resets_path, :method => :post do %>
  <div class="field">
    <%= label_tag :email %><br />
    <%= text_field_tag :email %> <%= submit_tag "Reset my password!" %>
  </div>
<% end %>
```

So now in the login form a user can put his email in the 'forgot password' form, get instructions to his email with a link. With that link, we get to a form where the user can enter a new password, and from there he is set.