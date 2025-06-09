# What is devise-jwt needed for?

Github -> https://github.com/waiting-for-dev/devise-jwt

This gem was set up 3 years ago.

Commits introducing changes with devise-jwt:

- [Add gem](https://github.com/lunacare/physician-portal/commit/ab9dcd8509dca4ef216840ce1d46d85d9d2c09b2)
- [Mar 15th, 2022](https://github.com/lunacare/physician-portal/commit/8155784d77d25c1a2fc330538c53d6d2843573d4)

These things were also:

- jwt_denylist table and JwtDenylist model.
	- 0 records to date in alpha
- `:jwt_authenticatable, jwt_revocation_strategy: JwtDenylist` in User devise setup

This configuration in devise initializer:
```ruby
  config.jwt do |jwt|
    jwt.secret = ENV['JWT_SECRET'] || ENV['SECRET_KEY_BASE']
    jwt.expiration_time = 20.minutes.to_i # consistent with Luxe user timeout
    jwt.dispatch_requests = [
      ['POST', %r{^/login$}],
      ['GET', %r{^/dashboard$}]
    ]
    jwt.revocation_requests = [
      ['DELETE', %r{^/logout$}]
    ]
  end
```

Are all these needed?

## Removing devise-jwt code

Going to try to remove this code and find out what is affected.

Removed this from User model:
```diff
-:trackable,
-:jwt_authenticatable, jwt_revocation_strategy: JwtDenylist
+:trackable
```

And this from Devise initializer:
```diff
-  config.jwt do |jwt|
-    jwt.secret = ENV['JWT_SECRET'] || ENV['SECRET_KEY_BASE']
-    jwt.expiration_time = ENV['JWT_EXPIRATION_SECONDS_ADMIN_USER']&.to_i&.seconds || 20.minutes # consistent with Luxe user timeout
-    jwt.dispatch_requests = [
-      ['POST', %r{^/login$}],
-      ['GET', %r{^/dashboard$}]
-    ]
-    jwt.revocation_requests = [
-      ['DELETE', %r{^/logout$}]
-    ]
-  end
-
```

Launched the Rails server and the NPM server and was able to:

- log in using admin credentials
- filter a physician from the dropdown
- generate a physician link

# Can this devise-jwt code removed?

Looks like it. Gotta leave this baking in alpha for a while.