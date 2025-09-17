# Migración a Rails 6

# Pasos
1. Asegurarse de incrementar el nivel de *coverage* de la suite de pruebas
2. Estar en la versión de Ruby 2.5.3
3. Estar en la versión de Rails 5.2.3
4. Cambiar la versión de Rails en el Gemfile a la más reciente
5. Correr el comando `bundle update`
6. Correr el comando `rails app:update`
    1. Usar la opción `d` para ver las diferencias en los archivos
    2. Usar la opción `m` para hacer una mezcla sin borrar lo existente
        1. Se puede [configurar Sublime Text](https://stackoverflow.com/questions/44549733/how-to-use-visual-studio-code-as-default-editor-for-git-mergetool/44549734#44549734) como herramienta de *merge* para simplificar esta parte.
7. Actualiza las gemas faltantes según lo que se indique en [RailsDiff](http://railsdiff.org/5.2.3/6.0.2.rc1)
# Enlaces de Apoyo
- [How To Upgrade To Rails 6](https://selleo.com/blog/how-to-upgrade-to-rails-6)
- [Upgrading Rails apps with rake app:update](https://www.joshmcarthur.com/til/2019/07/25/upgrading-rails-apps-with-rake-appupdate.html)
- [Upgrading Ruby on Rails, Rails Guides](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html)
# Errores Encontrados Luego de Migrar
## DNS Rebinding protection
    To allow requests to api.lvh.me, add the following to your environment configuration:
    
    config.hosts << "api.lvh.me"

Lo resolví agregando a `config/environments/development.rb`:

    config.hosts << "api.lvh.me"

y reiniciar el servidor de rails.

Al respecto de este error:

- [Blocked host on Rails 6](https://www.fngtps.com/2019/rails6-blocked-host/)
- [Rails 6 adds guard against DNS Rebinding attacks](https://blog.bigbinary.com/2019/11/05/rails-6-adds-guard-against-dns-rebinding-attacks.html)
## Docker Alpine Image Building

The building process is the same as when using Rails 5.2 **BUT** in the step `rails assets:precompile` this error happens:


    Step 14/16 : RUN rails assets:precompile
     ---> Running in 1789ea9e4ee7
    /bin/sh: rails: not found
    The command '/bin/sh -c rails assets:precompile' returned a non-zero code: 127

Inspecting the container in Circle CI, I found out that it doesn’t have Ruby installed so couldn’t test very much with it.

In local environment, created the image and reproduced the error. Seems to be something related to [Nokogiri](https://nokogiri.org/) and missing system libraries to compile it and install:

    /usr/src/app # gem install rails -v 6.0.2
    Fetching activesupport-6.0.2.gem
    Fetching activejob-6.0.2.gem
    Fetching actionpack-6.0.2.gem
    Fetching actionview-6.0.2.gem
    Fetching activemodel-6.0.2.gem
    Fetching activerecord-6.0.2.gem
    Fetching actionmailer-6.0.2.gem
    Fetching actioncable-6.0.2.gem
    Fetching rails-6.0.2.gem
    Fetching actionmailbox-6.0.2.gem
    Fetching actiontext-6.0.2.gem
    Fetching railties-6.0.2.gem
    Fetching activestorage-6.0.2.gem
    Successfully installed activesupport-6.0.2
    Building native extensions. This could take a while...
    ERROR:  Error installing rails:
    ERROR: Failed to build gem native extension.
    
        current directory: /usr/local/bundle/gems/nokogiri-1.10.7/ext/nokogiri
    /usr/local/bin/ruby -I /usr/local/lib/ruby/site_ruby/2.5.0 -r ./siteconf20191218-31-djw7o.rb extconf.rb
    checking if the C compiler accepts ... *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of necessary
    libraries and/or headers.  Check the mkmf.log file for more details.  You may
    need configuration options.
    
    Provided configuration options:
    --with-opt-dir
    --without-opt-dir
    --with-opt-include
    --without-opt-include=${opt-dir}/include
    --with-opt-lib
    --without-opt-lib=${opt-dir}/lib
    --with-make-prog
    --without-make-prog
    --srcdir=.
    --curdir
    --ruby=/usr/local/bin/$(RUBY_BASE_NAME)
    --help
    --clean
    /usr/local/lib/ruby/2.5.0/mkmf.rb:456:in `try_do': The compiler failed to generate an executable file. (RuntimeError)
    You have to install development tools first.
    from /usr/local/lib/ruby/2.5.0/mkmf.rb:574:in `block in try_compile'
    from /usr/local/lib/ruby/2.5.0/mkmf.rb:521:in `with_werror'
    from /usr/local/lib/ruby/2.5.0/mkmf.rb:574:in `try_compile'
    from extconf.rb:138:in `nokogiri_try_compile'
    from extconf.rb:162:in `block in add_cflags'
    from /usr/local/lib/ruby/2.5.0/mkmf.rb:632:in `with_cflags'
    from extconf.rb:161:in `add_cflags'
    from extconf.rb:416:in `<main>'
    
    To see why this extension failed to compile, please check the mkmf.log which can be found here:
    
      /usr/local/bundle/extensions/x86_64-linux/2.5.0/nokogiri-1.10.7/mkmf.log
    
    extconf failed, exit code 1
    
    Gem files will remain installed in /usr/local/bundle/gems/nokogiri-1.10.7 for inspection.
    Results logged to /usr/local/bundle/extensions/x86_64-linux/2.5.0/nokogiri-1.10.7/gem_make.out

But when I’d install in the container the removed libraries (`libxml2-dev and libxslt-dev`), it would work.

First, reinstalled packages `build-base libxml2-dev libxslt-dev`, as per the [instructions for Docker Alpine](https://nokogiri.org/tutorials/installing_nokogiri.html#ruby-on-alpine-linux-docker_1) image in the Nokogiri docs.

> These packages are removed in the build process to make the image small


    /usr/src/app # apk add --no-cache build-base libxml2-dev libxslt-dev
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
    (1/15) Installing binutils (2.31.1-r2)
    (2/15) Installing libmagic (5.36-r1)
    (3/15) Installing file (5.36-r1)
    (4/15) Installing isl (0.18-r0)
    (5/15) Installing libgomp (8.3.0-r0)
    (6/15) Installing libatomic (8.3.0-r0)
    (7/15) Installing mpfr3 (3.1.5-r1)
    (8/15) Installing mpc1 (1.0.3-r1)
    (9/15) Installing gcc (8.3.0-r0)
    (10/15) Installing musl-dev (1.1.20-r5)
    (11/15) Installing libc-dev (0.7.1-r0)
    (12/15) Installing g++ (8.3.0-r0)
    (13/15) Installing make (4.2.1-r2)
    (14/15) Installing fortify-headers (1.0-r0)
    (15/15) Installing build-base (0.5-r1)
    Executing busybox-1.29.3-r10.trigger
    OK: 531 MiB in 176 packages

Then installed nokogiri:

    /usr/src/app # gem install nokogiri
    Building native extensions. This could take a while...
    Successfully installed nokogiri-1.10.7
    1 gem installed

And finally ran `bundle install`:

    /usr/src/app # bundle install
    (...)
    Using rails 6.0.2
    Using recaptcha 5.2.1
    Using redis 4.0.3
    Using redis-namespace 1.7.0
    Using tilt 2.0.10
    Using sinatra 2.0.7
    Using vegas 0.1.11
    Using resque 1.27.4
    Using sassc 2.2.1
    Using sassc-rails 2.1.2
    Using sass-rails 6.0.0
    Using sentry-raven 2.13.0
    Using uglifier 4.2.0
    Using wicked_pdf 1.4.0
    Bundle complete! 42 Gemfile dependencies, 95 gems now installed.
    Gems in the groups development and test were not installed.
    Bundled gems are installed into `/usr/local/bundle`

However, `gem list rails` wouldn’t show rails gem until it was installed directly:

    /usr/src/app # gem install rails

Finally, solved this issue by installing gems rails and nokogiri in the build process:

    RUN set -ex \
        && apk add --upgrade --no-cache --virtual .app-builddeps \
            $BUILD_PACKAGES \
            $DEV_PACKAGES \
        && apk add --no-cache --virtual .app-rundeps $RUNTIME_PACKAGES \
        && update-ca-certificates \
        && gem install nokogiri -- --use-system-libraries \
        && gem install rails -v 6.0.2 \
        && gem install bundler && bundle config disable_multisource \
        && bundle install --jobs 2 --retry 5 --without $BUNDLE_WITHOUT \
        && apk del .app-builddeps

Related links:

- [Install Nokogiri Ruby on Alpine Linux](https://nokogiri.org/tutorials/installing_nokogiri.html#ruby-on-alpine-linux-docker_1)
## active_model_serialiazers gem dependencies locking

This error:

    Bundler could not find compatible versions for gem "activemodel":
      In Gemfile:
        active_model_serializers (= 0.10.10) was resolved to 0.10.10, which depends on
          activemodel (>= 4.1, < 6.1)
    
        rails (= 6.0.0) was resolved to 6.0.0, which depends on
          activemodel (= 6.0.0)

And also:

    Bundler could not find compatible versions for gem "activerecord":
      In Gemfile:
        annotate (~> 3) was resolved to 3.0.3, which depends on
          activerecord (>= 3.2, < 7.0)
    
        rails (= 6.0.0) was resolved to 6.0.0, which depends on
          activerecord (= 6.0.0)

Though I tried forking and [increasing the maximum valid Rails version](https://github.com/cesc1989/active_model_serializers/commit/b52652dad9528f84c9a54cb1f0a735dec48e53ac) for Active Model Serializers what ended working for me was commenting offending gems, running `bundle update rails`, uncommenting the offending gems and finally running `bundle install` so they be available again.

This [issue](https://github.com/rails-api/active_model_serializers/issues/2320) highlights some problems that arose with the gem but it was already solved by a PR.

## Error ActiveRecord::NoEnvironmentInSchemaError

Appeared after upgrading to Rails 6.0.2 from 6.0.0.rc2:

    rails aborted!
    ActiveRecord::NoEnvironmentInSchemaError: 
    
    Environment data not found in the schema. To resolve this issue, run: 
    
            rails db:environment:set RAILS_ENV=test
    
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/activerecord-6.0.2/lib/active_record/migration.rb:1151:in `last_stored_environment'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/activerecord-6.0.2/lib/active_record/tasks/database_tasks.rb:60:in `check_protected_environments!'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/activerecord-6.0.2/lib/active_record/railties/databases.rake:15:in `block (2 levels) in <top (required)>'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/activerecord-6.0.2/lib/active_record/railties/databases.rake:494:in `block (3 levels) in <top (required)>'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/railties-6.0.2/lib/rails/commands/rake/rake_command.rb:23:in `block in perform'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/railties-6.0.2/lib/rails/commands/rake/rake_command.rb:20:in `perform'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/railties-6.0.2/lib/rails/command.rb:48:in `invoke'
    /Users/fquintero/.rvm/gems/ruby-2.5.3/gems/railties-6.0.2/lib/rails/commands.rb:18:in `<top (required)>'
    bin/rails:4:in `require'
    bin/rails:4:in `<main>'
    Tasks: TOP => db:test:load => db:test:purge => db:check_protected_environments
    (See full trace by running task with --trace)

Using the command it suggests solves the thing but just temporarily. Also the suggestions in this [Stack Overflow question](https://stackoverflow.com/questions/38209186/rails-5-rspec-environment-data-not-found-in-the-schema) work but also temporary.

