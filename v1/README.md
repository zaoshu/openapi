# 造数智能云爬虫 开放应用程序接口

[English Version](./README_en-US.md)

*注意: 当前版本为提供给早期开发者用户的演示版本，请勿用于生产环境。本文档内容也可能会很快失效，请您及时根据文档进行必要的更新。*

*如果您想对公开的网做要抓取和解析，但并没有网页爬虫编程的相关技能与知识，我们推荐您使用网页云版本的[`造数智能云爬虫`](https://zaoshu.io)。*

## V1 (alpha)

### 接口入口: 

    协议: https 
    域名: openapi.zaoshu.io
    入口路径: /v1

### 接入密钥

在 URL 中构造 `/v1/u_${ACCESS_KEY}/` 来进行接口身份认证，例如：

    https://openapi.zaoshu.io/v1/u_9083adfb6d50484b8c2ee3c504012def

你可以在[`个人设置`](https://dashboard.zaoshu.io/?settings)的`开发者功能`中获取接入密钥。

### 有且仅有的一个 API 

#### 获取指定实例最近一次执行结果 

    entry: /v1/u_${ACCESS_KEY}/instance_${INSTANCE_ID}/downloadLatest/${CONTENT_TYPE}/?since=${SINCE_WHEN}

* ${INSTANCE_ID}: 
    * 描述: 实例 ID，请在`我的爬虫`中，具体实例设置下方的`开发者功能`中获取
    * 可选: False
    * 类型: String
* ${CONTENT_TYPE}: 
    * 描述: 取回的结果内容文件类型
    * 可选: False
    * 选项: csv/xls/json/xml
    * 类型: String
* ${SINCE_WHEN}:
    * 描述: 指定的秒级时间戳时间后，距今最近的一次成功执行任务的结果
    * 可选: True
    * 类型: Int64 

例如:

    https://openapi.zaoshu.io/v1/ \
        u_9083adfb6d50484b8c2ee3c504012def/ \ 
        instance_77c35ed148134a67b091deb258efeade/ \
        downloadLatest/json?since=1489145879

*注意：如果你的爬虫实例在运行时打开了`深度爬取`功能，该次运行结果将会被压缩打包，API 取回的结果将是包含你选择的类型文件的`zip`文件。*
