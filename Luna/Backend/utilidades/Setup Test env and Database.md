# Setup Test DB

Happened that tried to run rspec command and got all migrations pending???

```bash
While loading rails_helper an `exit` / `raise SystemExit` occurred, RSpec will now quit.
Failure/Error: exit 1

SystemExit:
  exit
# ./spec/rails_helper.rb:50:in `exit'
# ./spec/rails_helper.rb:50:in `rescue in <top (required)>'
# ./spec/rails_helper.rb:46:in `<top (required)>'
# ------------------
# --- Caused by: ---
# ActiveRecord::PendingMigrationError:
#
#
#   Migrations are pending. To resolve this issue, run:
#
#           bin/rails db:migrate RAILS_ENV=test
#
#   You have 782 pending migrations:
```

And running the migration command would fail. So I needed to run the `db:schema:load` command instead. This is the way to setup the test db using schema instead:
```bash
bundle exec rails db:environment:set RAILS_ENV=test
bundle exec rake db:drop RAILS_ENV=test
bundle exec rake db:create RAILS_ENV=test
bundle exec rake db:schema:load RAILS_ENV=test
```

And that's it.

One line:
```
bundle exec rails db:environment:set RAILS_ENV=test && bundle exec rake db:drop RAILS_ENV=test && bundle exec rake db:create RAILS_ENV=test && bundle exec rake db:schema:load RAILS_ENV=test
```