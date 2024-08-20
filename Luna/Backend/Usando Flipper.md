# Usa Flipper para feature flags

Basically,  start by setting up a feature flag. If the feature flag hasn't been activated, querying `Flipper.enabled?(:feature_flag)` will return `false`.  To activate the feature flag from the Rails console, you can use:  

```ruby
Flipper.enable(:feature_flag)
```

After that, when you check `Flipper.enabled?(:feature_flag)`, it will return `true`.  
If you need to disable the feature flag, you can simply run:  

```ruby
Flipper.disable(:feature_flag)
```

Example:
```ruby
Flipper.enable(:patient_forms_v3_url)
Flipper.disable(:patient_forms_v3_url)
```


## Links

- Flipper in [GitHub](https://github.com/flippercloud/flipper)
- Flipper [Docs](https://www.flippercloud.io/docs/features)