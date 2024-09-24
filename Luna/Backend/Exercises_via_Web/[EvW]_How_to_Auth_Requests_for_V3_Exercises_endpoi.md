# [EvW] How to Auth Requests for V3 Exercises endpoints
To get credentials, we need an Account record and run the `create_new_auth_token` method.

# Notice

Accounts are for Patient and Therapist. A single account does not belong to both.

So to auth requests in `v3/patient` we’d need to find an `Account.find().patient` and same for Therapist in `v3/therapist`.

# Patient Account

Example with a test account in alpha
```ruby
  Account.find("fff93dfc-1843-4ca2-a0fe-acb66635a904").create_new_auth_token
    
    {
     "access-token"=>"S2El6SgrWKQHwESIi8WWZA",
     "token-type"=>"Bearer",
     "client"=>"GVHrPGCwFH-XszO1TPFUkA",
     "expiry"=>"1756623625",
     "uid"=>"ryan+mailmereceipt2@getluna.com"
    }
```


# Therapist Account

Example therapist account in alpha
```ruby
Therapist.find("468fefc7-42ce-4777-9107-ad6bc2b0909c").account.create_new_auth_token
```


# How to use account credentials via Postman?

Given that auth for these endpoints are handled via the gem [devise-token-auth](https://devise-token-auth.gitbook.io/devise-token-auth/usage/testing), the values returned before need to be sent as headers:
```ruby
"ACCESS_TOKEN"=>"RlSe9WdjpX1ZBKxt1z637g",
"TOKEN_TYPE"=>"Bearer",
"CLIENT"=>"0r_BGSUwhKfiybPAqmkSAA",
"EXPIRY"=>"1539543600",
"UID"=>"53219041@random.com",
```

This is also how it is handled in tests:
```ruby
let(:patient_account) { create(:account, :patient) }

get(
	"/api/v3/patient/exercises/#{exercise.id}",
	headers: patient_account.create_new_auth_token
)
```

# List request headers in Rails

How to [list request headers](https://stackoverflow.com/a/32405432/1407371) in Rails
```ruby
pp request.headers.env.select{|k, _| k.in?(ActionDispatch::Http::Headers::CGI_VARIABLES) || k =~ /^HTTP_/}

{"REQUEST_METHOD"=>"GET",
 "SERVER_NAME"=>"www.example.com",
 "SERVER_PORT"=>"80",
 "QUERY_STRING"=>"",
 "PATH_INFO"=>"/api/v3/patient/exercises/cb3b81b2-e590-49a4-827f-956d5395d193",
 "HTTPS"=>"off",
 "SCRIPT_NAME"=>"",
 "CONTENT_LENGTH"=>"0",
 "REMOTE_ADDR"=>"127.0.0.1",
 "SERVER_PROTOCOL"=>"HTTP/1.0",
 "HTTP_VERSION"=>"HTTP/1.0",
 "HTTP_HOST"=>"www.example.com",
 "CONTENT_TYPE"=>nil,
 "HTTP_ACCEPT"=>
	"text/xml,application/xml,application/xhtml+xml,text/html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5",
 "HTTP_ACCESS_TOKEN"=>"RlSe9WdjpX1ZBKxt1z637g",
 "HTTP_TOKEN_TYPE"=>"Bearer",
 "HTTP_CLIENT"=>"0r_BGSUwhKfiybPAqmkSAA",
 "HTTP_EXPIRY"=>"1539543600",
 "HTTP_UID"=>"53219041@random.com",
 "HTTP_COOKIE"=>""}
```

