import ChangeLog from '../changelog/connector-http.md';

# Http

> Http 数据接收器

## 支持引擎

> Spark<br/>
> Flink<br/>
> SeaTunnel Zeta<br/>

## 主要特性

- [ ] [精确一次](../../introduction/concepts/connector-v2-features.md)
- [ ] [变更数据捕获（CDC）](../../introduction/concepts/connector-v2-features.md)
- [x] [支持多表写入](../../introduction/concepts/connector-v2-features.md)

## 描述

接收Source端传入的数据，利用数据触发 web hooks。

> 例如，来自上游的数据为[`age: 12, name: tyrantlucifer`]，则body内容如下：`{"age": 12, "name": "tyrantlucifer"}`

**Tips: Http 接收器仅支持 `post json` 类型的 web hook，source 数据将被视为 webhook 中的 body 内容。**

## 支持的数据源信息

想使用 Http 连接器，需要安装以下必要的依赖。可以通过运行 install-plugin.sh 脚本或者从 Maven 中央仓库下载这些依赖

| 数据源  | 支持版本 | 依赖                                                                           |
|------|------|------------------------------------------------------------------------------|
| Http | 通用   | [下载](https://mvnrepository.com/artifact/org.apache.seatunnel/connector-http) |

## 接收器选项

|             名称              |   类型   | 是否必须 |  默认值  |                             描述                             |
|-----------------------------|--------|------|-------|------------------------------------------------------------|
| url                         | String | 是    | -     | Http 请求链接                                                  |
| headers                     | Map    | 否    | -     | Http 标头                                                    |
| params                      | Map    | 否    | -     | 追加到请求 URL 上的查询参数。                                      |
| retry                       | Int    | 否    | -     | 如果请求http返回`IOException`的最大重试次数                             |
| retry_backoff_multiplier_ms | Int    | 否    | 100   | http请求失败，重试回退次数（毫秒）乘数                                      |
| retry_backoff_max_ms        | Int    | 否    | 10000 | http请求失败，最大重试回退时间(毫秒)                                      |
| array_mode                  | Boolean| 否    | false | 为true时将数据作为JSON数组发送，为false时作为单个JSON对象发送（默认）                |
| batch_size                  | Int    | 否    | 1     | 在一个HTTP请求中发送的记录批量大小。仅在array_mode为true时有效                   |
| request_interval_ms         | Int    | 否    | 0     | 两次HTTP请求之间的间隔毫秒数，以避免请求过于频繁                                 |
| multi_table_sink_replica    | Int    | 否    | 1     | 多表写入时，每个表对应写入器的副本数量。                                  |
| common-options              |        | 否    | -     | Sink插件常用参数，请参考 [Sink常用选项 ](../common-options/sink-common-options.md) 了解详情 |

## 示例

简单示例:

```hocon
Http {
    url = "http://localhost/test/webhook"
    headers {
        token = "9e32e859ef044462a257e1fc76730066"
    }
}
```

### 带批处理的示例

```hocon
Http {
    url = "http://localhost/test/webhook"
    headers {
        token = "9e32e859ef044462a257e1fc76730066"
        Content-Type = "application/json"
    }
    array_mode = true
    batch_size = 50
    request_interval_ms = 500
}
```

### 多表写入

Http Sink 支持从多表来源写入。每条数据会按其所属表发送到同一个 HTTP 地址；如果需要提高多表写入并发，可以调整 `multi_table_sink_replica`。

```hocon
env {
  parallelism = 1
  job.mode = "BATCH"
}

source {
  FakeSource {
    tables_configs = [
      {
        schema = {
          table = "http_sink_1"
          fields {
            id = int
            name = string
          }
        }
        rows = [
          {
            kind = INSERT
            fields = [1, "Alice"]
          }
        ]
      },
      {
        schema = {
          table = "http_sink_2"
          fields {
            id = int
            name = string
          }
        }
        rows = [
          {
            kind = INSERT
            fields = [2, "Bob"]
          }
        ]
      }
    ]
  }
}

sink {
  Http {
    url = "http://localhost/test/webhook"
    headers {
      token = "token"
    }
    multi_table_sink_replica = 1
  }
}
```

## 变更日志

<ChangeLog />
