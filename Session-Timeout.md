First add this submodule to sorcery:

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:session_timeout]
```
Session timeout is configurable like this:

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.configure do |config|
  config.session_timeout 3600 # This is in seconds. You could also write 1.hour
  config.session_timeout_from_last_action = true # session timeout is calculated from the last valid activity. By default this is false.
end
```
