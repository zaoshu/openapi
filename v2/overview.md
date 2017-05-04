# Overview

This describes the resources that make up the official zaoshu API v2.

* [Version](#version)
* [Schema](#schema)
* [Parameters](#parameters)
* [Root Endpoint](#root-endpoint)
* [Client Errors](#client-errors)
* [HTTP Verbs](#http-verbs)
* [Authentication](#authentication)
* [Pagination](#pagination)
* [Rate Limiting](#rate-limiting)
* [User Agent Required](#user-agent-required)

## Version

By default, all requests receive the **v2** version of the API. We encourage you to explicitly request this version via the Accept header.

    Accept: application/vnd.zaoshu.v2+json

## Schema 

All API access is over HTTPS, and accessed from the `https://openapi.zaoshu.io`. All data is sent and received as JSON.

    curl -i https://openapi.zaoshu.io

    HTTP/1.1 200 OK
    Server: nginx
    Date: Sat, 21 Oct 2017 23:59:00 GMT
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Status: 200 OK

    Content-Length: 5
    Cache-Control: max-age=0, private, must-revalidate

## Parameters

Many API methods take optional parameters. For GET requests, any parameters not specified as a segment in the path can be passed as an HTTP query string parameter:

    curl -i "https://openapi.zaoshu.io/instance/d872c7bbfca643a0834d7d2241f69e2b?status=running"

In this example, the value 'd872c7bbfca643a0834d7d2241f69e2b' are provided for the `:instance_id` parameter in the path while `:status` is passed in the query string.

For POST, PATCH, PUT, and DELETE requests, parameters not included in the URL should be encoded as JSON with a Content-Type of 'application/json':

    curl -i -u username -d '{"scopes":["public_repo"]}' https://openapi.zaoshu.io/authorizations

## Root Endpoint

You can issue a `GET` request to the root endpoint to get all the endpoint categories that the API supports:

    curl https://openapi.zaoshu.io

## Client Errors

There are three possible types of client errors on API calls that receive request bodies:

1.  Sending invalid JSON will result in a `400 Bad Request` response.

        HTTP/1.1 400 Bad Request

        {"message":"error occurred when parsing JSON"}

2.  Sending the wrong type of JSON values will result in a `400 Bad Request` response.

        HTTP/1.1 400 Bad Request

        {"message":"body should be a JSON object"}

3.  Sending invalid fields will result in a `422 Unprocessable Entity` response.

        HTTP/1.1 422 Unprocessable Entity

        {
          "message": "Validation Failed",
          "errors": [
            {
              "resource": "instance",
              "field": "id",
              "code": "missing_field"
            }
          ]
        }

All error objects have resource and field properties so that your client can tell what the problem is. There's also an error code to let you know what is wrong with the field. These are the possible validation error codes:

<table>
<thead>
<tr>
<th>Error Name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>`missing`</td>
<td>Resource does not exist.</td>
</tr>
<tr>
<td>`missing_field`</td>
<td>Required field on a resource has not been set.</td>
</tr>
<tr>
<td>`invalid`</td>
<td>Formatting of a field is invalid. The documentation for that resource should be able to give you more specific information.</td>
</tr>
<tr>
<td>`already_exists`</td>
<td>This means another resource has the same value as this field. This can happen in resources that must have some unique key (such as Label names).</td>
</tr>
</tbody>
</table>

Resources may also send custom validation errors (where `code` is `custom`). Custom errors will always have a `message` field describing the error, and most errors will also include a `documentation_url` field pointing to some content that might help you resolve the error.

## HTTP Verbs

Where possible, API v2 strives to use appropriate HTTP verbs for each action.

<table>
<thead>
<tr>
<th>Verb</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>`HEAD`</td>
<td>Can be issued against any resource to get just the HTTP header info.</td>
</tr>
<tr>
<td>`GET`</td>
<td>Used for retrieving resources.</td>
</tr>
<tr>
<td>`POST`</td>
<td>Used for creating resources.</td>
</tr>
<tr>
<td>`PATCH`</td>
<td>Used for updating resources with partial JSON data. For instance, an Issue resource has `title` and `body` attributes. A PATCH request may accept one or more of the attributes to update the resource. PATCH is a relatively new and uncommon HTTP verb, so resource endpoints also accept `POST` requests.</td>
</tr>
<tr>
<td>`PUT`</td>
<td>Used for replacing resources or collections. For `PUT` requests with no `body` attribute, be sure to set the `Content-Length` header to zero.</td>
</tr>
<tr>
<td>`DELETE`</td>
<td>Used for deleting resources.</td>
</tr>
</tbody>
</table>

## Authentication

There are two ways to authenticate through zaoshu API v2. Requests that require authentication will return `404 Not Found`, instead of `403 Forbidden`, in some places. 

## Basic Authentication with Key/Secret

	curl -u "key:secret" https://openapi.zaoshu.io

## OAuth2 Token (sent in a header)

	curl -H "Authorization: token OAUTH-TOKEN" https://openapi.zaoshu.io

## Failed login limit

Authenticating with invalid credentials will return `401 Unauthorized`:

	curl -i https://openapi.zaoshu.io -u foo:bar
	HTTP/1.1 401 Unauthorized

	{
        "message": "Bad credentials"
	}

After detecting several requests with invalid credentials within a short period, the API will temporarily reject all authentication attempts for that user (including ones with valid credentials) with `403 Forbidden`:

	curl -i https://openapi.zaoshu.io -u valid_key:valid_secret
	HTTP/1.1 403 Forbidden

	{
        "message": "Maximum number of login attempts exceeded. Please try again later."
	}


## Pagination

Requests that return multiple items will be paginated to 30 items by default. You can specify further pages with the `?page` parameter. For some resources, you can also set a custom page size up to 100 with the `?per_page` parameter. Note that for technical reasons not all endpoints respect the `?per_page` parameter.

    curl 'https://openapi.zaoshu.io/instances?page=2&per_page=100'

Note that page numbering is 1-based and that omitting the `?page` parameter will return the first page.

## Rate Limiting

For requests using Basic Authentication or OAuth, you can make up to 5,000 requests per hour. For unauthenticated requests, the rate limit allows you to make up to 60 requests per hour. Unauthenticated requests are associated with your IP address, and not the user making requests. 

You can check the returned HTTP headers of any API request to see your current rate limit status:

curl -i https://openapi.zaoshu.io/users/whatever
HTTP/1.1 200 OK
Date: Mon, 01 Jul 2017 17:27:06 GMT
Status: 200 OK
X-RateLimit-Limit: 6000
X-RateLimit-Remaining: 4500 
X-RateLimit-Reset: 137270087

The headers tell you everything you need to know about your current rate limit status:

<table>
<thead>
<tr>
<th>Header Name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>`X-RateLimit-Limit`</td>
<td>The maximum number of requests that the consumer is permitted to make per hour.</td>
</tr>
<tr>
<td>`X-RateLimit-Remaining`</td>
<td>The number of requests remaining in the current rate limit window.</td>
</tr>
<tr>
<td>`X-RateLimit-Reset`</td>
<td>The time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time).</td>
</tr>
</tbody>
</table>

Once you go over the rate limit you will receive an error response:

	HTTP/1.1 403 Forbidden
	Date: Tue, 20 Aug 2017 14:50:41 GMT
	Status: 403 Forbidden
	X-RateLimit-Limit: 6000
	X-RateLimit-Remaining: 0
	X-RateLimit-Reset: 1377013266

	{
        "message": "API rate limit exceeded for xxx.xxx.xxx.xxx."
	}


You can also [check your rate limit status](/v2/rate_limit) without incurring an API hit.  

## Abuse Rate Limits

To protect the quality of service from zaoshu, additional rate limits may apply to some actions. For example, rapidly creating content, polling aggressively instead of using webhooks, making API calls with a high concurrency, or repeatedly requesting data that is computationally expensive may result in abuse rate limiting.

It is not intended for this rate limit to interfere with any legitimate use of the API. Your normal [rate limits](#rate-limiting) should be the only limit you target. 

If your application triggers this rate limit, you'll receive an informative response:

	HTTP/1.1 403 Forbidden
	Content-Type: application/json; charset=utf-8
	Connection: close

	{
        "message": "You have triggered an abuse detection mechanism and have been temporarily blocked from content creation. Please retry your request again later."
	}


## User Agent Required

All API requests MUST include a valid `User-Agent` header. Requests with no `User-Agent` header will be rejected. We request that you use your zaoshu username, or the name of your application, for the `User-Agent` header value. This allows us to contact you if there are problems.

Here's an example:

    User-Agent: Awesome-Octocat-App


If you provide an invalid `User-Agent` header, you will receive a `403 Forbidden` response:

	curl -iH 'User-Agent: ' https://openapi.zaoshu.io
	HTTP/1.0 403 Forbidden
	Connection: close
	Content-Type: text/html

	Request forbidden by administrative rules.
	Please make sure your request has a User-Agent header.