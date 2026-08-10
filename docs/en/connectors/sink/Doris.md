import ChangeLog from '../changelog/connector-doris.md';

# Doris

> Doris sink connector

## Support Doris Version

- exactly-once & cdc supported  `Doris version is >= 1.1.x`
- Array data type supported  `Doris version is >= 1.2.x`
- Map data type will be support in `Doris version is 2.x`

## Support Those Engines

> Spark<br/>
> Flink<br/>
> SeaTunnel Zeta<br/>

## Key Features

- [x] [exactly-once](../../introduction/concepts/connector-v2-features.md)
- [x] [cdc](../../introduction/concepts/connector-v2-features.md)
- [x] [support multiple table write](../../introduction/concepts/connector-v2-features.md)
- [x] [timer flush](../../introduction/concepts/connector-v2-features.md)

## Description

Used to send data to Doris. Both support streaming and batch mode.
The internal implementation of Doris sink connector is cached and imported by stream load in batches.

## Using Dependency

### For Spark/Flink Engine

> 1. You need to ensure that the [jdbc driver jar package](https://mvnrepository.com/artifact/mysql/mysql-connector-java) has been placed in directory `${SEATUNNEL_HOME}/plugins/`.

### For SeaTunnel Zeta Engine

> 1. You need to ensure that the [jdbc driver jar package](https://mvnrepository.com/artifact/mysql/mysql-connector-java) has been placed in directory `${SEATUNNEL_HOME}/lib/`.

## Sink Options

|              Name              |  Type   | Required |           Default            |                                                                                                                                      Description                                                                                                                                       |
|--------------------------------|---------|----------|------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| fenodes                        | String  | Yes      | -                            | `Doris` cluster fenodes address, the format is `"fe_ip:fe_http_port, ..."`                                                                                                                                                                                                             |
| benodes                        | String  | No       | -                            | `Doris` BE http address list used when `direct_to_be=true`, the format is `"be_ip:be_http_port, ..."`                                                                                                                                                                                   |
| direct_to_be                   | bool    | No       | false                        | Whether to send stream load write requests directly to `benodes`. This is an opt-in mode and does not change the default FE path.                                                                                                                                                       |
| query-port                     | int     | No       | 9030                         | `Doris` Fenodes query_port                                                                                                                                                                                                                                                             |
| username                       | String  | Yes      | -                            | `Doris` user username                                                                                                                                                                                                                                                                  |
| password                       | String  | Yes      | -                            | `Doris` user password                                                                                                                                                                                                                                                                  |
| database                       | String  | Conditional | -                         | The database name of `Doris` table, use `${database_name}` to represent the upstream table name. Required unless `table.identifier` is set.                                                                                                                                              |
| table                          | String  | Conditional | -                         | The table name of `Doris` table,  use `${table_name}` to represent the upstream table name. Required unless `table.identifier` is set.                                                                                                                                                  |
| table.identifier               | String  | No       | -                            | Deprecated table identifier. Please use `database` and `table` instead. Existing e2e configs and templates may still use `table.identifier`; both forms are supported.                                                                                                              |
| sink.label-prefix              | String  | Yes      | -                            | The label prefix used by stream load imports. In the 2pc scenario, global uniqueness is required to ensure the EOS semantics of SeaTunnel.                                                                                                                                             |
| sink.enable-2pc                | bool    | No       | false                        | Whether to enable two-phase commit (2pc), the default is false. For two-phase commit, please refer to [here](https://doris.apache.org/docs/data-operate/transaction?_highlight=two&_highlight=phase#stream-load-2pc).                                                              |
| sink.enable-delete             | bool    | No       | -                            | Whether to enable deletion. This option requires Doris table to enable batch delete function (0.15+ version is enabled by default), and only supports Unique model. you can get more detail at this [link](https://doris.apache.org/docs/dev/data-operate/delete/batch-delete-manual/) |
| sink.check-interval            | int     | No       | 10000                        | check exception with the interval while loading                                                                                                                                                                                                                      |
| sink.max-retries               | int     | No       | 3                            | the max retry times if writing records to database failed                                                                                                                                                                                                            |
| sink.buffer-size               | int     | No       | 256 * 1024                   | the buffer size to cache data for stream load.                                                                                                                                                                                                                       |
| sink.buffer-count              | int     | No       | 3                            | the buffer count to cache data for stream load.                                                                                                                                                                                                                      |
| doris.batch.size               | int     | No       | 1024                         | the batch size of the write to doris each http request, when the row reaches the size or checkpoint is executed, the data of cached will write to server.                                                                                                            |
| needs_unsupported_type_casting | boolean | No       | false                        | Whether to enable conversion of Doris types that are not natively supported (e.g. Decimal64, complex types) in the catalog before write. When true, the catalog is pre-processed via `UnsupportedTypeConverterUtils` so write-time casting stays consistent.                                                                                                                          |
| case_sensitive                 | boolean | No       | true                         | Whether to preserve the original case of table and column names. When set to false, table and column names will be converted to lowercase. This also affects runtime placeholder substitution (`${database_name}`, `${table_name}`).                                                                                                                            |
| schema_save_mode               | Enum    | Yes      | CREATE_SCHEMA_WHEN_NOT_EXIST | The schema save mode; please refer to `schema_save_mode` below. Required because Doris Sink auto-creates the target table when the mode demands it.                                                                                                                                                                                                       |
| data_save_mode                 | Enum    | Yes      | APPEND_DATA                  | The data save mode; please refer to `data_save_mode` below. Required because Doris Sink needs to know how to handle existing rows when the job starts.                                                                                                                                                                                                           |
| save_mode_create_template      | string  | No       | see below                    | Custom DDL template used to create the target Doris table. Supports placeholders such as `${database}`, `${table}`, `${rowtype_fields}`, `${rowtype_primary_key}`, `${rowtype_duplicate_key}`, `${comment}`.                                                                                                                                                                                                                                                            |
| custom_sql                     | String  | Conditional (`data_save_mode=CUSTOM_PROCESSING`) | -                            | When data_save_mode selects CUSTOM_PROCESSING, you should fill in the CUSTOM_SQL parameter. This parameter usually fills in a SQL that can be executed. SQL will be executed before synchronization tasks.                                                           |
| doris.config                   | map     | yes      | -                            | Stream Load import options such as `format`, `read_json_by_line`, `column_separator`. Supports `insert`, `delete`, and `update` operations when the connector generates SQL.                                                                                                                            |
| default-database               | String  | No       | information_schema           | Default database used when the sink needs to resolve unqualified identifiers.                                                                                                                                                                                          |
| multi_table_sink_replica       | int     | No       | 1                            | Replica count when the connector writes to multiple tables at once. See [Sink Common Options](../common-options/sink-common-options.md).                                                                                                                                                                                          |

## Redirect Behavior

By default, Doris sink sends Stream Load requests to the FE nodes configured by `fenodes`.

When `direct_to_be=true`, SeaTunnel uses `benodes` for the Stream Load data write path.

If `sink.enable-2pc=true` at the same time:

- pre-commit data write requests use `benodes`
- 2PC commit/abort control requests still use `fenodes`

This mixed path keeps the default FE control path while allowing the data path to bypass unstable FE redirect scenarios.

### schema_save_mode [Enum]

Before the synchronous task is turned on, different treatment schemes are selected for the existing surface structure of the target side.  
Option introduction：  
`RECREATE_SCHEMA` ：Will create when the table does not exist, delete and rebuild when the table is saved        
`CREATE_SCHEMA_WHEN_NOT_EXIST` ：Will Created when the table does not exist, skipped when the table is saved        
`ERROR_WHEN_SCHEMA_NOT_EXIST` ：Error will be reported when the table does not exist  
`IGNORE` ：Ignore the treatment of the table

### data_save_mode [Enum]

Before the synchronous task is turned on, different processing schemes are selected for data existing data on the target side.  
Option introduction：  
`DROP_DATA`： Preserve database structure and delete data  
`APPEND_DATA`：Preserve database structure, preserve data  
`CUSTOM_PROCESSING`：User defined processing  
`ERROR_WHEN_DATA_EXISTS`：When there is data, an error is reported

### save_mode_create_template

We use templates to automatically create Doris tables,
which will create corresponding table creation statements based on the type of upstream data and schema type,
and the default template can be modified according to the situation.

Default template:

```sql
CREATE TABLE IF NOT EXISTS `${database}`.`${table}` (
${rowtype_primary_key},
${rowtype_fields}
) ENGINE=OLAP
 UNIQUE KEY (${rowtype_primary_key})
COMMENT '${comment}'
DISTRIBUTED BY HASH (${rowtype_primary_key})
 PROPERTIES (
"replication_allocation" = "tag.location.default: 1",
"in_memory" = "false",
"storage_format" = "V2",
"disable_auto_compaction" = "false"
)
```

If a custom field is filled in the template, such as adding an `id` field

```sql
CREATE TABLE IF NOT EXISTS `${database}`.`${table}`
(   
    id,
    ${rowtype_fields}
) ENGINE = OLAP UNIQUE KEY (${rowtype_primary_key})
    COMMENT '${comment}'
    DISTRIBUTED BY HASH (${rowtype_primary_key})
    PROPERTIES
(
    "replication_num" = "1"
);
```

The connector will automatically obtain the corresponding type from the upstream to complete the filling,
and remove the id field from `rowtype_fields`. This method can be used to customize the modification of field types and attributes.

You can use the following placeholders. `${table}` is the canonical placeholder; `${table_name}` is kept as a deprecated alias for backward compatibility.

- database: Used to get the database in the upstream schema
- table_name: Used to get the table name in the upstream schema (deprecated alias of `${table}`)
- table: Used to get the table name in the upstream schema
- rowtype_fields: Used to get all the fields in the upstream schema, we will automatically map to the field
  description of Doris
- rowtype_primary_key: Used to get the primary key in the upstream schema (maybe a list)
- rowtype_unique_key: Used to get the unique key in the upstream schema (maybe a list)
- rowtype_duplicate_key: Used to get the duplicate key in the upstream schema (only for doris source, maybe a list)
- comment: Used to get the table comment in the upstream schema

#### Duplicate-key table template

When the target Doris table uses the Duplicate Key model, you can use `rowtype_duplicate_key` to generate the key clause:

```sql
CREATE TABLE IF NOT EXISTS `${database}`.`${table}` (
${rowtype_fields}
) ENGINE=OLAP
DUPLICATE KEY (${rowtype_duplicate_key})
DISTRIBUTED BY HASH (${rowtype_duplicate_key})
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
)
```

## Data Type Mapping

| Doris Data Type |           SeaTunnel Data Type           |
|-----------------|-----------------------------------------|
| BOOLEAN         | BOOLEAN                                 |
| TINYINT         | TINYINT                                 |
| SMALLINT        | SMALLINT<br/>TINYINT                    |
| INT             | INT<br/>SMALLINT<br/>TINYINT            |
| BIGINT          | BIGINT<br/>INT<br/>SMALLINT<br/>TINYINT |
| LARGEINT        | BIGINT<br/>INT<br/>SMALLINT<br/>TINYINT |
| FLOAT           | FLOAT                                   |
| DOUBLE          | DOUBLE<br/>FLOAT                        |
| DECIMAL         | DECIMAL<br/>DOUBLE<br/>FLOAT            |
| DATE            | DATE                                    |
| DATETIME        | TIMESTAMP                               |
| CHAR            | STRING                                  |
| VARCHAR         | STRING                                  |
| STRING          | STRING                                  |
| ARRAY           | ARRAY                                   |
| MAP             | MAP                                     |
| JSON            | STRING                                  |
| VARIANT         | STRING                                  |
| HLL             | Not supported yet                       |
| BITMAP          | Not supported yet                       |
| QUANTILE_STATE  | Not supported yet                       |
| STRUCT          | Not supported yet                       |

#### Supported import data formats

The supported formats include CSV and JSON

When writing to Doris `VARIANT` columns from SeaTunnel `STRING` fields, the field value should be a
valid JSON document.

## Tuning Guide
Appropriately increasing the value of `sink.buffer-size` and `doris.batch.size` can increase the write performance.

In stream mode, if the `doris.batch.size` and `checkpoint.interval` are both configured with a large value, The last data to arrive may have a large delay(The delay time is the checkpoint interval).

This is because the total amount of data arriving at the end may not exceed the threshold specified by `doris.batch.size`. Therefore, commit can only be triggered by checkpoint before the volume of received data does not exceed this threshold. Therefore, you should select an appropriate `checkpoint.interval`.

Otherwise, if you enable the 2pc by the property `sink.enable-2pc=true`.The `sink.buffer-size` will have no effect. So only the checkpoint can trigger the commit.

### Timer flush on Zeta

This engine-level feature is supported only by Zeta. Spark and Flink do not inject `FlushSignal` records. On Zeta, configure `sink.flush.interval` in the `env` block to finish the current Stream Load before `doris.batch.size` is reached.

Timer flush is registered only when `sink.enable-2pc=false`. It is intentionally disabled when `sink.enable-2pc=true` because flushing and opening a new Stream Load between checkpoints would break the 2PC transaction boundary and exactly-once guarantee. The initial timer flush implementation therefore provides at-least-once delivery only.

```hocon
env {
  job.mode = "STREAMING"
  checkpoint.interval = 300000
  sink.flush.interval = 5000
}

sink {
  Doris {
    fenodes = "doris-fe:8030"
    username = root
    password = ""
    database = "mydb"
    table = "mytable"
    sink.label-prefix = "timer-flush"
    sink.enable-2pc = false
    doris.batch.size = 10000
  }
}
```

## Troubleshooting 307 Temporary Redirect

If the job fails with `HTTP/1.1 307 Temporary Redirect`, check the following items first:

1. Whether the SeaTunnel worker can reach the redirected Doris BE address
2. Whether Doris FE is under heavy load, timeout, or Full GC pressure
3. Whether a proxy, SLB, ingress, or gateway rewrites or blocks the redirect path

If your environment already has reachable Doris BE HTTP addresses, you can configure `benodes` and set `direct_to_be=true` to bypass FE redirect on the data write path.

## Task Example

### Simple

> The following example describes writing multiple data types to Doris, and users need to create corresponding tables downstream

```hocon
env {
  parallelism = 1
  job.mode = "BATCH"
  checkpoint.interval = 10000
}

source {
  FakeSource {
    row.num = 10
    map.size = 10
    array.size = 10
    bytes.length = 10
    string.length = 10
    schema = {
      fields {
        c_map = "map<string, array<int>>"
        c_array = "array<int>"
        c_string = string
        c_boolean = boolean
        c_tinyint = tinyint
        c_smallint = smallint
        c_int = int
        c_bigint = bigint
        c_float = float
        c_double = double
        c_decimal = "decimal(16, 1)"
        c_null = "null"
        c_bytes = bytes
        c_date = date
        c_timestamp = timestamp
      }
    }
    }
}

sink {
  Doris {
    fenodes = "doris_cdc_e2e:8030"
    username = root
    password = ""
    database = "test"
    table = "e2e_table_sink"
    sink.label-prefix = "test-cdc"
    sink.enable-2pc = "true"
    sink.enable-delete = "true"
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

### Direct To BE

```hocon
sink {
  Doris {
    fenodes = "fe1:8030,fe2:8030"
    benodes = "be1:8040,be2:8040"
    direct_to_be = true
    username = root
    password = ""
    database = "test"
    table = "e2e_table_sink"
    sink.label-prefix = "test-direct-be"
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

### Direct To BE With 2PC

```hocon
sink {
  Doris {
    fenodes = "fe1:8030,fe2:8030"
    benodes = "be1:8040,be2:8040"
    direct_to_be = true
    username = root
    password = ""
    database = "test"
    table = "e2e_table_sink"
    sink.label-prefix = "test-direct-be-2pc"
    sink.enable-2pc = true
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

### CDC(Change Data Capture) Event

> This example defines a SeaTunnel synchronization task that automatically generates data through FakeSource and sends it to Doris Sink,FakeSource simulates CDC data with schema, score (int type),Doris needs to create a table sink named test.e2e_table_sink and a corresponding table for it.

```hocon
env {
  parallelism = 1
  job.mode = "BATCH"
  checkpoint.interval = 10000
}

source {
  FakeSource {
    schema = {
      fields {
        pk_id = bigint
        name = string
        score = int
        sex = boolean
        number = tinyint
        height = float
        sight = double
        create_time = date
        update_time = timestamp
      }
    }
    rows = [
      {
        kind = INSERT
        fields = [1, "A", 100, true, 1, 170.0, 4.3, "2020-02-02", "2020-02-02T02:02:02"]
      },
      {
        kind = INSERT
        fields = [2, "B", 100, true, 1, 170.0, 4.3, "2020-02-02", "2020-02-02T02:02:02"]
      },
      {
        kind = INSERT
        fields = [3, "C", 100, true, 1, 170.0, 4.3, "2020-02-02", "2020-02-02T02:02:02"]
      },
      {
        kind = UPDATE_BEFORE
        fields = [1, "A", 100, true, 1, 170.0, 4.3, "2020-02-02", "2020-02-02T02:02:02"]
      },
      {
        kind = UPDATE_AFTER
        fields = [1, "A_1", 100, true, 1, 170.0, 4.3, "2020-02-02", "2020-02-02T02:02:02"]
      },
      {
        kind = DELETE
        fields = [2, "B", 100, true, 1, 170.0, 4.3, "2020-02-02", "2020-02-02T02:02:02"]
      }
    ]
  }
}

sink {
  Doris {
    fenodes = "doris_cdc_e2e:8030"
    username = root
    password = ""
    database = "test"
    table = "e2e_table_sink"
    sink.label-prefix = "test-cdc"
    sink.enable-2pc = "true"
    sink.enable-delete = "true"
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}

```

### Use JSON format to import data

```hocon
sink {
    Doris {
        fenodes = "e2e_dorisdb:8030"
        username = root
        password = ""
        database = "test"
        table = "e2e_table_sink"
        sink.enable-2pc = "true"
        sink.label-prefix = "test_json"
        doris.config = {
            format = "json"
            read_json_by_line = "true"
        }
    }
}
```

### Use CSV format to import data

```hocon
sink {
    Doris {
        fenodes = "e2e_dorisdb:8030"
        username = root
        password = ""
        database = "test"
        table = "e2e_table_sink"
        sink.enable-2pc = "true"
        sink.label-prefix = "test_csv"
        doris.config = {
          format = "csv"
          column_separator = ","
        }
    }
}
```

### Case-Sensitive Configuration

```hocon
sink {
    Doris {
        fenodes = "e2e_dorisdb:8030"
        username = root
        password = ""
        database = "Test_DB"  # Original case will be preserved
        table = "Test_Table"  # Original case will be preserved
        case_sensitive = true # Default value, preserves original case
        sink.enable-2pc = "true"
        sink.label-prefix = "test_case_sensitive"
        doris.config = {
          format = "json"
          read_json_by_line = "true"
        }
    }
}
```

### Multiple table

#### example1

```hocon
env {
  parallelism = 1
  job.mode = "STREAMING"
  checkpoint.interval = 5000
}

source {
  Mysql-CDC {
    url = "jdbc:mysql://127.0.0.1:3306/seatunnel"
    username = "root"
    password = "******"

    table-names = ["seatunnel.role","seatunnel.user","galileo.Bucket"]
  }
}

transform {
}

sink {
  Doris {
    fenodes = "doris_cdc_e2e:8030"
    username = root
    password = ""
    database = "${database_name}_test"
    table = "${table_name}_test"
    sink.label-prefix = "test-cdc"
    sink.enable-2pc = "true"
    sink.enable-delete = "true"
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

#### example2

```hocon
env {
  parallelism = 1
  job.mode = "BATCH"
}

source {
  Jdbc {
    driver = oracle.jdbc.driver.OracleDriver
    url = "jdbc:oracle:thin:@localhost:1521/XE"
    user = testUser
    password = testPassword

    table_list = [
      {
        table_path = "TESTSCHEMA.TABLE_1"
      },
      {
        table_path = "TESTSCHEMA.TABLE_2"
      }
    ]
  }
}

transform {
}

sink {
  Doris {
    fenodes = "doris_cdc_e2e:8030"
    username = root
    password = ""
    database = "${schema_name}_test"
    table = "${table_name}_test"
    sink.label-prefix = "test-cdc"
    sink.enable-2pc = "true"
    sink.enable-delete = "true"
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

### CDC source feeding Doris Sink (streaming + timer flush)

This pattern combines a MySQL-CDC source with a Doris Sink and relies on Zeta's `sink.flush.interval` to commit Stream Load transactions at a fixed cadence.

```hocon
env {
  parallelism = 1
  job.mode = "STREAMING"
  checkpoint.interval = 300000
  sink.flush.interval = 500
}

source {
  MySQL-CDC {
    parallelism = 1
    server-id = 5664
    username = "st_user_source"
    password = "mysqlpw"
    table-names = ["mysql_cdc.mysql_cdc_e2e_source_table"]
    url = "jdbc:mysql://mysql_doris_timer_flush_e2e:3306/mysql_cdc"
  }
}

sink {
  Doris {
    fenodes = "doris_e2e:8030"
    username = root
    password = ""
    database = "timer_flush"
    table = "doris_timer_flush"
    sink.label-prefix = "timer-flush"
    sink.enable-2pc = false
    doris.batch.size = 100000
    schema_save_mode = "IGNORE"
    data_save_mode = "APPEND_DATA"
    doris.config {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

### Custom SQL with `save_mode_create_template`

When the connector should run your own INSERT statements instead of the generated upsert, set `data_save_mode = "CUSTOM_PROCESSING"` and provide both `custom_sql` and `save_mode_create_template`:

```hocon
sink {
  Doris {
    fenodes = "doris_e2e:8030"
    username = root
    password = ""
    table.identifier = "e2e_sink.doris_e2e_unique_table"
    data_save_mode = "CUSTOM_PROCESSING"
    custom_sql = "INSERT INTO e2e_sink.doris_e2e_unique_table (F_ID,F_INT,F_BIGINT) VALUES (1, 123, 1234567890123);"
    sink.enable-2pc = "true"
    sink.buffer-size = 2
    sink.buffer-count = 2
    sink.label-prefix = "test_json"
    doris.config = {
      format = "json"
      read_json_by_line = "true"
    }
    save_mode_create_template = """CREATE TABLE IF NOT EXISTS `${database}`.`${table}` (${rowtype_fields}) ENGINE=OLAP unique KEY (`F_ID`) DISTRIBUTED BY HASH (`F_ID`) PROPERTIES ("replication_allocation" = "tag.location.default: 1")"""
  }
}
```

### Direct To BE with 2PC and `table.identifier`

When you have reachable Doris BE addresses and want to bypass FE redirect on the write path, combine `direct_to_be=true` with `benodes`, `table.identifier`, and 2PC:

```hocon
sink {
  Doris {
    fenodes = "doris_e2e:8030"
    benodes = "doris_e2e:8040"
    direct_to_be = "true"
    schema_save_mode = "RECREATE_SCHEMA"
    username = root
    password = ""
    table.identifier = "e2e_sink.doris_e2e_unique_table"
    sink.enable-2pc = "true"
    sink.label-prefix = "test_json_direct_to_be"
    doris.config = {
      format = "json"
      read_json_by_line = "true"
    }
  }
}
```

## FAQ

### Does Doris Sink support automatic table creation?

Yes. Use the `schema_save_mode` and `save_mode_create_template` sections above as the canonical
reference for the exact behavior, defaults, and DDL customization path.

### How does exactly-once work with Doris Sink?

Doris Sink uses Stream Load with two-phase commit (2PC) for exactly-once semantics:

```hocon
sink {
  Doris {
    fenodes = "doris-fe:8030"
    username = root
    password = ""
    database = "mydb"
    table = "mytable"
    sink.enable-2pc = "true"
    sink.label-prefix = "unique-job-label"
  }
}
```

The `sink.label-prefix` must be unique per job to avoid label conflicts when retrying or restarting.

### Why do I get a "Label already exists" error?

Doris uses Stream Load labels to detect and reject duplicate submissions. If a job is restarted with 2PC enabled, the same label prefix may be reused. To resolve:

- Include a timestamp or unique token in `sink.label-prefix` to ensure uniqueness across restarts.
- Abort uncommitted transactions in Doris before restarting: `CANCEL LOAD WHERE LABEL LIKE 'your-prefix%'`.

### Does Doris Sink support DELETE propagation from CDC sources?

Yes. Set `sink.enable-delete = "true"` to propagate DELETE operations from CDC sources (e.g., MySQL CDC) to Doris. This requires the target table to use the Unique Key model in Doris.

### Are Doris column names case-sensitive?

See the case-sensitivity example above for the exact behavior. If upstream field names still do not
match the Doris schema, normalize them before the sink stage or align the target schema explicitly
instead of relying on an undocumented `column_mapping` option.

### What data format does Doris Stream Load use?

Doris Sink uses JSON format for Stream Load by default. Configure it explicitly if needed:

```hocon
doris.config {
  format = "json"
  read_json_by_line = "true"
}
```

CSV format is also supported but requires careful delimiter configuration.

## Changelog

<ChangeLog />
