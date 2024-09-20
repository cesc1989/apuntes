# What changes between config defaults?

Identify changes between the ==config defaults 6.0 and 7.0==.

See: https://guides.rubyonrails.org/v7.0/configuring.html

The section says:
> [config.load_defaults](https://api.rubyonrails.org/v7.0.8.4/classes/Rails/Application/Configuration.html#method-i-load_defaults) loads default configuration values for a target version and all versions prior. For example, `config.load_defaults 6.1` will load defaults for all versions up to and including version 6.1.

Caught my eye:
- Default Values for Target Version 6.1
	- [config.active_record.has_many_inversing](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-record-has-many-inversing): `true`
		- This gave me trouble with batch loader [[Browsing Batch Loader in Rails 7.0.4#A workaround // fix]]

And Anthony wants me to understand these:

- [config.active_storage.queues.analysis](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-analysis)
- [config.active_storage.queues.purge](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-purge)
- [config.active_storage.track_variants](https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-track-variants)

# What is config.active_storage.queues.analysis?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-analysis

![[as.queues.analysis.png]]

Once `config.load_defaults 7.0`, I have to setup:

```ruby
config.active_storage.queues.analysis = :active_storage_analysis
```

# What is config.active_storage.queues.purge?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-queues-purge

![[as.queues.purge.png]]

Once `config.load_defaults 7.0`, I have to setup:

```ruby
config.active_storage.queues.purge = :active_storage_purge
```

# What is config.active_storage.track_variants?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-storage-track-variants

![[as.track_variants.png]]

Once `config.load_defaults 7.0`, I have to setup:

```ruby
config.active_storage.track_variants = false
```

# What is config.active_record.legacy_connection_handling?

See https://guides.rubyonrails.org/v7.0/configuring.html#config-active-record-legacy-connection-handling

![[ar.legacy_connection_handling.png]]

Once `config.load_defaults 7.0`, I have to setup:

```ruby
config.active_record.legacy_connection_handling = true
```

More context on this setting -> https://guides.rubyonrails.org/v7.0.4/active_record_multiple_databases.html#migrate-to-the-new-connection-handling

> In Rails 6.1+, Active Record provides a new internal API for connection management. In most cases applications will not need to make any changes ==except to opt-in to the new behavior (if upgrading from 6.0 and below==) by setting [`config.active_record.legacy_connection_handling`](https://guides.rubyonrails.org/v7.0.4/configuring.html#config-active-record-legacy-connection-handling) to `false`.

Looks like it needs to be turned on because in Rails 7 `legacy_connection_handling` is deprecated. See [[002 - Important Updates per Changelogs#7.0.0 ✅]].

When Edge was in Rails 6.1.7.8, `config.load_defaults` was set to 6.0 so `legacy_connection_handling` was true. However, once I changed `config.load_defaults` to 7.0 it was set to false. Thus the error indicated below.

If not enabled, it produces the error [[Pruebas de Rails 7#while_preventing_writes is only available on the connection_handler with legacy_connection_handling]]