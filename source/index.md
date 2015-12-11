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

The DoSelect Partner API a.k.a. PAPI is organized around REST. Our API has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. We support cross-origin resource sharing, allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API key in any public website's client-side code). JSON is returned by all API responses, including errors, although our API libraries convert responses to appropriate language-specific objects.
[//]: # (This may be the most platform independent comment
To make the API as explorable as possible, accounts have test mode and live mode API keys. There is no "switch" for changing between modes, just use the appropriate key to perform a live or test transaction. Requests made with test mode credentials never hit the banking networks and incur no cost.)

# Authentication



>cURL uses the -u flag to pass basic auth credentials (adding a colon after your API key prevents cURL from asking for a password).A sample test API key is included in all the examples on this page, so you can test any example right away. To test requests using your account, replace the sample API key with your actual API key.

```shell
# With curl, you can just pass the correct header with each request
curl "api_endpoint_here" \
    -H "Authorization: ApiKey username:apikey"
```

Authenticate your account when using the API by including your secret API key in the request. You can manage your API keys in the Dashboard. Your API keys carry many privileges, so be sure to keep them secret! Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth.
Provide your API key in the Authorization Header along with your username. You do not need to provide a password.
All API requests must be made over HTTPS. Calls made over plain HTTP will fail. API requests without authentication will also fail. Also There must exist an `API_KEY` in the Authorization header and a `CONSUMER_KEY` in arguments with each request. PAPI would return `401-Unathorized` if either of them is not provided.

`API_KEY` : This is give to the Parters, so that they can make requests from their portal.
`CONSUMER_KEY`: This is the key given to the consumer interacting with our partner which asserts that the request the partner is making on behalf of the CONSUMER is valid, i.e. he has the permission to make that request from the consumer end.




<aside class="notice">
You must replace <code>apikey</code> with your personal API key.
You must replace <code>username</code> with your username.
</aside>


#Errors

PAPI uses conventional HTTP response codes to indicate the success or failure of an API request. In general, codes in the 2xx range indicate success, codes in the 4xx range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.), and codes in the 5xx range indicate an error with DoSelect's servers (these are rare).
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
500, 502, 503, 504 - Server Errors|Something went wrong on DoSelect's end. (These are rare.)



