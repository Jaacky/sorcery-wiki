Note: This will not work if you also using external authentication (OAuth).

I recommend following almost all the steps on the [[User Activation]] example.  But then modify things slightly as follows.

Fix the user model so that the activation emails will be sent even if no password is stored.

```ruby
class User < ActiveRecord::Base
  authenticates_with_sorcery!

  # fix sorcery so that we can activate without a password
  before_create :setup_activation
  def external?
    false
  end
end
```

Add a route for the confirmation to take place.

```ruby
# config/routes.rb
resources :users do
  member do
    get :activate
    put :confirm
  end
end
```

Add a confirmation action, and modify the activation action.

```ruby
# app/controllers/users_controller.rb
skip_before_filter :require_login, :only => [:index, :new, :create, :activate]

def activate
  if (@user = User.load_from_activation_token(params[:id]))
    @user.activate!
    password = TemporaryToken.generate_random_token
    @user.password = password
    @user.password_confirmation = password
    @user.save
  else
    not_authenticated
  end
end

def confirmation
  @user = User.find params[:id]
  if @user.update_attributes(params[:user])
    redirect_to @user, :notice => 'User was successfully activated.'
  else
    render :activate
  end
end
```