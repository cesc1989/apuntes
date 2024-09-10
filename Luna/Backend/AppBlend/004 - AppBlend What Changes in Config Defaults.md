# What changes between config defaults?

Identify changes between the config defaults 6.0 and 7.0.

See: https://guides.rubyonrails.org/v7.0/configuring.html

The section says:
> [`config.load_defaults`](https://api.rubyonrails.org/v7.0.8.4/classes/Rails/Application/Configuration.html#method-i-load_defaults) loads default configuration values for a target version and all versions prior. For example, `config.load_defaults 6.1` will load defaults for all versions up to and including version 6.1.

Caught my eye:
- Default Values for Target Version 6.1
	- [`config.active_record.has_many_inversing`](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-record-has-many-inversing): `true`
		- This gave me trouble with batch loader [[Browsing Batch Loader in Rails 7.0.4#A workaround // fix]]

And Anthony wants me to understand these:
- [`config.active_storage.queues.analysis`](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-analysis): `nil`
- [`config.active_storage.queues.purge`](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-purge): `nil`
- [`config.active_storage.track_variants`](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-track-variants): `true`

# What is config.active_storage.queues.analysis?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-analysis

![[as.queues.analysis.png]]

# What is config.active_storage.queues.purge?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-purge

![[as.queues.purge.png]]

# What is config.active_storage.track_variants?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-track-variants

![[as.track_variants.png]]