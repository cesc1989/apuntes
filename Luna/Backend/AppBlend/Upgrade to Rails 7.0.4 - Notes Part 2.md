# Upgrade to Rails 7.0.4 - Notes Part 2

This is a continuation of [[Upgrade to Rails 7.0.4 - Notes]]

# Errors in CI

## Unsafe redirect to "https://www.getluna.com", pass allow_other_host: true to redirect anyway

Error:
```
 1) PaymentsController Actions redirects without an appointment id `aid` should redirect to getluna.com
     Failure/Error: redirect_to "https://www.getluna.com"

     ActionController::Redirecting::UnsafeRedirectError:
       Unsafe redirect to "https://www.getluna.com", pass allow_other_host: true to redirect anyway.
     # /Users/francisco/.gem/ruby/3.1.0/gems/turbolinks-5.2.1/lib/turbolinks/redirection.rb:12:in `redirect_to'
     # ./app/controllers/payments_controller.rb:9:in `index'
```

Fix:
```ruby
redirect_to("https://www.getluna.com", allow_other_host: true)
```

About: https://blog.saeloun.com/2022/02/08/rails-7-raise-unsafe-redirect-error/

