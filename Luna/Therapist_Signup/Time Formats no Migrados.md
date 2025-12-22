# Time Formats no Migrados en AppBlend

Estos no fueron migrados a backend:
```ruby
Time::DATE_FORMATS[:luna] = '%m/%d/%Y'
Time::DATE_FORMATS[:luna_dash] = '%m-%d-%Y'
Time::DATE_FORMATS[:attachment_folder] = '%Y-%m-%d'

Date::DATE_FORMATS[:luna] = '%m/%d/%Y'
```

Sus reemplazos son con `DateTimeLocalizer` si se quiere;

Para `:luna` se reemplaza con:
```ruby
DateTimeLocalizer.new(DATE).american
```