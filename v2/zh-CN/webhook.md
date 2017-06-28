# Webhook

Webhook 允许你在 [zaoshu.io](https://zaoshu.io) 上订阅实例的运行状态。当触发事件时，我们将向webhook 配置的 URL 发送 HTTP POST 有效的内容。

Webhook 允许你在 zaoshu.io 上订阅某些实例，当触发事件时，我们将向 Webhook 配置的 URL 发送 HTTP POST 有效的内容，每个爬虫实例下可配置一个 webhook 一旦配置成功，每当订阅的事件发生在该爬虫实例上，它们将会被触发，目前每个爬虫实例只能创建一个 webhook。(目前暂不支持通配符)

下面是发送 webhook 例子:


    POST /payload HTTP/1.1
    Host: localhost:12138
    user-agent: zaoshu-hookshot
    Content-Type: application/json
    Content-Length: 6877
    x-zaoshu-event: default                           //通知事件，目前有且只有一个
    
    {
      "taskId": "de9d0c7496c247e2ba27f34cafeb052c",                //任务ID
      "title": "抓数据去造数！！！",                                 //实例标题
      "appInstanceId" : "aaaaf632-0d4b-42bc-b71f-669ee0d6f9aa",    //实例ID
      "createdAt" : 690177600,                                     //创建该实例时间戳
    }


## 如何使用WebHook

成功设置 WebHook 的开发者，实例完成时，造数会发送 JSON 数据格式，并且造数只会发送一次消息给 WebHook (目前 WebHook 没有做消息可靠性到达，如果目标服务返回的 http code 不为 200 即表示发送失败)。


## 注意事项

1、在配置 webhook 的 URL，造数并不会发送 HTTP 探针去验证服务的有效性，这里需要开发者，谨慎保管好自己的 WebHook。   
2、目前造数仅支持任务执行完成后给 WebHook 发送数据，```x-zaoshu-event: default``` 默认仅仅发送任务已完成数据   
3、造数目前没有针对每个 WebHook 设置 secret，所以开发者需要保护好自己的WebHook不让别人知道，在可预见的将来，将会完善此功能。
  