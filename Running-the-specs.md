The default 'rake' task from the gem root folder runs all specs.
That includes specs for all of Sorcery supported platforms and ORMs (Rails 3, Sinatra, ActiveRecord, Mongoid, Sinatra Modular style etc.).

The ActiveRecord specs expect a local MySQL server present at the default port.

The Mongoid specs expect a local Mongoid server present at the default port.