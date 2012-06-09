You can setup fixtures like this:

```erb
noam:
  email: whatever@whatever.com
  salt: <%= salt = "asdasdastr4325234324sdfds" %>
  crypted_password: <%= Sorcery::CryptoProviders::BCrypt.encrypt("secret", salt) %>
  activation_state: active
```


In your test/spec helper make sure to include:

```ruby
include Sorcery::TestHelpers::Rails
```

And now you can use the following helpers:

```ruby
login_user(user = nil) # by default looks for @user
logout_user
```


See the example app for a working example.


