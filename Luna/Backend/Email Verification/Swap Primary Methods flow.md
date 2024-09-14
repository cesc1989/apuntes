# Swap Primary Methods flow in EmailVerificationsController

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