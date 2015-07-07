You can setup fixtures like this:

```erb
noam:
  email: whatever@whatever.com
  salt: <%= salt = "asdasdastr4325234324sdfds" %>
  crypted_password: <%= Sorcery::CryptoProviders::BCrypt.encrypt("secret", salt) %>
  activation_state: active
```

# TestCase
For simple out of the box testing:
```ruby
class TeamsControllerTest < ActionController::TestCase
  include Sorcery::TestHelpers::Rails::Integration
  include Sorcery::TestHelpers::Rails::Controller

  setup do
    @team = teams(:one)
    @user = users(:one)

    login_user(user = @user, route = login_url)  # replace with your login url path
  end

  test "should get index" do
    get :index
    assert_response :success
    assert_not_nil assigns(:teams)
  end
...
```

# Functional Tests

In your test/spec helper make sure to include:

```ruby
RSpec.configure do |config|
  config.include Sorcery::TestHelpers::Rails::Controller, type: :controller
  config.include Sorcery::TestHelpers::Rails::Integration, type: :feature
end
```

And now you can use the following helpers:

```ruby
login_user(user = nil) # by default looks for @user
logout_user
```


See [the example app](https://github.com/NoamB/sorcery-example-app/blob/master/test/functional/users_controller_test.rb) for a working example.

# Feature Tests w/ Rack::Tests

In `spec/support/authentication.rb`

```ruby
module AuthenticationForFeatureRequest
  def login user, password = 'login'
    user.update_attributes password: password

    page.driver.post sessions_url, {email: user.email, password: password}
    visit root_url
  end
end
```

Note that you need to update password with `update_attributes` or `update_attributes!`, not with `update_attribute` because generating salt and encrypting password are triggered by `before_validation` since v0.9.0 but `update_attribute` skips validations.

In `spec/spec_helper.rb`

```ruby
RSpec.configure do |config|
  # ...
  config.include AuthenticationForFeatureRequest, type: :feature
  # ...
end
```

In `spec/features/your_feature_spec.rb`

```ruby
feature 'Your Feature' do
  let(:user) { Fabricate :user }
  scenario 'Your Scenario' do
    login user
  end
end
```

# Feature Tests w/ Selenium or Webkit

Please add working examples if you got any. The Rack::Test solution doesn't work since `page.driver.post` is not available.

