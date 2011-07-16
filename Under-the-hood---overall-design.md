In this page I'll try to explain the overall design of the Sorcery lib.
You are welcome to improve this page and make it clearer (this is true for the whole wiki).

As you can see under 'lib' the logic is divided into 'Model' and 'Controller'.
model.rb extends the model class that you call 'authenticates_with_sorcery!' in.
controller.rb extends the base controller, which is ActionController::Base in Rails.
These files are referred to in the code as the 'core'.

Also, each one of them has a subdirectory called 'submodules'. A submodule is a ruby Module which can be optionally mixed-in into the model/controller, to add a specific feature, like "Remember Me".
The mixing-in happens if the user states he would like to use this module in the beginning of the initializer file.

```
Rails.application.config.sorcery.submodules = [:remember_me]
```

To be continued...