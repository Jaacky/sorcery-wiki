### Controller

The core controller API is extremely short and simple:

> require_user_login # a before filter for you to use in controllers which require authentication.

> login(*credentials) => authenticated user or nil if not found

> logout

> logged_in? => true or false

> logged_in_user => authenticated user or false if not logged in

### View Helpers

> logged_in_user => runs the controller method of the same name.

### Model Configuration

Used like this:

```ruby
activate_sorcery! do |config|
  config.<some_config_option> = <some_value>
end
```


**:username_attribute_name** - change default username attribute, for example, to use :email as the login.


**:password_attribute_name** - change *virtual* password attribute, the one which is used until an encrypted one is

 
**:email_attribute_name** - change default email attribute.


**:crypted_password_attribute_name** - change default crypted_password attribute.


**:salt_join_token** - what pattern to use to join the password with the salt.


**:salt_attribute_name** - change default salt attribute.


**:stretches** - how many times to apply encryption to the password.


**:encryption_key** - encryption key used to encrypt reversible encryptions such as AES256.


**:before_authenticate** - an array of method names to call before authentication completes. used internally.


**:after_config** - an array of method names to call after configuration by user. used internally.


**:encryption_provider** - change default encryption_provider.


**:custom_encryption_provider** - use an external encryption class.


**:encryption_algorithm** - encryption algorithm name. See 'encryption_algorithm=' below for available options.


