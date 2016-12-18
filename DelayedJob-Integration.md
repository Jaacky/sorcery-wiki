By default emails are sent synchronously. You can send them asynchronously by
using the [delayed_job gem](https://github.com/collectiveidea/delayed_job).

After implementing the `delayed_job` into your project add the code below at
the end of the `config/initializers/sorcery.rb` file. After that all emails
will be sent asynchronously.

```ruby
module Sorcery
  module Model
    module InstanceMethods
      def generic_send_email(method, mailer)
        config = sorcery_config
        mail = config.send(mailer).delay.send(config.send(method), self)
      end
    end
  end
end
```