### Controller

The core controller API is extremely short and simple:

> require_user_login # a before filter for you to use in controllers which require authentication.

> login(*credentials) => authenticated user or nil if not found

> logout

> logged_in? => true or false

> logged_in_user => authenticated user or false if not logged in

### View Helpers

> logged_in_user => runs the controller method of the same name.

### Configuration

Used like this in the model/controller:

```ruby
activate_sorcery! do |config|
  config.<some_config_option> = <some_value>
end
```

For the specific options available in the model/controller please see the documentation.