# Authentication

When you send HTTP requests to Zaoshu, you sign the requests and add to HTTP `Authorization` header so that Zaoshu can identify who sent them. You sign requests with your API key and secret, which can be found from [Dashboard](https://dashboard.zaoshu.io/?settings).

To sign a request, you use body of the request with some other values from the request and your API secret to create a signed hash, this is the signature.

## Create the string to sign

To create the string to sign, concatenate the method, content type, date, sorted query string and body of the request, as shown in the following pseudocode:

```
StringToSign =
    HTTP_METHOD + "\n" +
    Content-Type + "\n" +
    Date + "\n" +
    SortedQueryString + "\n" +
    Body
```

> Set it to empty string if there is no `SortedQueryString` or `Body`

#### Sort query string
To construct the sorted query string, complete the following steps:

1. Sort the parameter names by character code point in ascending order. For example, a parameter name that begins with the uppercase letter F precedes a parameter name that begins with a lowercase letter b.

2. For each parameter, append the parameter name, followed by the equals sign character (=), followed by the parameter value. Use an empty string for parameters that have no value.

3. Append the newline character (\n) after each parameter value, except for the last value in the list.

#### Example request
The following example shows how to construct the string to sign.

```
GET /test?a=1&b=2&Q= HTTP/1.1
Host: openapi.zaoshu.io
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Mar 2016 08:04:06 GMT
```

1. HTTP_METHOD is `"GET"`
2. Content-Type is `"application/json; charset=utf-8"`
3. Date is `"Wed, 18 Mar 2016 08:04:06 GMT"`
4. SortedQueryString is `"Q=\na=1\nb=2"`
5. Body is empty string

String to sign is:
```
"GET" + "\n" +
"application/json; charset=utf-8" + "\n" +
"Wed, 18 Mar 2016 08:04:06 GMT" + "\n" +
"Q=\na=1\nb=2" + "\n" +
""
```

And python example code is:
```python
query = {
    "a": "1",
    "b": "2",
    "Q": "",
}

values = [
    "GET",
    "application/json; charset=utf-8",
    "Wed, 18 Mar 2016 08:04:06 GMT",
]
# sorted query string
if len(query) > 0:
    values.extend("%s=%s" % (k, query[k]) for k in sorted(query.keys()))
else:
    values.append("")
# empty body
values.append("")

# string to sing
"\n".join(values)

# result is "GET\napplication/json; charset=utf-8\nWed, 18 Mar 2016 08:04:06 GMT\nQ=\na=1\nb=2\n"
```

## Calculate the Signature
The following pseudocode shows how to calculate the signature.
```
signature = base64(hmac-sha256(api secret, string to sign))
```

## Add Signature to the Authorization Header

You can include signature by adding it to an HTTP header named `Authorization`. Although the header is named Authorization, the signing information is actually used for authentication.

The following pseudocode shows the construction of the Authorization header.
```
Authorization: ZAOSHU api-key:signature
```

## Example

Request:
```
POST /test?a=1&b=2 HTTP/1.1
Host: openapi.zaoshu.io
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Mar 2016 08:04:06 GMT

{"v": "tt"}
```

API Key:
```
qwertyuiop
```

API Secret:
```
1234567890-=
```

String to sign:
```
POST\napplication/json; charset=utf-8\nWed, 18 Mar 2016 08:04:06 GMT\na=1\nb=2\n{"v": "tt"}

```

Signature:
```
EZlFQV45vYb+vGEqmBs2N0u2kWkOWzZujIF28wAXi0I=
```

Result request:
```
POST /test?a=1&b=2 HTTP/1.1
Host: openapi.zaoshu.io
Content-Type: application/json; charset=utf-8
Date: Wed, 18 Mar 2016 08:04:06 GMT
Authorization: ZAOSHU qwertyuiop:EZlFQV45vYb+vGEqmBs2N0u2kWkOWzZujIF28wAXi0I=

{"v": "tt"}
```

## Example Code

#### JS code for postman

```js
function getQueryParams(url){
    var qparams = {},
        parts = (url||'').split('?'),
        qparts, qpart,
        i=0;

    if(parts.length <= 1){
        return qparams;
    }else{
        qparts = parts[1].split('&');
        for(i in qparts){
            qpart = qparts[i].split('=');
            qparams[qpart[0]] = qpart[1] || '';
        }
    }
    return qparams;
}

function KeyValue(key, value) {
  this.key = key;
  this.value = value;
}

KeyValue.prototype = {
  toString: function() {
    return this.key + '=' + this.value;
  }
};

function getSortedQueryString(url) {
    var values = getQueryParams(url);

    var query = [];
    for (var key in values) {
        if (values.hasOwnProperty(key)) {
            query.push(new KeyValue(key, values[key]));
        }
    }

    query.sort(function(a, b){ return a.key < b.key ? -1 : 1 });
    return query.join('\n');
}

var date = (new Date()).toUTCString();
var values = [
    request.method,
    request.headers['Content-Type'],
    date,
    getSortedQueryString(request.url),
];

if (typeof request.data === 'string') {
    values.push(request.data || '');
}

var base_string = values.join('\n');
console.log(base_string);

var signature = CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(base_string, environment['api-secret']));
console.log(signature);

postman.setEnvironmentVariable('date', date);
postman.setEnvironmentVariable('signature', signature);
```