>###Error TYPES
Error | Description
-------|------------
api_connection_error|Failure to connect to PAPI.
api_error|API errors cover any other type of problem (e.g., a temporary problem with DoSelect's servers) and are extremely uncommon.
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



#Idempotent Requests

The API supports idempotency for safely retrying requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.
To perform an idempotent request, attach a unique key to any POST request made to the API via the Idempotency-Key: <key> header.
How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters. The keys expire after 24 hours.


```shell
curl https://api.doselect.com/v1/charges
   -u sk_test_BQokikJOvBiI2HlWgH4olfQ2:
   -H "Idempotency-Key: 48YnJFsWVbZtXi9A"
   -d amount=400
   -d source=tok_17BxFu2eZvKYlo2C3qeOBMKT
   -d currency=usd
```

#Metadata

Updatable DoSelect objects—including Account, Charge, Customer, Refund, Subscription, and Transfer—have a metadata parameter. You can use this parameter to attach key-value data to these DoSelect objects.
Metadata is useful for storing additional, structured information on an object. As an example, you could store your user's full name and corresponding unique identifier from your system on a DoSelect Customer object. Metadata is not used by DoSelect (e.g., to authorize or decline a charge), and won't be seen by your users unless you choose to show it to them.
Some of the objects listed above also support a description parameter. You can use the description parameter to annotate a charge, for example, with a human-readable description, such as "2 shirts for test@example.com". Unlike metadata, description is a single string, and your users may see it (e.g., in email receipts DoSelect sends on your behalf).
Note: You can have up to 20 keys, with key names up to 40 characters long and values up to 500 characters long.
SAMPLE METADATA USE CASES
Link IDs
Attach your system's unique IDs to a DoSelect object for easy lookups. Add your order number to a charge, your user ID to a customer or recipient, or a unique receipt number to a transfer, for example.
Refund papertrails
Store information about why a refund was created, and by whom.
Customer details
Annotate a customer by storing the customer's phone number for your later use.


```shell
curl https://api.doselect.com/v1/charges
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

All top-level API resources have support for bulk fetches via "list" API methods. For instance you can list Teams and Invites. These list API methods share a common structure, taking at least these three parameters: limit, starting_after, and ending_before.
DoSelect utilizes cursor-based pagination via the starting_after and ending_before parameters. Both take an existing object ID value (see below). The ending_before parameter returns objects created before the named object, in descending chronological order. The starting_after parameter returns objects created after the named object, in ascending chronological order. If both parameters are provided, only ending_before is used.


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
curl https://api.doselect.com/v1/customers?limit=3
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
To set the API version on a specific request, send a DoSelect-Version header.
You can visit your Dashboard to upgrade your API version. As a precaution, use API versioning to test a new API version before committing to an upgrade.

#Flows
##Partner Consumer flow
+ Each Team registered with us will have a CONSUMER_KEY,which can be accessed from the Accounts Section.
+ Each Partner will have an API_KEY through which they can access our partner api (PAPI)
+ In order to do any activity on behalf of the Team(Company) accociated with the Partner, the Partner
  must send the Team's CONSUMER_KEY along with his API_KEY with every request.
+ The API_KEY will authenticate the Partner to use our API while the CONSUMER_KEY authorizes the Partner to make  
  changes for the Team having the CONSUMER_KEY.
+ API_KEY and CONSUMER_KEY gives you several privileges and must be kept PRIVATE at all times.

##Test Creation Flow
+ Tests can be Created/Updated/Deleted by the partner for the consumer using the above flow
+ In order to do any of the operations on the test Resource you must pass the -
  + Team which you want to do the operations one
  + The name of the Tests
  + The Duration of the Test
  + Problem set from public or private problem repository

##Problem Addition flow
There are three places from where you can choose problems that should be added to the Test.

+ Private Library -You can Edit/Delete/Create the Problems in this library. This is only visible to you
+ Community Library -You can only Use problems from this library in your tests. This library is visible 
  to all the  Teams and Partners
+ Problem Market Place- You can Buy problem sets suiting you requirement from here which will be then 
  added to your Private Library.

##Create Invite for a Team's Test
To create an invite you must pass the email for invite and the test name for which the email is invited.
Deletion of an invite is not accepted

##Test Report 
Partners are allowed to read (both list and detail view) for each test


# Resources
## User

```shell

curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/user/USERNAME?key=CONSUMER_KEY|python -m json.tool

# Replace USERNAME with the username you want to retrieve, API_KEY with your api_key and YOUR_USERNAME with your username
# Replace CONSUMER_KEY with your customer consumer key
#Piping `python -m json.tool` is optional

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

TODO: ADD ERRORS

##Team

```shell
curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/team/TEAM_NAME?key=CONSUMER_KEY|python -m json.tool

## Replace TEAM_NAME with the team name(on DOSELECT)you want to  you want to retrieve, API_KEY with your api_key and YOUR_USERNAME with your username
# Replace CONSUMER_KEY with your customer consumer key
#Piping `python -m json.tool` is optional
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

TODO: List Errors


## Test

```shell

curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/test/?key=CONSUMER_KEY|python -m json.tool

# Replace API_KEY with your api_key and YOUR_USERNAME with your username
# Replace CONSUMER_KEY with your customer consumer key
#Piping `python -m json.tool` is optional

```

>The above request generates the below given json response

```json
{
    "meta": {
        "limit": 50,
        "next": null,
        "offset": 0,
        "page_number": 1,
        "previous": null,
        "total_count": 4,
        "total_pages": 1
    },
    "objects": [
        {
            "creator": "9518774bbce2779fb5c20e4c0",
            "cutoff": 0,
            "duration": 100,
            "instructions": "test to check read operation",
            "is_activated": true,
            "name": "papitest",
            "owner": "localenv",
            "resource_uri": "/papi/v1/test/1",
            "sections": "[{u'content': u'con', u'title': u'test 2'}, {u'content': u'test contemt', u'title': u'test title'}]",
            "slug": "50vj0"
        },
        {
            "creator": "test_user",
            "cutoff": 0,
            "duration": 0,
            "instructions": "testslkjglsk",
            "is_activated": false,
            "name": "test11",
            "owner": "Camp",
            "resource_uri": "/papi/v1/test/2",
            "sections": "",
            "slug": "test11"
        },
        {
            "creator": "test_user",
            "cutoff": 0,
            "duration": 0,
            "instructions": "testslkjglsk",
            "is_activated": true,
            "name": "test111",
            "owner": "Camp",
            "resource_uri": "/papi/v1/test/4",
            "sections": "",
            "slug": "test111"
        },
        {
            "creator": "test_user",
            "cutoff": 0,
            "duration": 0,
            "instructions": "testselkjglsk",
            "is_activated": true,
            "name": "test111",
            "owner": "Camp",
            "resource_uri": "/papi/v1/test/100",
            "sections": "",
            "slug": "test11111"
        }
    ]
}

```

This endpoint gives access to Doselect Tests. You can retrieve it to see a list view of the tests on Doselect,
see example on the right


```shell

curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/test/TEST_ID?key=CONSUMER_KEY|python -m json.tool

# Replace API_KEY with your api_key and YOUR_USERNAME with your username
# Replace TEST_ID with the test id of the Test Resource you want to access
# Replace CONSUMER_KEY with your customer consumer key
#Piping `python -m json.tool` is optional

```

>The above request generates the below given json response

```json
{
  "creator": "9518774bbce2779fb5c20e4c0",
  "cutoff": 0,
  "duration": 100,
  "instructions": "test to check read operation",
  "is_activated": true,
  "name": "papitest",
  "owner": "localenv",
  "resource_uri": "/papi/v1/test/1",
  "sections": "[{u'content': u'con', u'title': u'test 2'}, {u'content': u'test contemt', u'title': u'test title'}]",
  "slug": "50vj0"
}
```
To generate a detail view of the Test Resorce follow the example on the right
TODO: ADD ERRORS

## Invites

```shell

curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/invite/?key=CONSUMER_KEY|python -m json.tool

# Replace API_KEY with your api_key and YOUR_USERNAME with your username
# Replace CONSUMER_KEY with your customer consumer key
#Piping `python -m json.tool` is optional

```

>The above request generates the below given json response

```json
{
  "meta": {
    "limit": 50,
    "next": null,
    "offset": 0,
    "page_number": 1,
    "previous": null,
    "total_count": 1,
    "total_pages": 1
  },
  "objects": [
    {
      "accepted": false,
      "access_code": "a3a2131962154d48b2bfa814c7f7bbf1",
      "company": "localenv",
      "creator": "9518774bbce2779fb5c20e4c0",
      "email": "some-email@doselect.com",
      "expiry": "2015-12-11T11:17:45.444623",
      "rejected": false,
      "resource_uri": "/papi/v1/invite/1",
      "times_reset": 0,
      "user": "devhacker"
    }
  ]
}
```

This gives out a list of invites in the database. You can retrieve it to see a list-view of the Invite Resource

`curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/invite/INVITE_ID?key=CONSUMER_KEY|python -m json.tool`


```shell

curl -H "Authorization: ApiKey YOUR_USERNAME:API_KEY" \
  http://doselect.local:8000/papi/v1/invite/INVITE_ID?key=CONSUMER_KEY|python -m json.tool

# Replace API_KEY with your api_key and YOUR_USERNAME with your username
# Replace INVITE_ID with the invite id you want to access
# Replace CONSUMER_KEY with your customer consumer key
#Piping `python -m json.tool` is optional

```
>The above request generates the below given json response


```json
{
  "accepted": false,
  "access_code": "a3a2131962154d48b2bfa814c7f7bbf1",
  "company": "localenv",
  "creator": "9518774bbce2779fb5c20e4c0",
  "email": "some-email@doselect.com",
  "expiry": "2015-12-11T11:17:45.444623",
  "rejected": false,
  "resource_uri": "/papi/v1/invite/1",
  "times_reset": 0,
  "user": "devhacker"
}
```
To generate a detail view of the Test Resorce follow the example on the right.

TODO: ADD ERRORS



