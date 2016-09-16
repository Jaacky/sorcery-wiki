As of Sorcery 0.8.7, the `current_users` controller method is deprecated, which means it will be removed in 0.9.0 or 1.0 version. Of course it can be defined by the user. Here are presented possible implementations (the ones that were used in 0.8.6 and earlier versions.)

**ActiveRecord**
```ruby
def self.get_current_users
  config = sorcery_config

  where("#{config.last_activity_at_attribute_name} IS NOT NULL") \
  .where("#{config.last_logout_at_attribute_name} IS NULL 
    OR #{config.last_activity_at_attribute_name} > #{config.last_logout_at_attribute_name}") \
  .where("#{config.last_activity_at_attribute_name} > ? ", config.activity_timeout.seconds.ago.utc.to_s(:db))
end
```

**Mongoid**
```ruby
def self.get_current_users
  config = sorcery_config

  where(config.last_activity_at_attribute_name.ne => nil) \
  .where("this.#{config.last_logout_at_attribute_name} == null
    || this.#{config.last_activity_at_attribute_name} > this.#{config.last_logout_at_attribute_name}") \
  .where(config.last_activity_at_attribute_name.gt => config.activity_timeout.seconds.ago.utc).order_by([:_id,:asc])
end
```

**DataMapper**
```ruby
def self.get_current_users
  unless repository.adapter.is_a?(::DataMapper::Adapters::MysqlAdapter)
    raise 'Unsupported DataMapper Adapter'
  end
  config = sorcery_config
  ret = all(config.last_logout_at_attribute_name => nil) |
        all(config.last_activity_at_attribute_name.gt => config.last_logout_at_attribute_name)
  ret = ret.all(config.last_activity_at_attribute_name.not => nil)
  ret = ret.all(config.last_activity_at_attribute_name.gt => config.activity_timeout.seconds.ago.utc)
  ret
end
```