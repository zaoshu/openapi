# 概述 

This describes the resources that make up the official zaoshu API v2.
该文档包含了 OpenAPI V2 的基本概念和所需资源的描述。


* [纲要](#schema)
* [参数传递](#parameters)
* [访问根节点](#root-endpoint)
* [客户端错误码](#client-errors)
* [HTTP 动词语义](#http-verbs)
* [授权验证](#authentication)
* [分页](#pagination)
* [请求频率限制](#rate-limiting)
<!--* [User Agent Required](#user-agent-required)-->

## 纲要

所有的 API 都将通过 HTTPS 从 `https://openapi.zaoshu.io/v2` 请求。往返数据内容均为 JSON 格式。

    curl -i https://openapi.zaoshu.io/v2

    HTTP/1.1 200 OK
    Server: nginx
    Date: Sat, 21 Oct 2017 23:59:00 GMT
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Status: 200 OK

    Content-Length: 5
    Cache-Control: max-age=0, private, must-revalidate

## 参数传递

很多的 API 接口接受可选参数的传递。对于 GET 请求，所有未在请求路径中以分段指定的参数，都可以通过 HTTP 请求字符串传递：

    curl -i "https://openapi.zaoshu.io/v2/instance/d872c7bbfca643a0834d7d2241f69e2b?status=running"

在该示例中，请求路径中的值 'd872c7bbfca643a0834d7d2241f69e2b' 将作为参数 `:id` 的传入，而 `:status` 则通过请求字符串传递。

对于 POST、PATCH、PUT 和 DELETE 请求，在 URL 请求字符串中未被包括的部分将会被编码后以 JSON 的方式传递：

    curl -i -u username -d '{"result_notify_uri":["https://your-webhook.com"]}' https://openapi.zaoshu.io/v2/instance/d872c7bbfca643a0834d7d2241f69e2b/run

## 访问根节点 

通过 `GET` 请求根节点能够得到所有的 API 接口项：

    curl https://openapi.zaoshu.io/v2

## 客户端错误码 

<!--
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

| Error Name | Description |
| --- | --- |
| `missing` | Resource does not exist. |
| `missing_field` | Required field on a resource has not been set. |
| `invalid` | Formatting of a field is invalid. The documentation for that resource should be able to give you more specific information. |
| `already_exists` | This means another resource has the same value as this field. This can happen in resources that must have some unique key (such as Label names). |

Resources may also send custom validation errors (where `code` is `custom`). Custom errors will always have a `message` field describing the error, and most errors will also include a `documentation_url` field pointing to some content that might help you resolve the error.-->

## HTTP 动词语义

Where possible, API v2 strives to use appropriate HTTP verbs for each action.
API v2 会尽可能为请求动作分配合适的 HTTP 动词。

| 动词 | 描述 |
| --- | --- |
<!--| `HEAD` | Can be issued against any resource to get just the HTTP header info. |-->
| `GET` | 用于资源索取 |
| `POST` | 用于资源创建 |
| `PATCH` | Used for updating resources with partial JSON data. For instance, an Instance resource has `title` and `result_notify_uriy` attributes. A PATCH request may accept one or more of the attributes to update the resource. 用于通过提供部分数据的 JSON 对资源进行更新。例如，爬虫实例有 `title` 和 `result_notify_uri` 两个属性。`PATCH` 请求可以接受一个或多个属性的参数传入来更新对应的资源。|
| `PUT` | Used for replacing resources or collections. 用于替换掉对应的资源。|
| `DELETE` | Used for deleting resources.用于（软）删除对应的资源 |

## 授权校验 

You can get API key and secret from [Dashboard](https://dashboard.zaoshu.io/?settings).
你可以在[控制面板->设置](https://dashboard.zaoshu.io/?settings)中获取 API 授权的 key 与 secret。


See [Authentication](authentication.md).
详情请查看[授权校验文档](authentication.md)。

## Failed login limit

Authenticating with invalid credentials will return `401 Unauthorized`:

	curl -i https://openapi.zaoshu.io/v2 -u foo:bar
	HTTP/1.1 401 Unauthorized

	{
        "message": "Bad credentials"
	}

After detecting several requests with invalid credentials within a short period, the API will temporarily reject all authentication attempts for that user (including ones with valid credentials) with `403 Forbidden`:

	curl -i https://openapi.zaoshu.io/v2 -u valid_key:valid_secret
	HTTP/1.1 403 Forbidden

	{
        "message": "Maximum number of login attempts exceeded. Please try again later."
	}


## 分页

返回多个项目的请求将会做分页处理，默认每页 30 个项目，你可以通过指定 `?page` 参数来获取更多的分页内容。部分 API 中你可以通过指定 `?per_page` 参数来增加单页的项目数量，最大值为 100。需要特别说明的是，由于实现原因，部分接口不支持 `?per_page` 参数。

    curl 'https://openapi.zaoshu.io/v2/instances?page=2&per_page=100'

注意，分页数从 1 开始计算，没有 `?page` 参数时默认返回第一页内容。

## 频率限制

对于授权成功的请求，你每小时最多可以发起 5000 次。你可以通过任意的 API 请求的返回头部获取到当前的频率限制详情： 

curl -i https://openapi.zaoshu.io/v2/users/whatever
HTTP/1.1 200 OK
Date: Mon, 01 Jul 2017 17:27:06 GMT
Status: 200 OK
X-RateLimit-Limit: 6000
X-RateLimit-Remaining: 4500
X-RateLimit-Reset: 137270087

返回头中的以下字段包含了关于频率限制的所有信息：

| 头名称 | 描述 |
| --- | --- |
| `X-RateLimit-Limit` | 使用者每小时被允许的最大请求量 |
| `X-RateLimit-Remaining` | 当前的频率限制窗口时间内剩余可用的请求量 |
| `X-RateLimit-Reset` | 当前频率限制窗口重设的剩余时间，格式为[UNIX 时间](http://en.wikipedia.org/wiki/Unix_time) |

Once you go over the rate limit you will receive an error response:
一旦使用超过频率限制，你回收到错误返回：

	HTTP/1.1 403 Forbidden
	Date: Tue, 20 Aug 2017 14:50:41 GMT
	Status: 403 Forbidden
	X-RateLimit-Limit: 6000
	X-RateLimit-Remaining: 0
	X-RateLimit-Reset: 1377013266

	{
                "message": "API rate limit exceeded for xxx.xxx.xxx.xxx."
	}


<!--You can also [check your rate limit status](/v2/rate_limit) without incurring an API hit.-->


<!--## Abuse Rate Limits

To protect the quality of service from zaoshu, additional rate limits may apply to some actions. For example, rapidly creating content, polling aggressively instead of using webhooks, making API calls with a high concurrency, or repeatedly requesting data that is computationally expensive may result in abuse rate limiting.

It is not intended for this rate limit to interfere with any legitimate use of the API. Your normal [rate limits](#rate-limiting) should be the only limit you target.

If your application triggers this rate limit, you'll receive an informative response:

	HTTP/1.1 403 Forbidden
	Content-Type: application/json; charset=utf-8
	Connection: close

	{
        "message": "You have triggered an abuse detection mechanism and have been temporarily blocked from content creation. Please retry your request again later."
	}-->


<!--## User Agent Required

All API requests MUST include a valid `User-Agent` header. Requests with no `User-Agent` header will be rejected. We request that you use your zaoshu username, for the `User-Agent` header value. This allows us to contact you if there are problems.

Here's an example:

    User-Agent:


If you provide an invalid `User-Agent` header, you will receive a `403 Forbidden` response:

	curl -iH 'User-Agent: ' https://openapi.zaoshu.io/v2
	HTTP/1.0 403 Forbidden
	Connection: close
	Content-Type: text/html

	Request forbidden by administrative rules.
	Please make sure your request has a User-Agent header.-->