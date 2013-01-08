Note: This will not work if you also using external authentication (OAuth).

I recommend following almost all the steps on the [[User Activation]] example.  But then modify things slightly as follows.

Fix the user model so that the activation emails will be sent even without password at the creation.

```ruby
class User < ActiveRecord::Base
  authenticates_with_sorcery!

  # change the validations
  validates :password, :presence => true, :confirmation => true, :on => :update
  validates :first_name, :presence => true, :on => :update
  validates :last_name,  :presence => true, :on => :update

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
You should have @user.activate! within activate, we gonna move that line to confirm so the account is only activated if the informations are given.

```ruby
# app/controllers/users_controller.rb
skip_before_filter :require_login, :only => [:index, :new, :create, :activate, :confirm]

def activate
  if (@user = User.load_from_activation_token(params[:id]))
    # no action needed here
  else
    not_authenticated
  end
end

def confirm
  @user = User.find params[:id]
  if @user.update_attributes(params[:user])
    @user.activate!
    redirect_to login_url, :notice => 'Your account is now activated.'
  else
    render :activate
  end
end
```

Here I redirect the user to the login page, but you can let the app handle that part:

```ruby
def confirm
  @user = User.find params[:id]
  if @user.update_attributes(params[:user])
    @user.activate!
    auto_login @user
    redirect_to @user, notice: 'Your account is now activated.'
  else
    render :activate
  end
end
```
Now just add the activation form (I use formtastic, but the standard form builder works fine).

```haml
# app/view/users/activate.html.haml
%h1 Activate

= semantic_form_for @user, :url => confirm_user_path(@user) do |f|
  = f.inputs do
    = f.input :first_name
    = f.input :last_name
    = f.input :password
    = f.input :password_confirmation
  = f.actions :submit
```