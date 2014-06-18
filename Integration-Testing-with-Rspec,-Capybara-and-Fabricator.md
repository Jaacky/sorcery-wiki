If you want to write integration tests with Sorcery, Capybara, and Fabricator you will need to login using Capybara.

### Create a test helper module 

Somewhere in your spec directory add the following module. `spec/support` is a good choice. The `user_sessions_url` is the generated path to your UserSessionsController.

```ruby
module Sorcery
  module TestHelpers
    module Rails
      def login_user_post(user, password)
        page.driver.post(user_sessions_url, { username: user, password: password}) 
      end
    end
  end
end
```

### Include the helper module

In `spec/spec_helper.rb`, include the test helper module.

```ruby
RSpec.configure do |config|
  config.include Sorcery::TestHelpers::Rails::Controler, type: [:controller]
  config.include Sorcery::TestHelpers::Rails::Integration, type: [:feature]
end
```
### Create a user fabrication

Create a user_fabricator.rb file in your `spec/fabricators` directory.

```ruby
Fabricator(:user, :class_name => "User") do
  id { sequence }
  username { "admin" }
  password { "admin" }
  display_name { "Admin Boom"}
  admin { true }
  email { "whatever@whatever.com" }
  salt { "asdasdastr4325234324sdfds" }
  crypted_password { Sorcery::CryptoProviders::BCrypt.encrypt("secret", 
                     "asdasdastr4325234324sdfds") }
end
```

### Write your test

You can now invoke your helper module to login a user with Capybara. `current_user` will be available in your controller code when visiting pages.

```ruby
describe "Integration Test" do
  let!(:user) { Fabricate(:user) }

  before(:each) do
    login_user_post("admin", "admin")
  end

  context "when I visit a page" do
    it "show awesome things" do
      #Test stuff
    end
  end
end
```