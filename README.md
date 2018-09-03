# Kudobuzz Api Standards

* [Guidelines](#guidelines)
* [RESTful URLs](#restful-urls)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [HTTP Status Codes](#http-status-codes)
* [Error handling](#error-handling)
* [Versions](#versions)
* [Record limits](#record-limits)
* [Request & Response Examples](#request--response-examples)
* [Rate Limiting](#rate-limiting)
* [Idempotent](#idempotent)
* [Modeling Actions](#modeling-actions)
* [Security](#security)
* [Api Docs](#docs)
* [Amazing Rest Apis](#examples)

## Guidelines

This document provides guidelines and examples for Kudobuzz APIs, encouraging consistency, maintainability, and best practices across services. Our APIs aim to balance a truly RESTful API interface with a positive developer experience (DX).

This document borrows heavily from:
* [Designing HTTP Interfaces and RESTful Web Services](https://www.youtube.com/watch?v=zEyg0TnieLg)
* [API Facade Pattern](http://apigee.com/about/resources/ebooks/api-fa%C3%A7ade-pattern), by Brian Mulloy, Apigee
* [Web API Design - Missing Link](https://pages.apigee.com/rs/351-WXY-166/images/Web-design-the-missing-link-ebook-2016-11.pdf), by Apigee
* [Web API Design](https://pages.apigee.com/rs/apigee/images/api-design-ebook-2012-03.pdf), by Apigee
* [Fielding's Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [Resource Modeling](https://www.thoughtworks.com/insights/blog/rest-api-design-resource-modeling)
* [Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api) by Vinay

## RESTful URLs

### General guidelines for RESTful URLs
* A URL identifies a resource.
* URLs should include nouns, not verbs.
* Use plural nouns only for consistency (no singular nouns).
* Use HTTP verbs (GET, POST, PUT, DELETE) to operate on the collections and elements.
* You shouldn’t need to go deeper than resource/identifier/resource.
* Put the version number at the base of your URL, for example http://api.kudobuzz.com/v1/path/to/resource
* URL v. header:
    * If it changes the logic you write to handle the response, put it in the URL.
    * If it doesn’t change the logic for each response, like OAuth info, put it in the header.
* Specify optional fields in a comma separated list.
* Formats should be in the form of api/v2/resource/{id}
* Design resource representations. Don’t simply represent database tables.
* Support field projections on resources. Allow clients to reduce the number of fields that come back in the response.

### Good URL examples
* List of magazines:
    * GET http://api.kudobuzz.com/v1/magazines
* Filtering and Sorting:
    * GET http://api.kudobuzz.com/v1/magazines?year=2011&sort=-created_at,year
    * GET http://api.kudobuzz.com/api/v1/magazines?topic=economy&year=2011
* A single magazine in JSON format:
    * GET http://api.kudobuzz.com/api/v1/magazines/1234
* All articles in (or belonging to) this magazine:
    * GET http://api.kudobuzz.com/v1/magazines/1234/
* Specify optional fields in a comma separated list:
    * GET http://api.kudobuzz.com/v1/magazines/1234?fields=title,subtitle,date
* Add a new article to a particular magazine:
    * POST http://api.kudobuzz.com/v1/magazines/1234/articles
* Search for articles
    * GET https://api.kudobuzz.com/v1/acticles?q=food

### Bad URL examples
* Non-plural noun:
    * http://api.kudobuzz.com/v1/magazine
    * http://api.kudobuzz.com/v1/magazine/1234
    * http://api.kudobuzz.com/v1/publisher/magazine/1234
* Verb in URL:
    * http://api.kudobuzz.com/v1/magazine/1234/create
* Filter outside of query string
    * http://api.kudobuzz.com/v1/magazines/2011/desc

## HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.
The action taken on the representation will be contextual to the media type being worked on and its current state. Here's an example of how HTTP verbs map to create, read, update, delete operations in a particular context:

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE      | DELETE |
| /dogs       | Create new dogs | List dogs | Bulk update | Delete all dogs |
| /dogs/1234  | Error           | Show Bo   | If exists, update Bo; If not, error | Delete Bo |

(Example from Web API Design, by Brian Mulloy, Apigee.)


## Responses

* No values in keys
* No internal-specific names (e.g. "node" and "taxonomy term")
* Metadata should only contain direct properties of the response set, not properties of the members of the response set

### Good examples

No values in keys:

    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],


### Bad examples

Values in keys:

    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],

## Http Status Codes
There are currently over 70 status codes, it is difficult for developers to memorize all of these codes. Use only a small subset of common status codes.

| Status Code  | Description                                                            |
|--------------|------------------------------------------------------------------------|
| 200 OK       | The request was successfully processed                                 |
| 201 Created  | The request has been fulfilled and the new resource has been created   |
| 400 Bad Request | The request was not understood by server due bad  syntax, request contains invalid and missing request details. |
| 404  Not Found | The resource was not found |
| 401 UnAuthorised | The request is missing the necessary authentication details |
| 403 Forbidden | The request credentials does not have the permission to perform that action on the resource |
| 422 UnProcessable Entity| The request body is well formed but contains semantical errors. The error response body will contain more details
| 429 Too Many Requests | The request was not accepted because the application has exceeded the rate limit. 

## Error handling

Error responses can include message for the developer,an optional internal error code (corresponding to some specific internally determined ID), details for validation errors. For example:

    {
      "message" : "Verbose, plain language description of the problem. Provide developers
       suggestions about how to solve their problems here",
      "code" : "444444", // this is not the same statusCodes
      "details": [{param, messeage, value}] // Validation Error details
    }

## Versions
* Never release an Public API Gateway  without a version number.
* Versions should be integers, not decimal numbers, prefixed with ‘v’. For example:
    * Good: v1, v2, v3
    * Bad: v-1.1, v1.2, 1.3
* Maintain APIs at least one version back.

## Record limits

* If no limit is specified, return results with a default limit.
* To get records 51 through 75 do this:
    * http://api.kudobuzz.com/v1/magazines?limit=25&cursor=383884834
    * cursor=383884834  means, ‘everything greater than 383884834’
    * limit=25 means, ‘return a maximum of 25 records’

Information about record limits and total available count should also be included in the response. Example:

    {
        "metadata": {
          "count": 227       
        },
        "data": []
    }

## Request & Response Examples

### API Resources

  - [GET /magazines](#get-magazines)
  - [GET /magazines/[id]](#get-magazinesid)
  - [POST /magazines/[id]/articles](#post-magazinesidarticles)

### GET /magazines

Example: http://example.gov/api/v1/magazines

Response body:

    {
        "metadata": {
           "count": 123
        },
        "data": [
            {
                "id": "1234",
                "type": "magazine",
                "title": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /magazines/[id]

Example: http:/api.kudouzz.com/v1/api/magazines/[id]

Response body:

    {
      "data": {
        "id": "1234",
        "type": "magazine",
        "title": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created_at": "1231621302",
        "updated_at": "12321621302"
     }
    }



### POST /magazines/[id]/articles

Example: Create – POST  http://api.kudobuzz.com/v1/magazines/[id]/articles

Request body:

    { 
    "data": {
       "title": "Raising Revenue",
       "author_first_name": "Jane",
       "author_last_name": "Smith",
       "author_email": "jane.smith@example.gov",
       "year": "2012",
       "month": "August",
       "day": "18",
       "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "
       }
    }
    
 ## Rate Limiting
 Ensure that your apis' are protected against sudden increase in request.
 Use the following headers to notify consumers if they exceeded the number request in a specif interval.
 
 `X-Rate-Limit-Limit` -  the number of request allowed in a given interval
 
 `X-Rate-Limit-Remaining`- the number of request remaining
 
 `X-Rate-Limit-Reset`- the next time for reset
 
 
 ## Idempotent
 Ensure that your GET, PUT, and DELETE operations are all [idempotent](http://www.restapitutorial.com/lessons/idempotency.html).  There should be no adverse side affects from operations.
 
 ## Security
 Use [OAuth2](http://oauth.net/2/) to secure your API.
   * Use a Bearer token for authentication.
   * Require HTTPS / TLS / SSL to access your APIs. OAuth2 Bearer tokens demand it. Unencrypted communication over HTTP allows for simple eavesdroppping and impersonation.
 
## Docs
You write APIs so others can use them, benefit from them. Providing an API documentation for your REST APIs are crucial. 
Use [Api Blueprint](https://apiblueprint.org/)

## Examples
[Github](https://developer.github.com/v3/)

[Twilio](https://www.twilio.com/docs/usage/api)

[Stripe](https://stripe.com/docs/api)

[DigitalOcean](https://developers.digitalocean.com/documentation/v2/#introduction)

