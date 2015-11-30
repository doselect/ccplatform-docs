---
title: API Reference

language_tabs:
  - shell: cURL
  - ruby
  - python

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

The Stripe API is organized around REST. Our API has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. We support cross-origin resource sharing, allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API key in any public website's client-side code). JSON is returned by all API responses, including errors, although our API libraries convert responses to appropriate language-specific objects.
To make the API as explorable as possible, accounts have test mode and live mode API keys. There is no "switch" for changing between modes, just use the appropriate key to perform a live or test transaction. Requests made with test mode credentials never hit the banking networks and incur no cost.

# Authentication

  
```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

'''shell
curl https://api.stripe.com/v1/charges \
    -u sk_test_BQokikJOvBiI2HlWgH4olfQ2:
'''

>cURL uses the -u flag to pass basic auth credentials (adding a colon after your API key prevents cURL from asking for a password).A sample test API key is included in all the examples on this page, so you can test any example right away. To test requests using your account, replace the sample API key with your actual API key.

```shell
# With curl, you can just pass the correct header with each request
curl "api_endpoint_here" \
    -H "Authorization: meowmeowmeow"
```

Authenticate your account when using the API by including your secret API key in the request. You can manage your API keys in the Dashboard. Your API keys carry many privileges, so be sure to keep them secret! Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth.
Authentication to the API is performed via HTTP Basic Auth. Provide your API key as the basic auth username value. You do not need to provide a password.
All API requests must be made over HTTPS. Calls made over plain HTTP will fail. API requests without authentication will also fail.




<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

#Errors

Stripe uses conventional HTTP response codes to indicate the success or failure of an API request. In general, codes in the 2xx range indicate success, codes in the 4xx range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.), and codes in the 5xx range indicate an error with Stripe's servers (these are rare).
Not all errors map cleanly onto HTTP response codes, however. When a request is valid but does not complete successfully (e.g., a card is declined), we return a 402 error code.

>###ATTRIBUTES
type | The type of error returned. Can be: `api_connection_error`, etc.
----- |--------------------------------------------------------------------------------------------------------------------------------
message (optional)|A human-readable message providing more details about the error. For card errors, these messages can be shown to your users.
code (optional)|For card errors, a short string from amongst those listed on the right describing the kind of card error that occurred.
param (optional)|The parameter the error relates to if the error is parameter-specific. You can use this to display a message near the correct form field, for example.



>###HTTP status code summary
Error Code | Summary
---------- | ------- 
200 - OK|Everything worked as expected.
400 - Bad Request|The request was unacceptable, often due to missing a required parameter.
401 - Unauthorized|No valid API key provided.
402 - Request Failed|Theparameters were valid but the request failed.
404 - Not Found|The requested resource doesn't exist.
429 - Too Many Requests|Too many requests hit the API too quickly.
500, 502, 503, 504 - Server Errors|Something went wrong on Stripe's end. (These are rare.)



>###Error TYPES
Error | Description
-------|------------
api_connection_error|Failure to connect to Stripe's API.
api_error|API errors cover any other type of problem (e.g., a temporary problem with Stripe's servers) and are extremely uncommon.
authentication_error|Failure to properly authenticate yourself in the request.
card_error|Card errors are the most common type of error you should expect to handle. They result when the user enters a card that can't be charged for some reason.
invalid_request_error|Invalid request errors arise when your request has invalid parameters.
rate_limit_error|Too many requests hit the API too quickly.



>###CODES
Error|Description
------|-----------
invalid_number|The card number is not a valid credit card number.
invalid_expiry_month|The card's expiration month is invalid.
invalid_expiry_year|The card's expiration year is invalid.
invalid_cvc|The card's security code is invalid.
incorrect_number|The card number is incorrect.
expired_card|The card has expired.
incorrect_cvc|The card's security code is incorrect.
incorrect_zip|The card's zip code failed validation.
card_declined|The card was declined.
missing|There is no card on a customer that is being charged.
processing_error|An error occurred while processing the card.

>CVC validation and zip validation can be enabled/disabled in your settings

#Expanding Objects

Many objects contain the ID of a related object in their response properties. For example, a Charge may have an associated Customer ID. Those objects can be expanded inline with the expand request parameter. Objects that can be expanded are noted in this documentation. This parameter is available on all API requests, and applies to the response of that request only.
You can nest expand requests with the dot property. For example, requesting invoice.customer on a charge will expand the invoice property into a full Invoice object, and will then expand the customer property on that invoice into a full Customer object.
You can expand multiple objects at once by identifying multiple items in the expand array.


```shell
curl https://api.stripe.com/v1/charges/ch_17CYv92eZvKYlo2 \
   -u sk_test_BQokikJOvBiI2HlWgH4olfQ2: \
   -d expand[]=customer
