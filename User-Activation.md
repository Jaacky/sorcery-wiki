Under construction - incomplete!

First add the user_activation submodule:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:user_activation, blabla, blablu, ...]
```

Now when a user is created, an email will be sent to him with activation instructions.
**@user.activate!** will make the user active, send success email if configured and clear the activation code.