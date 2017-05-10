# Instances

* [List your instances](#list-your-instances)
* [Get Instance Detail](#get-instance-detail)
* [Edit Instance](#edit-instance)
* [Run Instance](#run-instance)
* [List the Tasks of Instance](#list-the-tasks-of-instance)
* [Get Task Detail](#get-task-detail)

## List your instances

List instances created by authenticated user.

    GET /instances

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `status` | `string` | `false` | Can be one of: `all`, `running` or `idle` | `all` |

### Response

    Status: 200 OK

    {
        
    }

## Get Instance Detail 

Get detail of single instance.

    GET /instance/:instance_id

### Response

    Status: 200 OK

    {

    }

## Edit Instance

Edit meta-data and configuration of instance.

    PATCH /instance/:instance_id

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `title` | `string` | `false` | The title of instance | `null` |
| `result_notify_uri` | `string` | `false` | The result notification webhook uri. | `null` |

### Response

    Status: 200 OK

    {

    }

## Run Instance 

Run instance with optional configurations.

    POST /instance/:instance_id

### Parameters

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `target_sources` | `object array` | `false` | Crawl & Parse target config. | `[{"urls": [${DEFAULT_URL}]}]` |
| `result_notify_uri` | `string` | `false` | The result notification webhook uri. | `null` |

#### Example

To run a instance, you can set up the crawl & parse target by `target_source` whose object contains the following fields:

| Name | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `urls` | `array` | `false` |  | `[${DEFAULT_URL}]` |
| `method` | `string` | `false` | HTTP method | `GET` |
| `headers` | `object` | `false` | Key/value pairs for `HTTP Request Headers` | `null` |

Default URL rule you set in [Dashboard](https://dashboard.zaoshu.io) will be used when field `urls` found omitted.

### Response

    Status: 200 OK

## List the Tasks of Instance 

List all tasks of specific instance.

    GET /instance/:instance_id/tasks

### Response

    Status: 200 OK

    {

    }

## Get Task Detail 

Get detial of single task.

    GET /instance/:instance_id/task/:task_id

### Response

    Status: 200 OK

    {

    }