**paper_trail** - Sorcery _versions v0.6.1 and previous_ do not play nice with paper_trail due to a before_filter that paper_trail adds behind the scenes that calls 'current_user' a bit too early for sorcery. This issue *might* be solved by calling
```
    prepend_before_filter :require_login
```

instead of the usual
```
    before_filter :require_login
```

Though this hasn't been proven (please report success).


**refinerycms**


```ruby
    # Gemfile  # works
    gem 'sorcery'
    gem 'refinerycms'
```


The 'sorcery' gem need to be placed before 'rerinferycms' gem, otherwise it won't work.

under the hood, for example

```ruby
    # Gemfile  # don't work in production mode
    gem 'refinerycms'
    gem 'sorcery'
```

when require 'refinerycms', it loads User model (I don't know how), which calls `User.authenticates_with_sorcery!`,  but it's not ready (means [user_config](https://github.com/NoamB/sorcery/blob/master/lib/sorcery/initializers/initializer.rb#L61) is not [loaded](https://github.com/NoamB/sorcery/blob/master/lib/sorcery/controller.rb#L15)).  We need first require 'sorcery', which call [ ActionController::Base.send(:include, Sorcery::Controller)](https://github.com/NoamB/sorcery/blob/master/lib/sorcery/engine.rb#L11),  it prepares user_config for `User.authenticates_with_sorcery!`.

**rails_admin**

List of potential problems and suggested solutions are provided in (RailsAdmin wiki)[https://github.com/sferik/rails_admin/wiki/Sorcery]