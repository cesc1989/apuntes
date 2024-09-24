# Troubleshooting

This section covers solutions for many errors that happen when trying to install a version of PostgreSQL or whenever an installation breaks.

Situations I've seen an installation breaks is upgrading or turning off and on the computer.

## invalid byte sequence for encoding “UTF8”

When you backup a database with `pg_dump` command, as far as I know, you [cannot](http://stackoverflow.com/a/4867690/1407371) [restore](http://dba.stackexchange.com/a/4781/69085) that database from that file using `pg_restore`. The reason behind this is that the output file produced by `pg_dump` is not readable by `pg_restore`.

You should restore a backup with `psql` command. However, sometimes you would stomp with error:

    $ ERROR:  invalid byte sequence for encoding "UTF8":

I'm not really sure why this happens but this is easily solvable. You need to fix the encoding of the file using a tool called `iconv` as follows:

    $ iconv -t utf-8 /path/to/file > /path/to/fixed_dump

> [Read more about `iconv`](https://linux.die.net/man/1/iconv)

Now, you can use `fixed_dump` to restore your database using `psql` command:

    $ psql [db_name] --host [host_name] --username [username] < /path/to/fixed_dump

## dyld: Library not loaded

Found this error after installing PostgreSQL in MacOS.

```
dyld: Library not loaded: /usr/local/opt/openssl/lib/libssl.1.0.0.dylib
 Referenced from: /usr/local/Cellar/postgresql/11.4/lib/libpq.5.11.dylib
 Reason: image not found
[1]  7064 abort   psql
```

What worked in that specific situation was to reinstall Postgres but also run some Homebrew commands to make sure everything was alright.

```
$ brew uninstall --force postgresql
$ rm -rf /usr/local/var/postgres # in macos Ventura is /opt/homebrew/var/postgres
$ brew doctor
$ brew update # If there are errors, run any command Brew tells you to
$ brew install postgresql
$ brew services start postgresql
$ createdb $(whoami)
```

## PG::ConnectionBad or could not connect to server

This error

```
psql: error: could not connect to server: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

If `brew services restart postgresql` doesn't work try `rm /usr/local/var/postgres/postmaster.pid`.

Seen at [Stack Overflow](https://stackoverflow.com/a/18832331/1407371).

If above commands don't work, run `postgres -D /usr/local/var/postgres` to get an ouput of what's going on.

> For Macos Ventura it should be `postgres -D /opt/homebrew/var/postgres`.

Example:

```
2020-08-03 16:08:47.900 -05 [76016] FATAL:  database files are incompatible with server
2020-08-03 16:08:47.900 -05 [76016] DETAIL:  The data directory was initialized by PostgreSQL version 11, which is not compatible with this version 12.3.
```

The second error message can be solved with `brew postgresql-upgrade-database`.

Seen at [Stack Overflow](https://stackoverflow.com/a/48409221/1407371)

## Error after updating Postgres via Homebrew autoupdates command

I ran some brew commands in order to install PHP and it updated several other packages. Few days later, I was about to create a record in a Rails app that uses the `pgcrypto` extension to generate UUIDs and it broke with this message:

```
PG::UndefinedFile: ERROR:  could not load library "/usr/local/lib/postgresql/pgcrypto.so": dlopen(/usr/local/lib/postgresql/pgcrypto.so, 10): Symbol not found: _gen_random_uuid
```

No results on Google and luckyly I read the last approach in previous section: `brew postgresql-upgrade-database`.

Nothing was working. When I listed the extensions via `psql` it'd only list `plpgsql`. If I tried installing `pgcrypto`, it'd fail:

```sql
fquintero=# CREATE EXTENSION pgcrypto;
ERROR:  unrecognized parameter "trusted" in file "/usr/local/share/postgresql/extension/pgcrypto.control"
```

So, already in disbelief and about to wipe out Postgres from the computer, I ran:
```
$ brew postgresql-upgrade-database

(...)
==> Summary
🍺  /usr/local/Cellar/postgresql@12/12.5: 3,224 files, 37.6MB
==> Upgrading postgresql data from 12 to 13...
Stopping `postgresql`... (might take a while)
==> Successfully stopped `postgresql` (label: homebrew.mxcl.postgresql)
(...)
```

It did its magic and could install the extension and keep working happily.

## dyld: lazy symbol binding failed: Symbol not found: _PQresultMemorySize

After having to remove and reinstall postgresql@14 via Brew I came across this error:

```
dyld: lazy symbol binding failed: Symbol not found: _PQresultMemorySize
  Referenced from: /Users/fquintero/.rvm/gems/ruby-2.7.1/gems/pg-1.4.5/lib/pg_ext.bundle
  Expected in: /usr/lib/libpq.5.dylib

dyld: Symbol not found: _PQresultMemorySize
  Referenced from: /Users/fquintero/.rvm/gems/ruby-2.7.1/gems/pg-1.4.5/lib/pg_ext.bundle
  Expected in: /usr/lib/libpq.5.dylib

Abort trap: 6
```

Found this on [Stack Overflow](https://stackoverflow.com/a/64601542/1407371).

Some recommend to reinstall gem `pg` but the simplest solution that worked for me was using `gem pristine pg`.	

## Bootstrap failed: 5: Input/output error

This error:

```
$ brew services start postgresql@14
Bootstrap failed: 5: Input/output error
Try re-running the command as root for richer errors.
Error: Failure while executing; `/bin/launchctl bootstrap gui/502 /Users/francisco/Library/LaunchAgents/homebrew.mxcl.postgresql@14.plist` exited with 5.
```

Appeared in Macos Ventura after running the brew command to install postgresql. Looks like it was already installed or something but it broke.

The fix? Remove everything as noted above and install again.

## dyld[14489]: Library not loaded: /opt/homebrew/opt/icu4c/lib/libicui18n.72.dylib

This error happened after I had shut down the PC. When turned on, PostgreSQL service was turned off and couldn't be started.

```
$ brew services info postgresql@14

postgresql@14 (homebrew.mxcl.postgresql@14)
Running: ✘
Loaded: ✔
Schedulable: ✘
```

Solutions mentioned above in this same document wouldn't work. To find what was the error this time needed to run this command:

```
$ postgres -D /opt/homebrew/var/postgres

dyld[12502]: Library not loaded: /opt/homebrew/opt/icu4c/lib/libicui18n.72.dylib
  Referenced from: <346EBF4B-6F73-3273-ADB1-08B455DBED37> /opt/homebrew/Cellar/postgresql@14/14.8/bin/postgres
  Reason: tried: '/opt/homebrew/opt/icu4c/lib/libicui18n.72.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OS/opt/homebrew/opt/icu4c/lib/libicui18n.72.dylib' (no such file), '/opt/homebrew/opt/icu4c/lib/libicui18n.72.dylib' (no such file), '/usr/local/lib/libicui18n.72.dylib' (no such file), '/usr/lib/libicui18n.72.dylib' (no such file, not in dyld cache), '/opt/homebrew/Cellar/icu4c/73.2/lib/libicui18n.72.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OS/opt/homebrew/Cellar/icu4c/73.2/lib/libicui18n.72.dylib' (no such file), '/opt/homebrew/Cellar/icu4c/73.2/lib/libicui18n.72.dylib' (no such file), '/usr/local/lib/libicui18n.72.dylib' (no such file), '/usr/lib/libicui18n.72.dylib' (no such file, not in dyld cache)
Abort trap: 6
```

The key here is that **Postgres is trying to load library icu4c version 72 but when inspecting the installed version is different one**:

```
$ brew info icu4c

==> icu4c: stable 73.2 (bottled) [keg-only]
C/C++ and Java libraries for Unicode and globalization
https://icu.unicode.org/home
/opt/homebrew/Cellar/icu4c/73.2 (268 files, 80.1MB)
```

This is the key to fix this error. We have to get the expected version for Postgres. To do this we need to get the version file from GitHub because trying to install via brew wouldn't work.

```
$ brew reinstall icu4c

==> Downloading https://ghcr.io/v2/homebrew/core/icu4c/manifests/73.2
Already downloaded: /Users/francisco/Library/Caches/Homebrew/downloads/2e5082de52a2c85ae665e51f8d0de0651611397cb02f4b4e2bb37898ba52a629--icu4c-73.2.bottle_manifest.json
==> Fetching icu4c
==> Downloading https://ghcr.io/v2/homebrew/core/icu4c/blobs/sha256:953797d46546c570c4fab4e8b2395624ae90acd492f75b68ff99fbd115ccd748
Already downloaded: /Users/francisco/Library/Caches/Homebrew/downloads/1e0f31a89e4d047dc95e651cf911b97e33b91f7e13d8909be0f60a5c18c1dc7c--icu4c--73.2.arm64_ventura.bottle.tar.gz
==> Reinstalling icu4c 
==> Pouring icu4c--73.2.arm64_ventura.bottle.tar.gz
🍺  /opt/homebrew/Cellar/icu4c/73.2: 268 files, 80.1MB
==> Running `brew cleanup icu4c`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
==> Upgrading 2 dependents of upgraded formula:
Disable this behaviour by setting HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
libxslt 1.1.37_1 -> 1.1.38_1, postgresql@14 14.8 -> 14.9
==> Downloading https://ghcr.io/v2/homebrew/core/libxslt/manifests/1.1.38_1
Already downloaded: /Users/francisco/Library/Caches/Homebrew/downloads/919836ac13cf7f2b1a61c2a03611718923ac8cde386b11b757e91025591c3f43--libxslt-1.1.38_1.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/postgresql/14/manifests/14.9
Already downloaded: /Users/francisco/Library/Caches/Homebrew/downloads/4b57da1f9c51857d70615ba4a417e293db01f6d2309a29ca617dead69a8da01f--postgresql@14-14.9.bottle_manifest.json
==> Checking for dependents of upgraded formulae...
==> No broken dependents to reinstall!
```

We need to find the formula file in Brew repo.

> For this case, I [found it here](https://github.com/Homebrew/homebrew-core/blob/11249c583b5b03fec5ac29e8d2a724f29c6ddcfd/Formula/icu4c.rb).

Download the file, and run `brew reinstall` using the downloaded file.

```
$ curl -O https://raw.githubusercontent.com/Homebrew/homebrew-core/11249c583b5b03fec5ac29e8d2a724f29c6ddcfd/Formula/icu4c.rb
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2034  100  2034    0     0   6742      0 --:--:-- --:--:-- --:--:--  6894

$ brew reinstall icu4c.rb

Error: Failed to load cask: icu4c.rb
Cask 'icu4c' is unreadable: wrong constant name #<Class:0x00000001272af0a8>
Warning: Treating icu4c.rb as a formula.
==> Downloading https://ghcr.io/v2/homebrew/core/icu4c/manifests/72.1
##################################################################################################################################################### 100.0%
==> Fetching icu4c
==> Downloading https://ghcr.io/v2/homebrew/core/icu4c/blobs/sha256:0666e999875e81eadd009af55007c95cda37c7234acc12eb85cf42177984c699
##################################################################################################################################################### 100.0%
==> Reinstalling icu4c 
Warning: icu4c 73.2 is available and more recent than version 72.1.
==> Pouring icu4c--72.1.arm64_ventura.bottle.tar.gz
🍺  /opt/homebrew/Cellar/icu4c/72.1: 263 files, 78.4MB
==> Running `brew cleanup icu4c`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Removing: /Users/francisco/Library/Caches/Homebrew/icu4c--72.1... (28.9MB)
==> Upgrading 2 dependents of upgraded formula:
Disable this behaviour by setting HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
libxslt 1.1.37_1 -> 1.1.38_1, postgresql@14 14.8 -> 14.9
==> Downloading https://ghcr.io/v2/homebrew/core/libxslt/manifests/1.1.38_1
Already downloaded: /Users/francisco/Library/Caches/Homebrew/downloads/919836ac13cf7f2b1a61c2a03611718923ac8cde386b11b757e91025591c3f43--libxslt-1.1.38_1.bottle_manifest.json
==> Fetching dependencies for libxslt: icu4c
==> Downloading https://ghcr.io/v2/homebrew/core/icu4c/manifests/72.1
Already downloaded: /Users/francisco/Library/Caches/Homebrew/downloads/adaa2fd48ef337b5bfee51ed8902dab61120bb41bd18f98eb20e744fdd5548c1--icu4c-72.1.bottle_manifest.json
Error: Formula installation already attempted: icu4c
```

It finished with an error but the reinstallation worked.

First, restarted postgres

```
$ brew services restart postgresql@14

Stopping `postgresql@14`... (might take a while)
==> Successfully stopped `postgresql@14` (label: homebrew.mxcl.postgresql@14)
==> Successfully started `postgresql@14` (label: homebrew.mxcl.postgresql@14)
```

Now check service is actually running

```
$ ls /opt/homebrew/Cellar/icu4c
72.1

$ postgres -D /opt/homebrew/var/postgres
postgres: could not access directory "/opt/homebrew/var/postgres": No such file or directory
Run initdb or pg_basebackup to initialize a PostgreSQL data directory.

$ brew services info postgresql@14
postgresql@14 (homebrew.mxcl.postgresql@14)
Running: ✔
Loaded: ✔
Schedulable: ✘
User: francisco
PID: 18634
```

Found this approach in [Stack Overflow](https://stackoverflow.com/a/56242725/1407371).

## Segmentation Fault when queryin in Rails apps

In the rails console, got this kind of error when trying to access any table.

```
[1] pry(main)> Workout.find("40218ba7-b5fd-479d-a551-5354cb9721b3")
/Users/francisco/.gem/ruby/3.0.6/gems/pg-1.2.3/lib/pg.rb:58: [BUG] Segmentation fault at 0x0000000000000110
ruby 3.0.6p216 (2023-03-30 revision 23a532679b) [arm64-darwin22]

-- Crash Report log information --------------------------------------------
   See Crash Report log file under the one of following:                    
     * ~/Library/Logs/DiagnosticReports                                     
     * /Library/Logs/DiagnosticReports                                      
   for more details.                                                        
Don't forget to include the above Crash Report log file in bug reports.     

-- Control frame information -----------------------------------------------
c:0073 p:---- s:0413 e:000412 CFUNC  :initialize
c:0072 p:---- s:0410 e:000409 CFUNC  :new
c:0071 p:0019 s:0405 e:000404 METHOD /Users/francisco/.gem/ruby/3.0.6/gems/pg-1.2.3/lib/pg.rb:58
```

To fix this, export ENV `PGGSSENCMODE`

```
export PGGSSENCMODE="disable"
```

Found this approach in [this comment](https://github.com/ged/ruby-pg/issues/538#issuecomment-1591629049) to the same issue in the ruby-pg repository.