```


#Idempotent Requests

The API supports idempotency for safely retrying requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.
To perform an idempotent request, attach a unique key to any POST request made to the API via the Idempotency-Key: <key> header.
How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters. The keys expire after 24 hours.


```shell
curl https://api.stripe.com/v1/charges
   -u sk_test_BQokikJOvBiI2HlWgH4olfQ2:
   -H "Idempotency-Key: 48YnJFsWVbZtXi9A"
   -d amount=400
   -d source=tok_17BxFu2eZvKYlo2C3qeOBMKT
   -d currency=usd
```

#Metadata

Updatable Stripe objects—including Account, Charge, Customer, Refund, Subscription, and Transfer—have a metadata parameter. You can use this parameter to attach key-value data to these Stripe objects.
Metadata is useful for storing additional, structured information on an object. As an example, you could store your user's full name and corresponding unique identifier from your system on a Stripe Customer object. Metadata is not used by Stripe (e.g., to authorize or decline a charge), and won't be seen by your users unless you choose to show it to them.
Some of the objects listed above also support a description parameter. You can use the description parameter to annotate a charge, for example, with a human-readable description, such as "2 shirts for test@example.com". Unlike metadata, description is a single string, and your users may see it (e.g., in email receipts Stripe sends on your behalf).
Note: You can have up to 20 keys, with key names up to 40 characters long and values up to 500 characters long.
SAMPLE METADATA USE CASES
Link IDs
Attach your system's unique IDs to a Stripe object for easy lookups. Add your order number to a charge, your user ID to a customer or recipient, or a unique receipt number to a transfer, for example.
Refund papertrails
Store information about why a refund was created, and by whom.
Customer details
Annotate a customer by storing the customer's phone number for your later use.


```shell
curl https://api.stripe.com/v1/charges
   -u sk_test_BQokikJOvBiI2HlWgH4olfQ2:
   -d amount=400
   -d currency=usd
   -d source=tok_17BxFu2eZvKYlo2C3qeOBMKT
   -d metadata[order_id]=6735
```

```json
{
  "id": "ch_17CYv92eZvKYlo2CxoRDVUAg",
  "object": "charge",
  "amount": 1000,
  "amount_refunded": 0,
  "application_fee": null,
  "balance_transaction": "txn_17A7uR2eZvKYlo2CwqYV1UXn",
  "captured": true,
  "created": 1448817907,
  "currency": "usd",
  "customer": "cus_3kv1TGwZm5jjF6",
  "description": null,
  "destination": null,
  "dispute": null,
  "failure_code": null,
  "failure_message": null,
  "fraud_details": {
  },
  "invoice": "in_17CXwc2eZvKYlo2CkkvMVhCM",
  "livemode": false,
  "metadata": {
    "order_id": "6735"
  },
  "paid": true,
  "receipt_email": null,
  "receipt_number": null,
  "refunded": false,
  "refunds": {
    "object": "list",
    "data": [

    ],
    "has_more": false,
    "total_count": 0,
    "url": "/v1/charges/ch_17CYv92eZvKYlo2CxoRDVUAg/refunds"
  },
  "shipping": null,
  "source": {
    "id": "card_103kv12eZvKYlo2CkoyNDCVN",
    "object": "card",
    "address_city": null,
    "address_country": null,
    "address_line1": null,
    "address_line1_check": null,
    "address_line2": null,
    "address_state": null,
    "address_zip": null,
    "address_zip_check": null,
    "brand": "Visa",
    "country": "US",
    "customer": "cus_3kv1TGwZm5jjF6",
    "cvc_check": null,
    "dynamic_last4": null,
    "exp_month": 8,
    "exp_year": 2019,
    "funding": "credit",
    "last4": "4242",
    "metadata": {
    },
    "name": null,
    "tokenization_method": null
  },
  "statement_descriptor": null,
  "status": "succeeded"
}
```

#Pagination

All top-level API resources have support for bulk fetches via "list" API methods. For instance you can list charges, list customers, and list invoices. These list API methods share a common structure, taking at least these three parameters: limit, starting_after, and ending_before.
Stripe utilizes cursor-based pagination via the starting_after and ending_before parameters. Both take an existing object ID value (see below). The ending_before parameter returns objects created before the named object, in descending chronological order. The starting_after parameter returns objects created after the named object, in ascending chronological order. If both parameters are provided, only ending_before is used.

>###ARGUMENTS
 | 
------|-------
limit (optional)   | A limit on the number of objects to be returned, between 1 and 100.
starting_after (optional) | A cursor for use in pagination. starting_after is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with obj_foo, your subsequent call can include starting_after=obj_foo in order to fetch the next page of the list.
ending_before(optional)| A cursor for use in pagination. ending_before is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with obj_bar, your subsequent call can include ending_before=obj_bar in order to fetch the previous page of the list.

>###LIST RESPONSE FORMAT
 | 
------|-------
object string, value is "list"| A string describing the object type returned.
data array| An array containing the actual response elements, paginated by any request parameters.
has_more boolean |Whether or not there are more elements available after this set. If false, this set comprises the end of the list.
url string| The URL for accessing this list.


```shell
curl https://api.stripe.com/v1/customers?limit=3
   -u sk_test_BQokikJOvBiI2HlWgH4olfQ2:
