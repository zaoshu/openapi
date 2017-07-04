# 爬虫实例

* [实例列表](#实例列表)
* [实例详情](#实例详情)
* [实例数据格式](#实例数据格式)
* [编辑实例](#编辑实例)
* [执行实例](#执行实例)
* [实例任务列表](#实例任务列表)
* [实例任务详情](#实例任务详情)
* [实例任务数据下载](#实例任务数据下载)

## 实例列表 

获取登录用户的爬虫实例列表。

    GET /instances

### 参数

| 名称     | 类型     | 必要参数 | 描述 | 缺省值 |
| -------- | -------- | -------- | ---------------------------------------- | ------- |
| `status` | `string` | `false`  | 可选状态值：`all`, `running` 或 `idle` | `all`   |

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": [
        {
          "id": "instance id",
          "title": "instance title",
          "status": "idle/error/running",
          "createdAt": 1492593196000,
          "totalExecution": 1,
          "totalFetchingRows": 1
        }
      ],
      "msg": ""
    }

## 实例详情

获取单个实例的详情。

    GET /instance/:instance_id

### 返回 

    Status: 200 OK

    {
      "code": 0,
      "data": {
        "id": "instance id",
        "title": "instance title",
        "status": "idle/error/running",
        "createdAt": 1494844957000,
        "totalExecution": 1,
        "totalFetchingRows": 1
      },
      "msg": ""
    }


## 实例数据格式

获取单个实例数据格式。

    GET /instance/:instance_id/schema

### 返回 

    Status: 200 OK

    {
        "code": 0,
        "data": {
            "schemas": [
                {
                    "title": "姓名",
                    "id": "C339",
                    "type": "string"
                },
                {
                    "title": "班级",
                    "id": "C337",
                    "type": "string"
                },
                {
                    "title": "学号",
                    "id": "C341",
                    "type": "string"
                },
            ]
        },
        "msg": ""
    }

## 编辑实例 

编辑实例的元数据与相关配置。

    PATCH /instance/:instance_id

### 参数 

| 名称               | 类型     | 必要参数 | 描述                          | 缺省值 |
| ------------------- | -------- | -------- | ------------------------------------ | ------- |
| `title`             | `string` | `false`  | 爬虫实例标题。 | `null`  |
| `result_notify_uri` | `string`       | `false`  | 爬取结果通知回调地址 | `null`                      |

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": null,
      "msg": ""
    }

## 执行实例 

执行实例，支持可选配置。

    POST /instance/:instance_id

### 参数

| 名称               | 类型           | 必要参数 | 描述               | 缺省值                         |
| ------------------- | -------------- | -------- | -----------------| ----------------------------- |
| `target_sources`    | `object array` | `false`  | 爬取&解析目标配置 | `[{"urls": [${DEFAULT_URL}]}]` |
| `result_notify_uri` | `string`       | `false`  | 爬取结果通知回调地址 | `null`                      |

#### 样例

执行实例时，你可以通过 `target_source` 来设置爬取&解析目标，其中包含了以下字段：

| 名称     | 类型     | 必要参数 | 描述                | 缺省值            |
| --------- | -------- | -------- | ----------------- | ------------------ |
| `urls`    | `array`  | `false`  | 目标 URL 数组 | `[${DEFAULT_URL}]` |
| `method`  | `string` | `false`  | HTTP 方法 | `GET` |
| `headers` | `object` | `false`  | HTTP 请求头键值对 | `null` |

参数 `urls` 为空时，默认将会使用您在[控制面板->我的爬虫](https://dashboard.zaoshu.io)中设置的网址规则。

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": null,
      "msg": ""
    }

## 实例任务列表 

获取指定爬虫实例下的任务列表。

    GET /instance/:instance_id/tasks

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": [
        {
          "id": "task id",
          "status": "done/error",
          "createdAt": 1494810771000
        }
      ],
      "msg": ""
    }

## 实例任务详情 

获取单个任务详情。

    GET /instance/:instance_id/task/:task_id

### 返回

    Status: 200 OK

    {
      "code": 0,
      "data": {
        "id": "task id",
        "status": "done/error",
        "createdAt": 1494810771000,
        "totalUrlCount": 1,
        "failedUrlCount": 0
      },
      "msg": ""
    }


## 实例任务数据下载

下载单个实例的数据文件

    GET /instance/:instance_id/task/:task_id/result/file

### 参数
  
|名称|类型|必要参数|描述|默认值|
| ------------- | -------- | ------ | ------------------------------------- | - |
| `contentType` | `string` | `true` | 可选值：`csv`、`xls`、`json` 或 `xml` | - |

举例:
	
    curl http://openapi.zaoshu.io/v2/instance/d3c903c9e11d4f45bec82a6640f06c59/1e229da9e5454b66a713bddba7b3e012?contentType=json

### 返回

  Status: 200 OK   
  Content-Type →application/octet-stream
