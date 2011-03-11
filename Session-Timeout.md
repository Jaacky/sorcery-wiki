First add this submodule to sorcery:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:session_timeout]
```
Session timeout is configurable from the controller:
```ruby
    # app/controllers/application_controller.rb
    activate_sorcery! do |config|
      config.session_timeout 3600 # This is in seconds. You could also write 1.hour
      config.session_timeout_from_last_action = true # session timeout is calculated from the last valid activity. By default this is false.
    end
```

You could also configure these options at application.rb, like so:
```ruby
    # config/application.rb
    config.sorcery.submodules = [:session_timeout]
    config.sorcery.session_timeout = 3600
```
