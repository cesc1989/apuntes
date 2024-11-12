# Make an HTTP request to a service endpoint from itself

Is it possible to make HTTP requests to a Rails backend from itself?

Let's take this endpoint from Patient Self Report:
```json
POST {{BASE_URL}}/patient_forms

{
  "patient": {
    "first_name": "Imagenes 01",
    "last_name": "Captadas 01",
    "internal_id": "858277623766",
    "forms_attributes": [
      {
        "type_name": "quick_dash",
        "injury_name": "shoulder"
      }
    ]
  }
}
```

Is it possible to make a request from the Rails server itself? The answer is yes.

One way to do it is by using an instance of the controller. Other way is using Net::Http or Faraday, for example.

# Instance of Controller

> [!INFO]
> This way does not need the server to be up.

For the described endpoint, we can make a request from itself by setting it up like this:
```ruby
controller = Api::V1::PatientFormsController.new

controller.request = ActionDispatch::TestRequest.create
controller.response = ActionDispatch::TestResponse.new

params = {
  "patient": {
    "first_name": "Imagenes 02",
    "last_name": "Captadas 02",
    "internal_id": "858277623766907",
    "forms_attributes": [
      {
        "type_name": "quick_dash",
        "injury_name": "shoulder"
      }
    ]
  }
}
controller.params = ActionController::Parameters.new(params)

result = controller.create

response_body = JSON.parse(controller.response.body)
ap response_body
```

The response:
```ruby
{
    "form" => {
                 "id" => 3,
         "patient_id" => 2,
               "link" => "localhost:8080/patients/858277623766907/forms/840ef7c2-e787-4e74-a0ae-01e7b5507fcd",
         "created_at" => "2024-10-16",
               "uuid" => "840ef7c2-e787-4e74-a0ae-01e7b5507fcd",
        "status_uuid" => "d4e8da14-749d-4e81-b45f-b20911a0ebc4",
            "v3_link" => "localhost:8080/v3/patients/858277623766907/forms/840ef7c2-e787-4e74-a0ae-01e7b5507fcd"
    }
}
```

These two:
```ruby
controller.request = ActionDispatch::TestRequest.create
controller.response = ActionDispatch::TestResponse.new
```

are needed to be able to pass params in the request object and be able to read a response from the response object.

Otherwise it throws an error:
```bash
/Users/francisco/.gem/ruby/3.1.0/gems/actionpack-7.0.4/lib/action_controller/metal.rb:147:in `rescue in status=': ActionController::Metal#status= delegated to @_response.status=, but @_response is nil: #<Api::V1::PatientFormsController:0x00000000220cb8> (Module::DelegationError)
/Users/francisco/.gem/ruby/3.1.0/gems/actionpack-7.0.4/lib/action_controller/metal.rb:147:in `status=': undefined method `status=' for nil:NilClass (NoMethodError)
```

## Works for GET requests (index action)

Example of an index action request:
```ruby
controller = Api::V3::PhysicianDashboard::Physicians::Patients::ChartsController.new
controller.request = ActionDispatch::TestRequest.create
controller.response = ActionDispatch::TestResponse.new

params = { "physician_id": "7e6fe728-a2f8-4e75-9935-cf2384999385", "patient_id": "07f2016d-e285-4864-ac2b-e28508d19175" }
controller.params = ActionController::Parameters.new(params)

result = controller.index
response_body = JSON.parse(controller.response.body)
```

# HTTP Request

> [!INFO]
> This way needs the server to be up.

```ruby
require 'net/http'
require 'uri'
require 'json'

uri = URI('http://localhost:3000/patient_forms')

http = Net::HTTP.new(uri.host, uri.port)
request = Net::HTTP::Post.new(uri.path, { 'Content-Type' => 'application/json' })

params = {
  "patient": {
    "first_name": "Imagenes 02",
    "last_name": "Captadas 02",
    "internal_id": "858277623766907",
    "forms_attributes": [
      {
        "type_name": "quick_dash",
        "injury_name": "shoulder"
      }
    ]
  }
}

request.body = params.to_json

response = http.request(request)

ap response.body
```

Response:
```ruby
{
    "form" => {
                 "id" => 8,
         "patient_id" => 2,
               "link" => "localhost:8080/patients/858277623766907/forms/c71c3dde-3211-4b31-9e46-da361e766ecf",
         "created_at" => "2024-10-16",
               "uuid" => "c71c3dde-3211-4b31-9e46-da361e766ecf",
        "status_uuid" => "e03de92a-d4ee-4770-bbb7-dc199d44707b",
            "v3_link" => "localhost:8080/v3/patients/858277623766907/forms/c71c3dde-3211-4b31-9e46-da361e766ecf"
    }
}
```