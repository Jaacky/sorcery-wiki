**Sorcery supports both classic style AND modular style since v0.5.2**

First we'll require sinatra and enable sessions:
```ruby
    # myapp.rb
    require 'sinatra'
    enable :sessions
```

Sorcery comes in like this (order IS important):
```ruby
    # myapp.rb
    require 'sorcery'
    require_relative 'sorcery_config'

    # sorcery_config.rb
    Sorcery::Controller::Config.submodules = [:user_activation, :http_basic_auth, :remember_me, :reset_password, :session_timeout, :brute_force_protection, :activity_logging, :oauth]

    Sorcery::Controller::Config.configure do |config|
      ...
      
      config.user_config do |user|
        ...

      end
      ...
 
    end

    include Sorcery::Controller::Adapters::Sinatra
    include Sorcery::Controller
```

Before filters can be added like this:
```ruby
    # myapp.rb
    ['/logout'].each do |patt|
      before patt do
        require_login
      end
    end

    before '/login/http' do
      require_login_from_http_basic
    end
```

For more examples check out the example Sinatra app.