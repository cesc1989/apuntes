# Swap Primary Methods flow in EmailVerificationsController - 🟢

> [!INFO]
> Commit where all of the EmailVerification stuff was added
>
> `git show 9e126d598e`

> [!INFO]
> This is where the swapping was introduced
> 
> `git show e60a468cce`

We want to split the landing page (`app/views/user_communication_methods/email_verifications/new.html.haml`) into two different pages:
- `email_verifications/verified`
- `email_verifications/unverified`

But to do this first I have to understand this snippet in the controller:
```ruby
def new
	@communication_method.verify!(@code) if @communication_method.present? && @code.present?

	# For patients, handle the case where the email address has changed. Once they verify the
	# new email address, we want to switch the newly verified one to be the primary method.
	user = @communication_method&.user
	swap_primary_methods(@communication_method) if user.is_a?(Account) && user.patient?
rescue *ERRORS => e
	ExceptionLogger.log(e, "Error verifying email: #{e}")
end
```

With the two different landing pages, by bringing the conditional from the view:
```ruby
# email_verifications/new.html.erb

if @communication_method&.verified?
  # verified
elsif @communication_method&.verification_code != @code || @communication_method&.expired_verification_code?
  # invalid or expired
else
  # error
end
```

the controller would look like this:
```ruby
def new
  if @communication_method&.verified?
    redirect_to email_verifications_verified_path(@communication_method)
  elsif @communication_method&.verification_code != @code || @communication_method&.expired_verification_code?
    redirect_to email_verifications_unverified_path(@communication_method)
  else
    render :error_page
  end

  user = @communication_method&.user
  swap_primary_methods(@communication_method) if user.is_a?(Account) && user.patient?
end
```

Would this work? ~~No. The redirect or render will finish the action leaving the final part unexecuted. For it to work, I'd need to call the `swap_primary_methods` in every conditional so that it gets executed wherever it falls.~~

It should work. Did a test in a test application and it executed the code in the end always.

## Example Controller

Wrote this controller:
```ruby
class VerificationsController < ApplicationController
  def new
    verified = params[:result]

    if verified == "ok"
      redirect_to success_verifications_path
    elsif verified == "failure"
      redirect_to failure_verifications_path
    else
      redirect_to error_verifications_path
    end

    ap "Hola mundo cruel\nAqui estoys"
  end
end

```

The puts happens always regardless of the redirect. Rails executes all of the code in the controller and in the end it makes the redirect:
```bash
Started GET "/verifications/new" for 127.0.0.1 at 2024-09-13 22:12:45 -0500
Processing by VerificationsController#new as HTML
Redirected to http://localhost:3001/verifications/error
"Hola mundo cruel\nAqui estoys"
Completed 302 Found in 1ms (ActiveRecord: 0.0ms | Allocations: 223)


Started GET "/verifications/error" for 127.0.0.1 at 2024-09-13 22:12:45 -0500
Processing by VerificationsController#error as HTML
  Rendering layout layouts/application.html.erb
  Rendering verifications/error.html.erb within layouts/application
  Rendered verifications/error.html.erb within layouts/application (Duration: 0.8ms | Allocations: 80)
  Rendered shared/_head.html.erb (Duration: 10.4ms | Allocations: 7788)
  Rendered shared/_navbar.html.erb (Duration: 0.7ms | Allocations: 240)
  Rendered shared/_notices.html.erb (Duration: 0.0ms | Allocations: 20)
  Rendered shared/_footer.html.erb (Duration: 0.0ms | Allocations: 11)
  Rendered layout layouts/application.html.erb (Duration: 12.7ms | Allocations: 8454)
Completed 200 OK in 14ms (Views: 13.1ms | ActiveRecord: 0.0ms | Allocations: 8760)
```

In this log we can see it does everything in the controller action and afterwards it goes to the redirected page.