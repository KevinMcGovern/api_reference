---
title: API Reference

language_tabs:
  - shell: Shell
  - javascript: Node
  - python: Python
  - ruby: Ruby

search: false
---

# Introduction

Welcome to Authentimate's API! You can use this API to access all of our API endpoints, such as <a href="#recover-api">Recover API</a> to reset passwords for your users.

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

Authentication is done via your account’s API key which is found in the <a href="https://app.authentimate.com/keys">dashboard</a>.

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

# Webhooks

> Your webhook URL may look something like this:

```json
"https://myserver.com/api/webhooks/authentimate"
```

For some APIs, such as <a href="#recover-api">Recover API</a>, the result of the user's action is asynchronous (since they can choose to perform the action whenever they feel like it). For these requests, we will call the webhook URL you provide to us when we receive input from the user.

You can set a webhook URL in your <a href="https://app.authentimate.com/project">project settings</a> to be used with all requests.

If you return anything other than a `HTTP 200` status to the webhook POST then we will report an error to the user and they will need to submit the form again.

# Recover API

The Recover API lets you send a password reset form to a user's email address.

## Create Reset Request

```shell
curl "https://recover.authentimate.com/v1/resets"
    -u YOUR_SECRET_KEY:
    -d '{"email":"test@example.com"}'
```

```javascript
npm install request
```

```javascript
var request = require('request');

request.post(
    'https://recover.authentimate.com/v1/resets',
    { 
        form: { email: 'test@example.com' },
        auth: { user: 'YOUR_SECRET_KEY' },
        json: true 
    },
    function (error, response, body) {
        if (!error && response.statusCode == 201) {
            // Save body.id to the user's profile in your database
            // Optionally save body.expires to the user profile as well
        }
    }
);
```

```python
pip install requests
```

```python
import requests
r = requests.post('https://recover.authentimate.com/v1/resets', data={ 'email':'test@example.com' }, auth=('YOUR_SECRET_KEY', ''))
r.json()
```

```ruby
require 'net/http'
require 'uri'

url = URI.parse('https://recover.authentimate.com/v1/resets')
req = Net::HTTP::Post.new(url.path)
req.basic_auth 'YOUR_SECRET_KEY', ''
req.use_ssl = true
req.form_data({'email' => 'test@example.com'})

resp = Net::HTTP.new(url.host, url.port).start {|http| http.request(req) }
puts resp
```


> The API response object looks like the following. 
> You should store the `id` property with the user in your database. The result webhook will contain the same `id` so that you can look up the user and update their password. 
> The `expires` property is the Unix timestamp of the expiry date. We handle the actual rejecting of expired password resets, the expiry date is provided for your convenience in clearing out expired tokens from your database.

```json
{
    "id": "7b19ddcc6402fd0cd28ade60246ca60c",
    "expires": 1487207640
}
```

> In order to receive the user's updated password, you'll need to provide us with an endpoint on your server where we can send the password. The webhook request body looks like the following. As you can see, the `id` property in the webhook matches the `id` property you receive in the API response object. You can use this identifier to look up the user and update their password.

```json
{
    "id": "7b19ddcc6402fd0cd28ade60246ca60c",
    "password": "h8A3fJ9d9L"
}
```

To reset a user's password, you create a Reset object. This will send an email to the user with a link they can follow to enter a new password. After they've done this, we'll send the new password to the webhook URL that you set in your <a href="https://app.authentimate.com/project">project settings</a>.

### HTTP Request

`POST https://recover.authentimate.com/v1/resets`

### Parameters

Parameter | Type | Description
--------- | ------- | -----------
email | <strong>string (required)<strong> | The email address that a password reset link will be sent to

