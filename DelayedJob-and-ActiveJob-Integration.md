By default emails are sent synchronously. You can send them asynchronously by
using the [delayed_job gem](https://github.com/collectiveidea/delayed_job) or
[Rails' ActiveJob](https://edgeguides.rubyonrails.org/active_job_basics.html).

After implementing the `delayed_job` or `ActiveJob` into your project add the
code below at the end of the `config/initializers/sorcery.rb` file. After that
all emails will be sent asynchronously.

```ruby
module Sorcery
  module Model
    module InstanceMethods
      def generic_send_email(method, mailer)
        config = sorcery_config

        # DelayedJob
        mail = config.send(mailer).delay.send(config.send(method), self)

        # ActiveJob
        mail = config.send(mailer).send(config.send(method), self).deliver_later
      end
    end
  end
end
```
