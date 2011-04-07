First we'll require sinatra and enable sessions:
```ruby
    require 'sinatra'
    enable :sessions
```

Sorcery comes in like this (order IS important):
```ruby
    require 'sorcery'
    Sorcery::Controller::Config.submodules = [:user_activation, :http_basic_auth, :remember_me, :reset_password, :session_timeout, :brute_force_protection, :activity_logging, :oauth]
    include Sorcery::Controller::Adapters::Sinatra
    include Sorcery::Controller
```

Do controller configuration in the main app file:
```ruby
    Sinatra::Application.activate_sorcery! do |config|
      ...
    end
```

Before filters can be added like this:
```ruby
    # filters
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