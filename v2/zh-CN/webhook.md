#Webhooks


Webhook允许你在zaoshu.io上订阅某些实例，当触发事件时，我们将向Webhook配置的URL发送HTTP POST有效的内容，每个爬虫实例下可配置一个webhook一旦配置成功，每当订阅的事件发生在该爬虫实例上，它们将会被触发，目前每个爬虫实例只能创建一个webhook。(目前暂不支持通配符)

下面是发送webhook例子:

```
POST /payload HTTP/1.1
Host: localhost:12138
User-Agent: zaoshu-Hookshot
Content-Type: application/json
Content-Length: 6877
X-zaoshu-Event: default

{
  "taskId": "de9d0c7496c247e2ba27f34cafeb052c",
  "title": "抓数据去造数！！！",
  "UUID" : "aaaaf632-0d4b-42bc-b71f-669ee0d6f9aa",
  "createdAt" : 690177600,
}

```

  