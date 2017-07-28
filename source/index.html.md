---
title: API Reference

language_tabs:
  - shell: Shell
  - javascript: Node

search: false
---

# Introduction

Welcome to Authentimate's API! You can use this API to access all of our API endpoints, such as <a href="#verify-api">Verify API</a> to verify your users' phones.

The API is organized around <a href="https://en.wikipedia.org/wiki/Representational_state_transfer">REST</a>. All requests should be made over SSL. All request and response bodies, including errors, are encoded in JSON.

# Authentication

```shell
#Authentication using HTTP basic auth.

curl "api_endpoint_here"
    -u YOUR_SECRET_KEY:
```

```shell
#Alternatively pass a Bearer token in an Authorization header.

curl "api_endpoint_here"
  -H "Authorization: Bearer YOUR_SECRET_KEY"
```

Authentication is done via your account’s API key which is found in the <a href="https://dashboard.authentimate.com/keys">dashboard</a>.

Requests are authenticated using <a href="https://en.wikipedia.org/wiki/Basic_access_authentication">HTTP Basic Auth</a>. Provide your API key as the basic auth username. You do not need to provide a password.

Alternatively you can also pass your API key as a bearer token in an `Authorization` header.

# Errors

The Authentimate API uses the following error codes:

Code | Title | Description
---- | ----- | -------
200 | OK | The request was successful.
201 | Created | The resource was successfully created.
202 | Async created | The resource was asynchronously created
400 | Bad request | Bad request
422 | Validation error | A validation error occurred.
401 | Unauthorized | Your API key is invalid.
404 | Not found | The resource does not exist.
50X | Internal Server Error | An error occurred with our API.










# Verify API

The Verify API lets you send a verification code to a user's phone number to verify that they own the phone number.

## Create Verification Request

```shell
curl "https://api.authentimate.com/v1/verifications"
    -u YOUR_SECRET_KEY:
    -d '{"phoneNumber":"16072039850"}'
```

```javascript
npm install request
```

```javascript
var request = require('request');

request.post(
    'https://api.authentimate.com/v1/verifications',
    { 
        form: { phoneNumber: '16072039850' },
        auth: { user: 'YOUR_SECRET_KEY' },
        json: true 
    },
    function (error, response, body) {
        if (!error && response.statusCode == 201) {
            // Save body.id to the user's profile in your database so you can find this user later to check the code they enter
            // Optionally store body.expires as well if you'd like to reject expired verification codes without sending a request to Authentimate
        }
    }
);
```


> The API response object looks like the following. 
> You should store the `id` property with the user in your database. You'll need to send it along when you check the user's verification code
> The `expires` property is the Unix timestamp of the expiry date. We handle the actual rejecting of expired verification requests, the expiry date is provided for your convenience in clearing out expired tokens from your database or for rejecting verification checks without needing to send a request to Authentimate.

```json
{
    "id": "ver_7b19ddcc6402fd0cd28ade60246ca60c",
    "expires": 1487207640
}
```

To verify a user's phone number, you create a Verification object. This will send a verification code to the user's phone with a code they can enter on your website to prove that they own the phone number. After they've done this, you'll need to send us a request with the code they entered as well as the `id` from the API response to verify that the code is correct.

### HTTP Request

`POST https://api.authentimate.com/v1/verifications`

### Parameters

Parameter | Type | Description
--------- | ------- | -----------
phoneNumber | <strong>string (required)<strong> | The phone number that the verification code will be sent to. NOTE: This phone number must include the country code, or we will not be able to deliver the message.











## Check Verification Code

```shell
curl "https://api.authentimate.com/v1/verifications/check?id=ver_7b19ddcc6402fd0cd28ade60246ca60c&code=928571
    -u YOUR_SECRET_KEY:
```

```javascript
npm install request
```

```javascript
var request = require('request');

request.get(
    'https://api.authentimate.com/v1/verifications/check?id=ver_7b19ddcc6402fd0cd28ade60246ca60c&code=928571',
    { 
        auth: { user: 'YOUR_SECRET_KEY' },
        json: true 
    },
    function (error, response, body) {
        if (!error && response.statusCode == 200) {
            // Check body.verified for the status
            // body.verified will be true if the code was correct, and false if the code was incorrect.
        }
    }
);
```


> The API response object looks like the following. 
> You should check the `verified` property in the response body to determine whether the code the user entered was correct or not. A correct code will return `verified: true`, while an incorrect code will return `verified: false`.

```json
{
    "verified": true
}
```

After you create a verification request for a user, you'll need to prompt them to enter the code that we sent to their phone. In order to verify that this code is correct, you send it to Authentimate's API along with the `id` returned in the API response.

### HTTP Request

`GET https://api.authentimate.com/v1/verifications/check`

### Parameters

Parameter | Type | Description
--------- | ------- | -----------
id | <strong>string (required)<strong> | The identifier returned in the `id` property in the Create Verification Request endpoint
code | <strong>string (required)<strong> | The code that your user entered after you generated a Verification request for their phone number.




