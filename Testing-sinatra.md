First include the following in your TestCase or 'describe' block:
```ruby
include Sorcery::TestHelpers::Sinatra
```

Now in your 'setup' or 'before(:each)' block make sure to call this line:
```ruby
::Sinatra::Application.rack_test_session = rack_test_session # Sinatra::Base should work too
```

This helps us access the session in a context outside of a request.

Now you can use the following helpers in your tests:
```
login_user(user = nil) # by default looks for @user
logout_user
```

For a working example see example sinatra app.
