Important notes:

- Expected to work with DM adapters: `dm-mysql-adapter`, `dm-redis-adapter`.
- Submodules DM adapter dependent: `activity_logging` (`dm-mysql-adapter`)
- Usage: include `DataMapper::Resource` in user model, follow sorcery instructions (remember to add property id, validators and accessor attributes such as password and password_confirmation)
- Option downcase_username_before_authenticating and dm-mysql. See this [article](http://datamapper.lighthouseapp.com/projects/20609/tickets/1105-add-support-for-definingchanging-default-collation).