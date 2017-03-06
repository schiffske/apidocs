Quoting Service API
----
##### Document v1.5


Matic Quoting Service is HTTP RESTful API to provide a homeowners insurance quote.

It uses [JSON API](http://jsonapi.org/) standard for communication, so take a look,
probably there is already a library available for your platform. List of available
libraries is located [here](http://jsonapi.org/implementations/).

1. `API Client` on behalf of borrower sends `QuoteRequestsPayload` to Quoting Service.
2. `Quoting Service` creates `QuoteRequest` based on its Payload, and returns its `ID`.
3. `API Client` starts to poll status of `QuoteRequest` by `ID`.
4. There is a chance, that `Quoting Service` will prepare a quote within a 30 seconds.
   It’s up to `API Client` to render spinner, set time limit and make borrower to wait
   that time.
5. If there was no `InstaQuote™`, we will update status within 24 hours. `API Client`
   is to poll status every hour or so.

## Environments

There are two environments of Quoting Service:

* production: https://api.maticinsurance.com
* testing: https://api-uat.maticinsurance.com

Credentials are different for different instances. Endpoints and JSON formats are
equal across environments, only the hostname is different.

All further examples in this documentation are for production environment.

## Endpoints

### `POST https://api.maticinsurance.com/quote_requests`

Creates `QuoteRequest` based on `QuoteRequestPayload`. Data structure for
`QuoteRequestPayload`:

* `*customer`
    * `*name`
        * `*first_name`
        * `middle_name`
        * `*last_name`
    * `*email`
    * `phone` (any format, 10 digits)
    * `business_phone` (the same as phone)
    * `ssn`
    * `**date_of_birth` (in ‘MM/DD/YYYY’ format)
    * `fico_score`
    * `current_address`
        * `*address_line1`
        * `address_line2`
        * `*city`
        * `*state` (short version, two letters)
        * `*zip`
    * `previous_address`
        * `*address_line1`
        * `address_line2`
        * `*city`
        * `*state` (short version, two letters)
        * `*zip`
    * `residency_duration`
        * `years` (int)
        * `months` (int)
        * (at least years or months should be present)
    * `occupation` (any string)
* `*property`
    * `*address`
        * `*address_line1`
        * `address_line2`
        * `*city`
        * `*state` (short version, two letters)
        * `*zip`
    * `**property_type` (string from the list: 'single_family_detached', 'condominium',
    'duplex', 'multifamily', 'townhouse', 'other')
    * `residency_type` (‘primary_residence’, ‘second_home’, ‘investment’)
    * `purchase_date` (in ‘MM/DD/YYYY’ format)
    * `**year_built` (in ‘YYYY’ format)
    * `**stories` (in decimal format, e.g. ‘2.5’)
    * `**square_feet`
units (for multi-units property, rarely used)
* `*loan`
    * `purpose` (‘purchase’, ‘refinance’, ‘construction’)
    * `*lender_name`
    * `loan_amount`
    * `loan_number`
    * `closing_date` (in ‘MM/DD/YYYY’ format)
    * `loan_officer`
        * `*name`
        * `email`
        * `phone`
        * (at least phone or email should be present)
    * `loan_processor`
        * `*name`
        * `email`
        * `phone`
        * (at least phone or email should be present)

_* Fields are required_

_** Fields are desired_ (for a more accurate insurance rate)

---

Typical request example:

```javascript
{
  "data": {
    "type": "quote_request_payload",
    "attributes": {
      "customer": {
        "email": "josiah_schaefer8@beier.com",
        "name": {
          "first_name": "Trisha",
          "last_name": "Ernser"
        },
        "phone": "+17036560453",
        "date_of_birth": "07/15/1985"
      },
      "property": {
        "address": {
          "address_line1": "79029 Alexandra Rest",
          "address_line2": null,
          "city": "Joanastad",
          "state": "KY",
          "zip": "25401"
        },
        "property_type": "Condo",
        "year_built": 1990,
        "stories": "3.0",
        "square_feet": 2102
      },
      "loan": {
        "lender_name": "AFN"
      }
    }
  }
}
```

Maximum request example:

```javascript
{
  "data": {
    "type": "quote_request_payload",
    "attributes": {
      "customer": {
        "email": "josiah_schaefer8@beier.com",
        "name": {
          "first_name": "Trisha",
          "middle_name": null,
          "last_name": "Ernser"
        },
        "ssn": "080491832",
        "phone": "+17036560453",
        "business_phone": "+13206523456",
        "date_of_birth": "07/15/1985",
        "fico_score": 574,
        "marital_status": "single",
        "current_address": {
          "address_line1": "694 Reichel Pines",
          "address_line2": null,
          "city": "Nealburgh",
          "state": "WV",
          "zip": "37049"
        },
        "previous_address": {
          "address_line1": "3404 Wade Curve",
          "address_line2": "Suite 467",
          "city": "Ondrickaborough",
          "state": "SC",
          "zip": "76033"
        },
        "residency_duration": {
          "years": 5,
          "months": 11
        },
        "occupation": "Self Employed"
      },
      "property": {
        "address": {
          "address_line1": "79029 Alexandra Rest",
          "address_line2": null,
          "city": "Joanastad",
          "state": "KY",
          "zip": "25401"
        },
        "property_type": "Condo",
        "residency_type": "second_home",
        "purchase_date": "08/25/1991",
        "year_built": 1990,
        "stories": "3.0",
        "square_feet": 2102,
        "units": 100
      },
      "loan": {
        "purpose": "purchase",
        "lender_name": "AFN",
        "loan_amount": "787000.0",
        "loan_number": '123456789',
        "closing_date": "08/14/2016",
        "loan_officer": {
          "name": "Dennis Willms",
          "email": "timmy5@cole.name",
          "phone": "+19072328125"
        },
        "loan_processor": {
          "name": "Elta Leffler",
          "email": "ryder6@rolfson.com",
          "phone": "+19578307649"
        }
      }
    }
  }
}
```

Expected response:

```javascript
{
  "data": {
    "id": "07480def-7803-41d1-a4e7-cce95c339671",
    "type": "quote_requests"
  }
}
```

---

On invalid `QuoteRequestPayload`, service returns 422 HTTP status code with

```javascript
{
  "errors": [
    {
      "status": 422,
      "title": "ActionFailed",
      "detail": "{\"requester\":[\"must be filled\"]}"
    }
  ]
}
```

where `detail` is JSON-formatted string.

### `GET https://api.maticinsurance.com/quote_requests/:id`

Get information about previously made `QuoteRequest`. Response time is immediate.

`QuoteRequest` has the following attributes:

* `status`: may be one of the following:
    * `waiting_for_quote`
    * `quote_ready`
    * `quote_unavailable`
    * `waiting_for_policy`
    * `policy_ready`
    * `terminated`
* `reason`: extra information for `quote_unavailable` status.

`QuoteRequest` may have associated `Quote` or/and `Policy`.

`Quote` has the following attributes:

* `premium`
* `deductible`
* `coverages`: hash describing coverage of `Quote`. Consists of:
    * `dwelling`: Coverage A
    * `personal_property`: Coverage C, for HO6
    * `file`: URL for the quote in PDF format

`Policy` has the following attributes:

* `file`: URL for the policy in PDF format

---

Initial response example:

```javascript
{
  "data": {
    "id": "xztn123",
    "type": "quote_requests",
    "attributes": {
      "status": "waiting_for_quote"
    },
    "relationships": {
      "agent": {
        "data": {"id": "marci@maticinsurance.com", "type": "agents"}
      }
    }
  },
  "included": [
    {
      "id": "marci@maticinsurance.com",
      "type": "agents",
      "attributes": {
        "name": "Marci Kaufman",
        "email": "marci@maticinsurance.com",
        "phone": "(818)465-5388"
      }
    }
  ]
}
```

Full response example:

```javascript
{
  "data": {
    "id": "xztn123",
    "type": "quote_requests",
    "attributes": {
      "status": "policy_ready",
      "reason": ""
    },
    "relationships": {
      "agent": {
        "data": {"id": "marci@maticinsurance.com", "type": "agents"}
      },
      "quote": {
        "data": {"id": "xztn124", "type": "quotes"}
      },
      "policy": {
        "data": {"id": "xztn125", "type": "policies"}
      }
    }
  },
  "included": [
    {
      "id": "marci@maticinsurance.com",
      "type": "agents",
      "attributes": {
        "name": "Marci Kaufman",
        "email": "marci@maticinsurance.com",
        "phone": "(818)465-5388"
      }
    }, {
      "id": "xztn124",
      "type": "quotes",
      "attributes": {
        "premium": 100,
        "deductible": 200,
        "file": "https://url.here"
      }
    }, {
      "id": "xztn125",
      "type": "policies",
      "attributes": {
        "file": "https://url.here"
      }
    }
  ]
}
```

---

If `ID` is not found, service returns 404 HTTP status code with

```javascript
{
  "errors": [
    {
      "status": 404,
      "title": "NotFound",
      "detail": "Quote Request not found"
    }
  ]
}
```

### `PATCH https://api.maticinsurance.com/quote_requests/:id/terminated`

Cancel the previously made `QuoteRequest`. Response time is immediate.

Expected response:

```javascript
{
  "data": {
    "id": "07480def-7803-41d1-a4e7-cce95c339671",
    "type": "quote_requests"
  }
}
```

If `ID` is not found, service returns 404 HTTP status code with

```javascript
{
  "errors": [
    {
      "status": 404,
      "title": "NotFound",
      "detail": "Quote Request not found"
    }
  ]
}
```

### API Authentication

In order to provide maximum security, the API requires authentication via a
public/private RSA key pair.

Every API client will be provided with a correspondent client name and a private key.

In order to proof authentication, `API Client` should create and provide a correct
signature along a request. To create signature, `API Client` should concatenate
request information into one string and sign SHA256 hash of it with his private key.

`API Server` concatenates incoming request information into the same string, and
verifies the provided signature with public key.

Following pseudo-code would show an example of signature creating.

```ruby
client_name = 'my_api_client'
private_key = '...'
post_body = '{"data":{...}}'
timestamp = Time.current_timestamp
request_method = 'POST'
api_endpoint = '/requests'

secret_string = [client_name, request_method, api_endpoint, post_body, timestamp].join(':')

key = OpenSSL::PKey::RSA.new(private_key)
Signature = key.sign(OpenSSL::Digest::SHA256.new, secret_string)
```

We have following variables there:

* `client_name`: login name that every API Client has
* `private_key`: RSA private key that every API Client user has
* `post_body`: Body of POST request (typically JSON API string). Empty string,
  if there is no any POST data
* `timestamp`: Current time in Unix Timestamp format
* `request_method`: HTTP method of a request. Should be in UPPERCASE format
* `api_endpoint`: URI of API endpoint, starting from leading slash
* `secret_string`: Concatenated string that uniquely identifies following request.
  Delimiter between fields is colon symbol `:`
* `signature`: Created Signature of given Request

API Service expects following headers along every request:

* `X-Client`: client_name from example above
* `X-Timestamp`: timestamp from example above
* `X-Signature`: signature from example above

If authentication fails, API returns 412 HTTP status code with

```javascript
{
  "errors": [
    {
      "status": 412,
      "title": "AuthenticationError",
      "detail": "Invalid signature"
    }
  ]
}
```

### Contact Us

If you have any technical or business questions, feel free to write us
at sergiy@maticinsurance.com
