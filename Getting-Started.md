This guide is currently in the middle of being written, in the interim please use the other wiki examples.

# Part 1 - Generating a new application

If you're installing Sorcery on an existing application, skip ahead to step 2.

1. Run the rails new command: `rails new ~/sorcery-tutorial-app` (path given as example, use whatever is convenient)
2. cd into your application: `cd ~/sorcery-tutorial-app`
3. Initialize git and make the initial commit:
  * `git init`
  * `git add -A`
  * `git commit -m "Initial Commit"`

# Part 2 - Installing Sorcery / Password-based Authentication

1. Add sorcery to the Gemfile:

  ```ruby
  # ./sorcery-tutorial-app/Gemfile
  gem 'sorcery'
  ```

2. Run `bundle` to install sorcery
3. Run `rails g sorcery:install` to install sorcery without any submodules enabled.
4. Run `rake db:migrate` to update the database schema with our new user model.

# Part 3 - Adding OAuth-based Authentication