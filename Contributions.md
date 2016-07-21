## Preparing local instance

1. Fork Sorcery on Github
2. Clone your forked copy onto your working machine
  * Open terminal and `cd` into where you would like the repository saved.
  * Run the clone command, example: `git clone https://github.com/<your_user_name>/sorcery.git`
3. With the same terminal, `cd` into your freshly made repository using: `cd ./sorcery`
4. Run `bundle install`
5. To ensure everything is ready, run `bundle exec rspec spec`

## Merging your contributions

1. Make your changes, adding additional specs if you are adding functionality.
2. Run `bundle exec rspec spec` to ensure your changes didn't break other functionality.
3. `git commit` with a relevant message, and `git push` to your forked copy.
4. Submit a pull-request via github.


This page is a work in progress; if you have any suggestions or see any mistakes, please update the page as needed!