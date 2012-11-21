Note: This will not work if you also using external authentication (OAuth).

I recommend following almost all the steps on the [[User Activation]] example.  But then modify things slightly as follows.

Fix the user model so that the activation emails will be sent even if no password is stored.

```ruby
class User < ActiveRecord::Base
  authenticates_with_sorcery!

  # fix sorcery so that we can activate without a password
  before_create :setup_activation
  def external?
    false
  end
end
```