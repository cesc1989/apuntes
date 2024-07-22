# Upgrade Ruby to 3.1.0 (Without Webpacker)

Notes, errors and solutions when upgrading Clinical Dashboard to Ruby 3.1.0

# Errors

## Rails::Engine is abstract, you cannot instantiate it directly

The combination of Ruby 3.1.0 and Rails 7.0.0 caused this error when running the rspec command:
```bash
$ pruebas
Calling `DidYouMean::SPELL_CHECKERS.merge!(error_name => spell_checker)' has been deprecated. Please call `DidYouMean.correct_error(error_name, spell_checker)' instead.

An error occurred while loading rails_helper.
Failure/Error: require File.expand_path('../config/environment', __dir__)

RuntimeError:
  Rails::Engine is abstract, you cannot instantiate it directly.
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.0/lib/rails/railtie.rb:246:in `initialize'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.0/lib/rails/railtie.rb:184:in `new'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.0/lib/rails/railtie.rb:184:in `instance'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.0/lib/rails/railtie.rb:223:in `method_missing'
```

It was fixed by updating Rails to version 7.0.1 as [commented](https://stackoverflow.com/a/76025360/1407371).


## Gem Load Error is: Unable to find a compatible xml library

```bash
Bundler::GemRequireError: There was an error while trying to load the gem 'aws-sdk-athena'.
Gem Load Error is: Unable to find a compatible xml library. Ensure that you have installed or added to your Gemfile one of ox, oga, libxml, nokogiri or rexml
Backtrace for gem load error is:
/usr/local/bundle/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/xml/parser.rb:74:in `set_default_engine'
/usr/local/bundle/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/xml/parser.rb:96:in `<class:Parser>'
/usr/local/bundle/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/xml/parser.rb:7:in `<module:Xml>'
/usr/local/bundle/gems/aws-sdk-core-3.181.0/lib/aws-sdk-core/xml/parser.rb:5:in `<module:Aws>'
```

This is solved by making sure `gcompat` software is installed:

```bash
RUN set -ex \
    && apk update \
    && apk add build-base \
         postgresql-dev \
         tzdata \
         nodejs \
         git \
         libxml2-dev \
         libxslt-dev \
         openssl \
         yarn \
         gcompat
```