```

```json
{
  "object": "list",
  "url": "/v1/customers",
  "has_more": false,
  "data": [
    {
      "id": "cus_7RRLUa32qryeH9",
      "object": "customer",
      "account_balance": 0,
      "created": 1448816155,
      "currency": "usd",
      "default_source": "card_17CYSq2eZvKYlo2CVQW2Z0RS",
      "delinquent": false,
      "description": null,
      "discount": null,
      "email": "newvmedix+p@gmail.com",
      "livemode": false,
      "metadata": {
      },
      "shipping": null,
      "sources": {
        "object": "list",
        "data": [
          {
            "id": "card_17CYSq2eZvKYlo2CVQW2Z0RS",
            "object": "card",
            "address_city": null,
            "address_country": null,
            "address_line1": null,
            "address_line1_check": null,
            "address_line2": null,
            "address_state": null,
            "address_zip": null,
            "address_zip_check": null,
            "brand": "Visa",
            "country": "US",
            "customer": "cus_7RRLUa32qryeH9",
            "cvc_check": "pass",
            "dynamic_last4": null,
            "exp_month": 11,
            "exp_year": 2016,
            "funding": "credit",
            "last4": "4242",
            "metadata": {
            },
            "name": null,
            "tokenization_method": null
          }
        ],
        "has_more": false,
        "total_count": 1,
        "url": "/v1/customers/cus_7RRLUa32qryeH9/sources"
      },
      "subscriptions": {
        "object": "list",
        "data": [
    
        ],
        "has_more": false,
        "total_count": 0,
        "url": "/v1/customers/cus_7RRLUa32qryeH9/subscriptions"
      }
    },
    {...},
    {...}
  ]
}
```

#Request IDs
Each API request has an associated request identifier. You can find this value in the response headers, under Request-Id. You can also find request identifiers in the URLs of individual request logs in your Dashboard. If you need to contact us about a specific request, providing the request identifier will ensure the fastest possible resolution.
Versioning
When we make backwards-incompatible changes to the API, we release new, dated versions. The current version is 2015-10-16. Read our API upgrades guide to see our API changelog and to learn more about backwards compatibility.
All requests will use your account API settings, unless you override the API version on a specific request. The changelog lists every available version. Note that events generated by API requests will always be structured according to your account API version.
To set the API version on a specific request, send a Stripe-Version header.
You can visit your Dashboard to upgrade your API version. As a precaution, use API versioning to test a new API version before committing to an upgrade.



# Resources
## User

```shell
curl http://doselect.local:8000/papi/user/<username> \
    -H Authorization: ApiKey <self_username>:<api_key>

# Replace <username> with the username, <api_key> with your api_key and <self_username> with your username
```

>The above request generates the below given json response

```json
{
  "profile": {
    "additional_data": {
      "social": {
        "blog": "",
        "github": "",
        "linkedin": ""
      },
      "top": {}
    },
    "avatar_url": "/static/usericons/male/6.png",
    "bio": null,
    "city": null,
    "companies": [],
    "country": null,
    "credits": 0,
    "dob": null,
    "interest": "[]",
    "last_login": "2015-11-25T13:20:56.387420",
    "name": "Parth",
    "phone": null,
    "primary_color": "",
    "rating": 0,
    "schools": [],
    "sex": null,
    "skype_id": null,
    "timezone": "AS/KO",
    "username": "devhacker"
  },
  "resource_uri": ""
}
```

This is the object representing Doselect Users. You can retrieve it to see a detail view of a particular Hacker,
Viewing of a list of Users is not permitted.

For the detail endoint of the user please refer to the example on the right.

This endpoint is only exposed for recruiters to view a hackers profile, any other case will return an Error.



##Team

```shell
curl http://doselect.local:8000/papi/v1/team/<team_name> \
    -H Authorization: ApiKey <self_username>:<api_key>

# Replace <team_name> with the name of the team, <api_key> with your api_key and <self_username> with your username
```

>The above request gives you the following json response

```json
{
  "profile": {
    "avatar_url": "/static/companyicons/2.png",
    "bio": null,
    "name": "papico",
    "slug": "papicomp"
  },
  "resource_uri": ""
}
```
This is the object representing DoSelect Teams. You can retrieve it to see a detail view of a particular team,
Viewing of list of Teams is not permitted

For the detail endpoint of the Team , refer to the example on the right.

This endpoint is only exposed for recruiters to view a Team profile, any other case will result in an Error.




