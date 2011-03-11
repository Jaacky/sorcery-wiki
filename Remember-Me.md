In this tutorial we will build upon the app created at [[Simple Password Authentication]] so make sure you understand it.

First add the remember_me submodule:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:remember_me, blabla, blablu, ...]
```

Adding remember me to the app is very simple - just use the good old **login** method with a third parameter.
This is usually the result of a "Remember me" check box from a form.
If it's anything that evals to true, it will 'remember' the user.
```ruby
    # app/controllers/user_sessions_controller.rb
    @user = login(params[:username],params[:password],params[:remember])
```

To "forget me" just **logout**.  
  
If you ever need finer control you can use the controller methods:
```ruby
    remember_me!
    forget_me!
```

There are options to configure expiration time and more. See the gem docs for a full list.