This tutorial shows how to use Rails routes constraints with Sorcery gem. Thanks to [@anthonator](https://github.com/anthonator) for writing it!


First, define `UserConstraint` module that will be used for all constraints:

```ruby
module RouteConstraints::UserConstraint
  def current_user(request)
    User.find_by_id(request.session[:user_id])
  end
end
```

Then, having that module defined, you can specify specific constraint classes. In these examples, first route will work only if there's no user logged in, the second will work only for logged user who is an admin:

```ruby
class RouteConstraints::NoUserRequiredConstraint
  include RouteConstraints::UserConstraint

  def matches?(request)
    !current_user(request).present?
  end
end
```

```ruby
class RouteConstraints::AdminRequiredConstraint
  include RouteConstraints::UserConstraint

  def matches?(request)
    user = current_user(request)
    user.present? && user.is_admin?
  end
end
```

Finally, you can add the constraints to the `config/routes.rb`:

```ruby
MyApp::Application.routes.draw do

  # other routes …

  root :to => 'admin#dashboard', :constraints => RouteConstraints::AdminRequiredConstraint.new
  root :to => 'home#welcome', :constraints => RouteConstraints::NoUserRequiredConstraint.new

end
```