# 授权验证

所有发送到造数 OpenAPI 的请求都需要签名，并设置请求头的`Authorization`。签名需要的`key`和`secret`可以从[这里](https://dashboard.zaoshu.io/?settings)获取。

## 获取需要签名的数据

伪代码如下:

```
StringToSign =
    HTTP_METHOD + "\n" +
    Content-Type + "\n" +
    Date + "\n" +
    SortedQueryString + "\n" +
    Body
```

> 如果没有查询参数就设置为空字符串

> 如果没有`Body`就设置为空字符串

#### SortedQueryString
步骤如下:

1. 按照码位（code point）升序排列参数名称。例如名字以F开头的参数要排在以b开头的参数前面。

2. 对于每个参数都要按照`name=value`的格式拼接起来，对于没有值的参数应该设置为空字符串。

3. 除了最后一个参数其他的都要加上换行符（\n）。

#### 请求示例
下面的例子展示了如何构造一个签名字符串：

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

对应的`Python`代码是:
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

## 计算签名
```
signature = base64(hmac-sha256(api secret, string to sign))
```

## 添加签名到`Authorization`

```
Authorization: ZAOSHU api-key:signature
```

## 示例

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

## 示例代码

#### Python 3

```python
import hmac
import hashlib
import base64

def sign(secret, method, content_type, date, query=None, body=None):
    values = [
        method,
        content_type,
        date,
    ]
    if query:
        values.extend("%s=%s" % (k, query[k]) for k in sorted(query.keys()))
    else:
        values.append("")

    if body:
        values.append(body)
    else:
        values.append("")
    base_string = "\n".join(values)
    print(base_string)

    digest = hmac.new(secret.encode("utf8"), base_string.encode("utf8"), hashlib.sha256).digest()
    return base64.b64encode(digest).decode("utf8")

if __name__ == '__main__':
    query = {
        'a': '1',
        'b': '2',
    }
    body = '{"v": "tt"}'
    s = sign(
        '1234567890-=',
        'POST',
        'application/json; charset=utf-8',
        'Wed, 18 Mar 2016 08:04:06 GMT',
        query, body
    )
    print(s)
```

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
    request.headers['content-type'],
    date,
    getSortedQueryString(request.url),
];

if (typeof request.data === 'string') {
    values.push(request.data || '');
} else {
    values.push('');
}

var base_string = values.join('\n');
console.log(base_string);

var signature = CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(base_string, environment['api-secret']));
console.log(signature);

postman.setEnvironmentVariable('date', date);
postman.setEnvironmentVariable('signature', signature);
```
