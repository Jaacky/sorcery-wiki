Important notes while upgrading:

- If you are upgrading from **<= 1.0.0**
    - `before_logout` does not take arguments anymore (`current_user` still returns user at this point)
    - `after_logout` takes one argument (`user`) as `current_user` returns `nil` then
- If you are upgrading from **<= 0.8.6** and you use Sorcery model methods in your app, you might need to change them from `user.method` to `user.sorcery_adapter.method` and from `User.method` to `User.sorcery_adapter_method`
- If you are upgrading from **<= 0.8.5** and you're using Sorcery test helpers, you need to change the way you include them to following code:

    ```ruby
    RSpec.configure do |config|
      config.include Sorcery::TestHelpers::Rails::Controller, type: :controller
      config.include Sorcery::TestHelpers::Rails::Integration, type: :feature
    end
    ```

- If are upgrading to **0.8.2** and use activity_logging feature with ActiveRecord, you will have to add a new column `last_login_from_ip_address` [#465](https://github.com/NoamB/sorcery/issues/465)
- Sinatra support existed until **0.7.0** (including), but was dropped later due to being a maintenance nightmare.
- If upgrading from **<= 0.6.1** to **>= 0.7.0** you need to change 'username_attribute_name' to 'username_attribute_names' in initializer.
- If upgrading from **<= v0.5.1** to **>= v0.5.2** you need to explicitly
set your user_class model in the initializer file.

```ruby
# This line must come after the 'user config' block.
config.user_class = User
```