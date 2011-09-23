**Rubies** - Sorcery was tested on ruby 1.9.2 and should be fully compatible with it.
Other rubies were not yet tested, though they might be in the future.

**Rails** - Rails 3+ is supported. There are currently no plans to support Rails 2.

**Single Table Inheritance** - This ActiveRecord feature doesn't play nice with Sorcery out of the box, but there is a hack that seems to work, and will probably get into a future release (**Update**: already supported).

What we want to do is copy the configuration of User into its subclasses. This doesn't happen by default because @sorcery_config is attached to the User singleton object.

The solution is this:

```ruby
class User < ActiveRecord::Base
  ...
  
  def self.inherited(subclass)
    subclass.class_eval do
      class << self
        attr_accessor :sorcery_config
      end
    end
    subclass.sorcery_config = sorcery_config
    super
  end
  authenticates_with_sorcery!
  
  ...
end
```


**paper_trail** - Sorcery versions v0.6.1 and previous do not play nice with paper_trail due to a before_filter that paper_trail adds behind the scenes that calls 'current_user' a bit too early for sorcery. This issue *might* be solved by calling
```
    prepend_before_filter :require_login
```

instead of the usual
```
    before_filter :require_login
```

Though this hasn't been proven (please report success).