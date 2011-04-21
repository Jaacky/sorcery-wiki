In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

Let's add the submodule and configuration:
```ruby
    # config/application.rb
    Rails.application.config.sorcery.submodules = [:http_basic_auth, blabla, blablu, ...]

    Rails.application.config.sorcery.configure do |config|
      ...
      config.controller_to_realm_map = {"application" => "MySite!"}
    end
```

We've just set a realm. This is the site name the user will see in the modal dialog.
Adding it in the hash for ApplicationController makes it the default for all controllers.

Now we'll add a before filter in the area that we want to protect with basic auth:
```ruby
   # some controller...
   before_filter :require_login_from_http_basic, :only => [:login_from_http_basic]
   
   def login_from_http_basic
     redirect_to users_path, :notice => 'Login from basic auth successful'
   end
```

If this controller uses 'require_login' we'll need to skip it for :login_from_http_basic.

We just need to route the new action we've added to the controller:
```ruby
    resources :users do
      collection do
        get :login_from_http_basic
      end
      ...
    end
```

That's it!