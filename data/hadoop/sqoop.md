# Sqoop

## 增量导入

sqoop import

可以进行增量导入，在业务处理中经常被使用。

### 核心参数


| Argument | Description |
| --- | --- |
| --check-column</br>(col) | 指定导入时需要检查的列，使用此列作为判断数据是否为增量数据</br>(这些列的类型不能为CHAR/NCHAR/VARCHAR/VARNCHAR/ LONGVARCHAR/LONGNVARCHAR)</br>可以指定多个列 |
| --incremental</br>(mode) | 指定如何判断行为新的数据</br>有两种模式`append`和`lastmodified` |
| --last-value</br>(value) | 指定上次导入数据时检查列的最大值(最新值) |

### 使用

#### append

当表中数据会有一个字段在新行中是不断增加的值时(例如自增键)，适合使用`append`模式。Sqoop将导入`--check-column`指定的列大于`--last-value`指定值的行。

| Argument | Description |
| --- | --- |
| --incremental append | 基于递增列的增量导入 |
| --check-column | 递增列(非字符) |
| --last-value | 阈值(导入**大于**阈值的行) |

#### lastmodified

当源表的行可能会更新时，应该使用`lastmodified`模式。此类更新将会保存上次修改的列设置为当前的时间戳，要求原表中有可以指定时间戳的字段。Sqoop将导入`--check-column`列保存的时间戳比`--last-value`指定的时间戳更新的时间戳的行。

在增量导入后，应该将打印到屏幕上的`--last-value`的值记录下来用于以后的增量导入。当运行后面的导入时，应该指定`--last-value`来确保只导入新增或者更新的数据。可以通过将增量导入作为一个 `saved job` 来自动处理此问题，这是执行循环增量导入的首选机制。

可以制定 `--merge-key` 参数以保证将后续新的记录与原有的记录合并。

| Argument | Description |
| --- | --- |
| --incremental lastmodified | 基于时间列的增量导入 |
| --check-column | 递增列(非字符) |
| --last-value | 阈值(导入**大于等于**阈值的行) |
| --merge-key | 合并列(主键，合并键值相同的记录) |

### 问题

为了防止数据库被Sqoop读取数据太多抗不住压力，所以一般从备库中抽取数据。

通过-m可以设置导入数据的任务数量，指定了-m表示为并发导入，这时候就必须指定`--split-by`参数，明确根据哪一列进行哈希分片，这个列应根据实际情况慎重选择，以避免产生数据倾斜。

空值替换问题

- `--null-string`：string类型的空值替换符(hive中null用\n表示)
- `--null-non-string`：非string类型空值的替换符
