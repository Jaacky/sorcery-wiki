After checking out the gem, I usually start by creating a gemset with rvm:

```
    rvm use 1.9.2@sorcery --create
```

Then I install bundler

```
    gem install bundler
```

Then run

```
    bundle install
```

Now you have a few important rake tasks at your disposal:

`rake bundle` - will run 'bundle install' recursively on all subfolders which have a Gemfile.

`rake bundle_update` - will run 'bundle update' recursively.

The default `rake` task from the gem root folder runs all specs.
That includes specs for all of Sorcery supported platforms and ORMs (Rails 3, ActiveRecord, Mongoid, MongoMapper etc.).

The ActiveRecord specs expect a local MySQL server present at the default port.

The Mongoid/MongoMapper specs expect a local Mongoid server present at the default port.
