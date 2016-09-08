## Preparing local instance

1. Fork Sorcery on Github
2. Clone your forked copy onto your working machine
  * Open terminal and `cd` into where you would like the repository saved.
  * Run the clone command, example: `git clone https://github.com/<your_user_name>/sorcery.git`
3. With the same terminal, `cd` into your freshly made repository using: `cd ./sorcery`
4. Add upstream to remotes
  * `git remote add upstream https://github.com/NoamB/sorcery.git`
5. Run `bundle install`
6. To ensure everything is ready, run `bundle exec rspec spec`, all tests should pass.

## Merging your contributions

1. Make your changes, adding additional specs if you are adding functionality.
2. Run `bundle exec rspec spec` to ensure your changes didn't break other functionality.
3. `git commit` with a relevant message
4. Pull down any updates
  1. `git pull --rebase upstream master`
  2. If there are any conflicts, resolve them via text editor then run:
    * `git add -A` then `git rebase --continue`
4. `git push --force-with-lease`
  * (WARNING: This will delete any history on your github fork and overwrite it with your local copy. This is necessary however if history was rewritten due to changes to the main repository.)
4. Submit a pull-request via github.


This page is a work in progress; if you have any suggestions or see any mistakes, please update the page as needed!