# Settings to Apply for load_defaults

In order to activate `config.load_defaults("7")` these configs need to be set:

```ruby
module LunaApi
	class Application < Rails::Application
		config.active_record.has_many_inversing = false
		config.active_support.use_rfc4122_namespaced_uuids = false
		config.active_storage.queues.analysis = :active_storage_analysis
		config.active_storage.queues.purge = :active_storage_purge
		config.active_storage.track_variants = false
	end
end
```

See:

- has_many_inversing [[Browsing Batch Loader in Rails 7.0.4#A workaround // fix]]
- use_rfc4122_namespaced_uuids [[Upgrade to Rails 7.0.4 - Notes#Only UUIDs are valid namespace identifiers]]
- queues.analysis [[004 - AppBlend What Changes in Config Defaults#What is config.active_storage.queues.analysis?]]
- queues.purge [[004 - AppBlend What Changes in Config Defaults#What is config.active_storage.queues.purge?]]
- track_variants [[004 - AppBlend What Changes in Config Defaults#What is config.active_storage.track_variants?]]