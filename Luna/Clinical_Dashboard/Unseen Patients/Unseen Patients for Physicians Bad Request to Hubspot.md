# Bad Request error when requesting Unseen Patients for Physicians

> [!INFO]
> This is for the Unseen Patients for Physicians feature.

This is the error in Sentry:
```
<HTTP::Response/1.1 
  400 Bad Request {
  	"Date"=>"Tue, 16 Jul 2024 15:23:25 GMT",
  	"Content-Type"=>"application/json;charset=utf-8",
  	"Content-Length"=>"221",
  	"Connection"=>"close",
  	"CF-Ray"=>"8a42fe4cad2976a5-SEA",
  	"CF-Cache-Status"=>"DYNAMIC",
  	"Strict-Transport-Security"=>"max-age=31536000; includeSubDomains; preload",
  	"Vary"=>"origin",
  	"access-control-allow-credentials"=>"false",
  	"x-content-type-options"=>"nosniff",
  	"x-envoy-upstream-service-time"=>"60",
  	"x-evy-trace-listener"=>"listener_https",
  	"x-evy-trace-route-configuration"=>"listener_https/all",
  	"x-evy-trace-route-service-name"=>"envoyset-translator",
  	"x-evy-trace-served-by-pod"=>"iad02/hubapi-td/envoy-proxy-7dd59b876-4rbtd",
  	"x-evy-trace-virtual-host"=>"all",
  	"x-hubspot-correlation-id"=>"8c82d381-0abb-4186-af91-216dff28d8df",
  	"x-hubspot-ratelimit-daily"=>"1000000",
  	"x-hubspot-ratelimit-daily-remaining"=>"919952",
  	"x-hubspot-ratelimit-interval-milliseconds"=>"10000",
  	"x-hubspot-ratelimit-max"=>"200",
  	"x-hubspot-ratelimit-remaining"=>"198",
  	"x-hubspot-ratelimit-secondly"=>"20",
  	"x-hubspot-ratelimit-secondly-remaining"=>"19",
  	"x-request-id"=>"8c82d381-0abb-4186-af91-216dff28d8df",
  	"Report-To"=>"{
  	   \"endpoints\":[{\"url\":\"https:\\/\\/a.nel.cloudflare.com\\/report\\/v4?s=p1%2BjHBOMeBmlNT0ND%2B6blkEmW2u1E9oxZjX%2B4uDxx2as3BYfEwTNkCwmH%2BECwgkudQaa6ZUr5SGslErTpVPcgOV%2BzgWrwjr68UK6Dy0hUn2%2Bd0SE1cyW%2FdNqEWjB1oMM\"}],
  	   \"group\":\"cf-nel\",
  	   \"max_age\":604800}",
  	"NEL"=>"{\"success_fraction\":0.01,\"report_to\":\"cf-nel\",\"max_age\":604800}",
  	"Server"=>"cloudflare"
  }>
```

At first I thought it might be rate limit but [Hubspot docs](https://developers.hubspot.com/beta-docs/guides/apps/api-usage/usage-details?uuid=416ebae4-6a4f-454e-91cc-736d8bbbb0c7#service-limits) say the return a 429 error for rate limits.

Then I added some extra logging, browsed the UI and found these log lines:
```
[HUBSPOT_LIMITS] ReferredPatientsInformation request to Hubspot Failed

[HUBSPOT_LIMITS] Made AssociatedContacts request to Hubspot
[HUBSPOT_LIMITS] Made AssociatedContacts request to Hubspot
[HUBSPOT_LIMITS] Made AssociatedContacts request to Hubspot
```

Surprisingly what's failing it's the batch request to get information for the specified properties. It fails when selecting multiple therapists not only Dr. Sal.

## Causes

I think it's the amount of elements in the `inputs` array of the request.

The request to the endpoint `crm/v3/objects/contacts/batch/read` accepts a JSON payload like this one:
```json
{
  "properties": [
    "firstname",
    "lastname",
    "email",
    "date_of_birth",
    "createdate",
    "hs_lead_status"
  ],
  "inputs": [
    {
      "id": "13947063307"
    },
    {
      "id": "13948203075"
    }
  ]
}
```

It all works fine until I try a request where the `inputs` list contains more than a hundred elements:
```bash
[HUBSPOT_LIMITS] Made AssociatedContacts request to Hubspot
[HUBSPOT_LIMITS] Made AssociatedContacts request to Hubspot
107
[HUBSPOT_LIMITS] ReferredPatientsInformation request to Hubspot Failed
```

And I get the bad request error.

And it is indeed:
```json
{
  "status": "error",
  "message": "The maximum number of inputs supported in a batch is 100. Please reduce the number of items and try again",
  "correlationId": "3784ea30-07ce-4866-98e8-bf691d18899e",
  "category": "VALIDATION_ERROR"
}
```

It's also stated in [Hubspot Docs](https://developers.hubspot.com/beta-docs/guides/api/crm/objects/contacts#limits) (found it late).

> Batch operations for creating, updating, and archiving are limited to batches of 100.

## Conclusions

### Is this an actual bug?

I found this by using the Admin Dashboard. In this version one can collect data for several physicians. It'll be a problem if internal users want to verify the Unseen Patients table loads correctly.

On the other hand, for actual physician dashboards, this might not be a problem. They'd probably won't have so many contacts to fetch but it's a problem waiting to happen.

### What's the solution?

Detect if the payload carries more than a 100 inputs. If it does, split the payload and make necessary requests to the batch endpoint.

This would be similar to the way data from Athena is fetched.