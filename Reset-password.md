In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

** Note: 'reset_password!(:password => "secret")' changed into 'change_password!(new_password)' in v0.5.0 **

First Add some db fields:

    rails g sorcery_migration reset_password

Which will create:

```ruby
class SorceryResetPassword < ActiveRecord::Migration
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
  skip_before_filter :require_login
    
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
    @user = User.load_from_reset_password_token(params[:id])
    @token = params[:id]
    not_authenticated if !@user
  end
      
  # This action fires when the user has sent the reset password form.
  def update
    @user = User.load_from_reset_password_token(params[:token])
    not_authenticated if !@user

    # TODO: validate that params[:user][:password] == params[:user][:password_confirmation]
    # if not, send an error to the user to correct it

    # the next line clears the temporary token and updates the password
    if @user.change_password!(params[:user][:password])
      redirect_to(root_path, :notice => 'Password was successfully updated.')
    else
      render :action => "edit"
    end
  end
end
```

Add the rest:

```ruby
# config/routes.rb
resources :password_resets
```

```rhtml
# app/views/user_mailer/reset_password.text.erb
Hello, <%= @user.email %>
===============================================
 
You have requested to reset your password.

To choose a new password, just follow this link: <%= @url %>.
 
Have a great day!
```

```ruby
# app/mailers/user_mailer.rb
def reset_password_email(user)
  @user = user
  @url  = "http://0.0.0.0:3000/password_resets/#{user.reset_password_token}/edit"
  mail(:to => user.email,
       :subject => "Your password has been reset")
end
```

Now we need some work on the reset password views:

```rhtml
# app/views/password_resets/_form.html.erb
<%= form_for @user, :url => password_reset_path(@user), :html => {:method => :put} do |f| %>
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
    <%= hidden_field_tag :token, @token %>
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

So now in the login form a user can put his email in the 'forgot password' form, get instructions to his email with a link. With that link, we get to a form where the user can enter a new password, and from there he is set.