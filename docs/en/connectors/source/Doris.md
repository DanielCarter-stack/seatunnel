import ChangeLog from '../changelog/connector-doris.md';

# Doris

> Doris source connector

## Support Those Engines

> Spark<br/>
> Flink<br/>
> SeaTunnel Zeta<br/>

## Key features

- [x] [batch](../../introduction/concepts/connector-v2-features.md)
- [ ] [stream](../../introduction/concepts/connector-v2-features.md)
- [ ] [exactly-once](../../introduction/concepts/connector-v2-features.md)
- [x] [column projection](../../introduction/concepts/connector-v2-features.md)
- [x] [parallelism](../../introduction/concepts/connector-v2-features.md)
- [x] [support user-defined split](../../introduction/concepts/connector-v2-features.md)
- [x] [support multiple table read](../../introduction/concepts/connector-v2-features.md)

## Description

Used to read data from Apache Doris. Doris source is batch-only — for CDC-style ingestion, use a CDC source (e.g. MySQL-CDC) upstream of a Doris Sink.

## Using Dependency

### For Spark/Flink Engine

> 1. You need to ensure that the [jdbc driver jar package](https://mvnrepository.com/artifact/mysql/mysql-connector-java) has been placed in directory `${SEATUNNEL_HOME}/plugins/`.

### For SeaTunnel Zeta Engine

> 1. You need to ensure that the [jdbc driver jar package](https://mvnrepository.com/artifact/mysql/mysql-connector-java) has been placed in directory `${SEATUNNEL_HOME}/lib/`.

## Supported DataSource Info

| Datasource |          Supported versions          | Driver | Url | Maven |
|------------|--------------------------------------|--------|-----|-------|
| Doris      | Only Doris2.0 or later is supported. | -      | -   | -     |

## Data Type Mapping

|           Doris Data type            |                                                                 SeaTunnel Data type                                                                 |
|--------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| INT                                  | INT                                                                                                                                                 |
| TINYINT                              | TINYINT                                                                                                                                             |
| SMALLINT                             | SMALLINT                                                                                                                                            |
| BIGINT                               | BIGINT                                                                                                                                              |
| LARGEINT                             | STRING                                                                                                                                              |
| BOOLEAN                              | BOOLEAN                                                                                                                                             |
| DECIMAL                              | DECIMAL((Get the designated column's specified column size)+1,<br/>(Gets the designated column's number of digits to right of the decimal point.))) |
| FLOAT                                | FLOAT                                                                                                                                               |
| DOUBLE                               | DOUBLE                                                                                                                                              |
| CHAR<br/>VARCHAR<br/>STRING<br/>TEXT | STRING                                                                                                                                              |
| JSON                                 | STRING                                                                                                                                              |
| VARIANT                              | STRING                                                                                                                                              |
| DATE                                 | DATE                                                                                                                                                |
| DATETIME<br/>DATETIME(p)             | TIMESTAMP                                                                                                                                           |
| ARRAY                                | ARRAY                                                                                                                                               |
| MAP                                  | MAP                                                                                                                                                 |

## Source Options

Base configuration:

|               Name               |  Type  | Required |  Default   |                                             Description                                             |
|----------------------------------|--------|----------|------------|-----------------------------------------------------------------------------------------------------|
| fenodes                          | string | yes      | -          | FE address, the format is `"fe_host:fe_http_port"`                                                  |
| username                         | string | yes      | -          | User username                                                                                       |
| password                         | string | yes      | -          | User password                                                                                       |
| database                         | string | Conditional | -       | The name of Doris database. Required when `table_list` is not configured.                          |
| table                            | string | Conditional | -       | The name of Doris table. Required when `table_list` is not configured.                             |
| doris.batch.size                 | int    | no       | 1024       | Maximum rows that can be obtained by reading Doris BE once.                                         |
| case_sensitive                   | boolean | no      | true       | Whether to preserve the original case of table and column names. When false, names are lowercased. |
| doris.request.retries            | int    | no       | 3          | Number of retries to send requests to Doris FE.                                                     |
| doris.request.read.timeout.ms    | int    | no       | 30000      | Socket read timeout for requests sent to Doris BE.                                                  |
| doris.request.connect.timeout.ms | int    | no       | 30000      | Connection timeout for requests sent to Doris FE or BE.                                             |
| query-port                       | int    | no       | 9030       | Doris query port.                                                                                   |
| doris.request.query.timeout.s    | int    | no       | 3600       | Timeout period of Doris scan data, expressed in seconds.                                            |
| doris.request.tablet.size        | int    | no       | Integer.MAX_VALUE | The number of Doris tablets grouped into each SeaTunnel split. The minimum value is `1`.       |
| doris.deserialize.arrow.async    | boolean | no      | false      | Whether to deserialize Arrow data asynchronously.                                                    |
| doris.request.retriesdoris.deserialize.queue.size | int | no | 64 | Queue size used by asynchronous Arrow deserialization.                                               |
| table_list                       | Array  | no       | -          | List of Doris tables to read. Each entry supports `database`, `table`, `doris.read.field`, `doris.filter.query`, `doris.request.tablet.size`, `doris.batch.size`, `doris.exec.mem.limit`.                                                                        |

> Note: The option key `doris.request.retriesdoris.deserialize.queue.size` is the literal runtime key (the connector preserves the original name for backward compatibility). Copy it exactly when tuning the asynchronous Arrow deserialization queue.

Table list configuration:

|               Name               |  Type  | Required |  Default   |                                             Description                                             |
|----------------------------------|--------|----------|------------|-----------------------------------------------------------------------------------------------------|
| database                         | string | yes (per entry) | -    | The name of Doris database                                                                          |
| table                            | string | yes (per entry) | -    | The name of Doris table                                                                             |
| doris.read.field                 | string | no       | -          | Use the 'doris.read.field' parameter to select the doris table columns to read                      |
| doris.filter.query               | string | no       | -          | Data filtering in doris. the format is "field = value",example : doris.filter.query = "F_ID > 2"    |
| doris.request.tablet.size        | int    | no       | Integer.MAX_VALUE | The number of Doris tablets grouped into each SeaTunnel split for this table. The minimum value is `1`. |
| doris.batch.size                 | int    | no       | 1024       | The maximum value that can be obtained by reading Doris BE once.                                    |
| doris.exec.mem.limit             | long   | no       | 2147483648 | Maximum memory that can be used by a single be scan request. The default memory is 2G (2147483648). |

Note: When this configuration corresponds to a single table, you can flatten the configuration items in table_list to the outer layer. If `table_list` is not configured, `database` and `table` must be configured at the outer source level.

### Tips

> It is not recommended to modify advanced parameters at will

## Example

### single table
> This is an example of reading a Doris table and writing to Console.

```
env {
  parallelism = 2
  job.mode = "BATCH"
}
source{
  Doris {
      fenodes = "doris_e2e:8030"
      username = root
      password = ""
      database = "e2e_source"
      table = "doris_e2e_table"
  }
}

transform {
    # If you would like to get more information about how to configure seatunnel and see full list of transform plugins,
    # please go to https://seatunnel.apache.org/docs/transforms/sql
}

sink {
    Console {}
}
```

Use the 'doris.read.field' parameter to select the doris table columns to read

```
env {
  parallelism = 2
  job.mode = "BATCH"
}
source{
  Doris {
      fenodes = "doris_e2e:8030"
      username = root
      password = ""
      database = "e2e_source"
      table = "doris_e2e_table"
      doris.read.field = "F_ID,F_INT,F_BIGINT,F_TINYINT,F_SMALLINT"
  }
}

transform {
    # If you would like to get more information about how to configure seatunnel and see full list of transform plugins,
    # please go to https://seatunnel.apache.org/docs/transforms/sql
}

sink {
    Console {}
}
```

Use 'doris.filter.query' to filter the data, and the parameter values are passed directly to doris

```
env {
  parallelism = 2
  job.mode = "BATCH"
}
source{
  Doris {
      fenodes = "doris_e2e:8030"
      username = root
      password = ""
      database = "e2e_source"
      table = "doris_e2e_table"
      doris.filter.query = "F_ID > 2"
  }
}

transform {
    # If you would like to get more information about how to configure seatunnel and see full list of transform plugins,
    # please go to https://seatunnel.apache.org/docs/transforms/sql
}

sink {
    Console {}
}
```
### Multiple table
```hocon
env {
  parallelism = 1
  job.mode = "BATCH"
}

source {
  Doris {
    fenodes = "doris_e2e:8030"
    username = root
    password = ""
    table_list = [
      {
        database = "e2e_source_0"
        table = "doris_e2e_unique_table_0"
        doris.read.field = "F_ID,F_INT,F_BIGINT,F_TINYINT,F_SMALLINT,F_DECIMAL,F_LARGEINT,F_BOOLEAN,F_DOUBLE,F_FLOAT,F_CHAR,F_VARCHAR_11,F_STRING,F_DATETIME_P,F_DATETIME,F_DATE,MAP_VARCHAR_BOOLEAN,MAP_CHAR_TINYINT"
        doris.filter.query = "F_ID >= 50"
      },
      {
        database = "e2e_source_1"
        table = "doris_e2e_unique_table_1"
        doris.read.field = "F_ID,F_INT,F_BIGINT,F_TINYINT,F_SMALLINT,F_DECIMAL,F_LARGEINT,F_BOOLEAN,F_DOUBLE,F_FLOAT,F_CHAR,F_VARCHAR_11,F_STRING,F_DATETIME_P,F_DATETIME,F_DATE"
        doris.filter.query = "F_ID < 40"
      }
    ]
  }
}

transform {}

sink {
  Doris {
    fenodes = "doris_e2e:8030"
    schema_save_mode = "RECREATE_SCHEMA"
    username = root
    password = ""
    database = "e2e_sink"
    table = "${table_name}"
    sink.enable-2pc = "true"
    sink.label-prefix = "test_json"
    doris.config = {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

### Multiple table — single source + filtered per entry

Use `table_list` with `doris.read.field` and `doris.filter.query` on each entry to read only the columns you need from each Doris table and to apply a row-level predicate.

```hocon
env {
  parallelism = 1
  job.mode = "BATCH"
}

source {
  Doris {
    fenodes = "doris_e2e:8030"
    username = root
    password = ""
    table_list = [
      {
        database = "e2e_source_0"
        table = "doris_e2e_unique_table_0"
        doris.read.field = "F_ID,F_INT,F_BIGINT,F_TINYINT"
        doris.filter.query = "F_ID >= 50"
        doris.request.tablet.size = 1
        doris.exec.mem.limit = 2147483648
      },
      {
        database = "e2e_source_1"
        table = "doris_e2e_unique_table_1"
      }
    ]
  }
}

transform {}

sink {
  Console {}
}
```

## Changelog

<ChangeLog />
