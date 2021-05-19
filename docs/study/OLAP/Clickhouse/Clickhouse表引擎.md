# Clickhouse表引擎
## VersionedCollapsingMergeTree：
> VersionedCollapsingMergeTree 引擎继承自 MergeTree并将折叠行的逻辑添加到合并数据部分得算法中。VersionedCollapsingMergeTree用于相同的目的 CollapsingMergeTree
> 但使用不同的折叠算法，允许以多个线程的任何顺序插入数据。Version列有助于正确折叠行，即使它们以错误的顺序插入。相比之下，ColllapsingMergeTree只允许严格连续插入。

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = VersionedCollapsingMergeTree(sign, version)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]

```

### 引擎参数
```sql
VersionedCollapsingMergeTree(sign, version)
```
- sign：指定行类型的列名： 1 是一个"state"行， -1 是一个"cancel" 行，列数据类型应为 Int8
- version：指定对象状态版本的列名。列数据类型应为 UInt*

#### Versioned data
考虑一种情况，您需要为某个对象保存不断变化的数据。 对于一个对象有一行，并在发生更改时更新该行是合理的。 但是，对于数据库管理系统来说，更新操作非常昂贵且速度很慢，因为它需要重写存储中的数据。 如果需要快速写入数据，则不能接受更新，但可以按如下顺序将更改写入对象。

使用Sign列写入行时。如果 Sign = 1 这意味着该行是一个对象的状态（让我们把它称为 “state” 行）。 如果 Sign = -1 它指示具有相同属性的对象的状态的取消（让我们称之为 “cancel” 行）。 还可以使用 Version 列，它应该用单独的数字标识对象的每个状态。


