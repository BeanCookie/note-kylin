# 从零创建Cube

#### 新建项目

1. 点击 `+ Project` 按钮添加一个新的项目。

![image-20210317212539690](https://github.com/BeanCookie/note-images/blob/main/kylin008.png)

2. 填写下列表单并点击 `submit` 按钮提交请求。

![image-20210317212632774](https://github.com/BeanCookie/note-images/blob/main/kylin009.png)

#### 准备Hive数据

1. 在顶部菜单栏点击 `Model`，然后点击左边的 `Data Source` 标签，它会列出所有加载进 Kylin 的表，点击 `Load Table` 按钮。

![image-20210317212821453](https://github.com/BeanCookie/note-images/blob/main/kylin010.png)

2. 输入表名并点击 `Sync` 按钮提交请求。

![image-20210317212841133](https://github.com/BeanCookie/note-images/blob/main/kylin011.png)

3. 【可选】如果你想要浏览 hive 数据库来选择表，点击 `Load Table From Tree` 按钮。

![image-20210317213152272](https://github.com/BeanCookie/note-images/blob/main/kylin012.png)

![image-20210317213203949](https://github.com/BeanCookie/note-images/blob/main/kylin013.png)

4. 成功的消息将会弹出，在左边的 `Tables` 部分，新加载的表已经被添加进来。点击表将会展开列。

![image-20210317212908212](https://github.com/BeanCookie/note-images/blob/main/kylin014.png)

5. 在后台Kylin 将会执行 MapReduce 任务计算新同步表的基数（cardinality），任务完成后刷新页面并点击表名，基数值将会显示在表信息中。

#### 创建Model

创建 cube 前需定义一个数据模型。数据模型定义了一个星型（star schema）或雪花（snowflake schema）模型，一个模型可以被多个 cube 使用。

1. 点击顶部的 `Model` ，然后点击 `Models` 标签。点击 `+New` 按钮，在下拉框中选择 `New Model`。
![kylin016](https://github.com/BeanCookie/note-images/blob/main/kylin016.png)
2. 输入 model 的名字和可选的描述。
![kylin017](https://github.com/BeanCookie/note-images/blob/main/kylin017.png)
3. 在 `Fact Table` 中，为模型选择事实表。
![kylin018](https://github.com/BeanCookie/note-images/blob/main/kylin018.png)
4. 【可选】点击 `Add Lookup Table` 按钮添加一个 lookup 表。选择表名和关联类型（内连接或左连接）。因为目前我们只导入了``KYLIN_CAL_DT``这一张Hive表，所有要导入Hive中的另两张`KYLIN_ACCOUNT``和``KYLIN_SALES``表。
![kylin015](https://github.com/BeanCookie/note-images/blob/main/kylin015.png)
5. 点击 `New Join Condition` 按钮，左边选择事实表的外键，右边选择 lookup 表的主键。如果有多于一个 join 列重复执行。
![kylin019](https://github.com/BeanCookie/note-images/blob/main/kylin019.png)
6. 点击 “OK”，重复4，5步来添加更多的 lookup 表。完成后，点击 “Next”。
![kylin020](https://github.com/BeanCookie/note-images/blob/main/kylin020.png)
7. `Dimensions` 页面允许选择在子 cube 中用作维度的列，然后点击 `Columns` 列，在下拉框中选择需要的列。
![kylin021](https://github.com/BeanCookie/note-images/blob/main/kylin021.png)
8. 点击 “Next” 到达 “Measures” 页面，选择作为 measure 的列，其只能从事实表中选择。
![kylin022](https://github.com/BeanCookie/note-images/blob/main/kylin022.png)
9. 点击 “Next” 到达 “Settings” 页面，如果事实表中的数据每日增长，选择 `Partition Date Column` 中相应的 日期列以及日期格式，否则就将其留白。
10. 【可选】选择是否需要 “time of the day” 列，默认情况下为 `No`。如果选择 `Yes`, 选择 `Partition Time Column` 中相应的 time 列以及 time 格式
11. 【可选】如果在从 hive 抽取数据时候想做一些筛选，可以在 `Filter` 中输入筛选条件。
![kylin023](https://github.com/BeanCookie/note-images/blob/main/kylin023.png)
12. 点击 `Save` 然后选择 `Yes` 来保存 data model。创建完成，data model 就会列在左边 `Models` 列表中。

#### 创建Cube
点击顶部 Model，然后点击 Models 标签。点击 +New 按钮，在下拉框中选择 New Cube。
![kylin024](https://github.com/BeanCookie/note-images/blob/main/kylin024.png)
- 设置Cube信息
1. 选择 data model，输入 cube 名字；点击 Next 进行下一步。
2. cube 名字可以使用字母，数字和下划线（空格不允许）。Notification Email List 是运用来通知job执行成功或失败情况的邮箱列表。Notification Events 是触发事件的状态。
![kylin025](https://github.com/BeanCookie/note-images/blob/main/kylin025.png)
- 设置维度
1. 点击 Add Dimension，在弹窗中显示的事实表和 lookup 表里勾选输入需要的列。Lookup 表的列有2个选项：“Normal” 和 “Derived”（默认）。“Normal” 添加一个普通独立的维度列，“Derived” 添加一个 derived 维度，derived 维度不会计算入 cube，将由事实表的外键推算出。
![kylin026](https://github.com/BeanCookie/note-images/blob/main/kylin026.png)
2. 选择所有维度后点击 “Next”。
![kylin027](https://github.com/BeanCookie/note-images/blob/main/kylin027.png)
- 设置度量
1. 点击 +Measure 按钮添加一个新的度量。
2. 根据它的表达式共有8种不同类型的度量：SUM、MAX、MIN、COUNT、COUNT_DISTINCT TOP_N, EXTENDED_COLUMN 和 PERCENTILE。请合理选择 COUNT_DISTINCT 和 TOP_N 返回类型，它与 cube 的大小相关。
![kylin028](https://github.com/BeanCookie/note-images/blob/main/kylin028.png)
- 更新设置，这一步骤是为增量构建 cube 而设计的。
![kylin029](https://github.com/BeanCookie/note-images/blob/main/kylin029.png)
Auto Merge Thresholds: 自动合并小的 segments 到中等甚至更大的 segment。如果不想自动合并，删除默认2个选项。

Volatile Range: 默认为0，会自动合并所有可能的 cube segments，或者用 ‘Auto Merge’ 将不会合并最新的 [Volatile Range] 天的 cube segments。

Retention Threshold: 只会保存 cube 过去几天的 segment，旧的 segment 将会自动从头部删除；0表示不启用这个功能。

Partition Start Date: cube 的开始日期.
- 高级设置
![kylin030](https://github.com/BeanCookie/note-images/blob/main/kylin030.png)
Aggregation Groups: Cube 中的维度可以划分到多个聚合组中。默认 kylin 会把所有维度放在一个聚合组，当维度较多时，产生的组合数可能是巨大的，会造成 Cube 爆炸；如果你很好的了解你的查询模式，那么你可以创建多个聚合组。在每个聚合组内，使用 “Mandatory Dimensions”, “Hierarchy Dimensions” 和 “Joint Dimensions” 来进一步优化维度组合。

Mandatory Dimensions: 必要维度，用于总是出现的维度。例如，如果你的查询中总是会带有 “ORDER_DATE” 做为 group by 或 过滤条件, 那么它可以被声明为必要维度。这样一来，所有不含此维度的 cuboid 就可以被跳过计算。

Hierarchy Dimensions: 层级维度，例如 “国家” -> “省” -> “市” 是一个层级；不符合此层级关系的 cuboid 可以被跳过计算，例如 [“省”], [“市”]. 定义层级维度时，将父级别维度放在子维度的左边。

Joint Dimensions:联合维度，有些维度往往一起出现，或者它们的基数非常接近（有1:1映射关系）。例如 “user_id” 和 “email”。把多个维度定义为组合关系后，所有不符合此关系的 cuboids 会被跳过计算。

Rowkeys: 是由维度编码值组成。”Dictionary” （字典）是默认的编码方式; 字典只能处理中低基数（少于一千万）的维度；如果维度基数很高（如大于1千万), 选择 “false” 然后为维度输入合适的长度，通常是那列的最大长度值; 如果超过最大值，会被截断。请注意，如果没有字典编码，cube 的大小可能会非常大。
你可以拖拽维度列去调整其在 rowkey 中位置; 位于rowkey前面的列，将可以用来大幅缩小查询的范围。通常建议将 mandantory 维度放在开头, 然后是在过滤 ( where 条件)中起到很大作用的维度；如果多个列都会被用于过滤，将高基数的维度（如 user_id）放在低基数的维度（如 age）的前面。

Mandatory Cuboids: 维度组合白名单。确保你想要构建的 cuboid 能被构建。

Cube Engine: cube 构建引擎。有两种：MapReduce 和 Spark。如果你的 cube 只有简单度量（SUM, MIN, MAX)，建议使用 Spark。如果 cube 中有复杂类型度量（COUNT DISTINCT, TOP_N），建议使用 MapReduce。

Advanced Dictionaries: “Global Dictionary” 是用于精确计算 COUNT DISTINCT 的字典, 它会将一个非 integer的值转成 integer，以便于 bitmap 进行去重。如果你要计算 COUNT DISTINCT 的列本身已经是 integer 类型，那么不需要定义 Global Dictionary。 Global Dictionary 会被所有 segment 共享，因此支持在跨 segments 之间做上卷去重操作。请注意，Global Dictionary 随着数据的加载，可能会不断变大。

“Segment Dictionary” 是另一个用于精确计算 COUNT DISTINCT 的字典，与 Global Dictionary 不同的是，它是基于一个 segment 的值构建的，因此不支持跨 segments 的汇总计算。如果你的 cube 不是分区的或者能保证你的所有 SQL 按照 partition_column 进行 group by, 那么你应该使用 “Segment Dictionary” 而不是 “Global Dictionary”，这样可以避免单个字典过大的问题。

请注意：”Global Dictionary” 和 “Segment Dictionary” 都是单向编码的字典，仅用于 COUNT DISTINCT 计算(将非 integer 类型转成 integer 用于 bitmap计算)，他们不支持解码，因此不能为普通维度编码。

Advanced Snapshot Table: 为全局 lookup 表而设计，提供不同的存储类型。

Advanced ColumnFamily: 如果有超过一个的COUNT DISTINCT 或 TopN 度量, 你可以将它们放在更多列簇中，以优化与HBase 的I/O。
-  重写配置
Kylin 允许在 Cube 级别覆盖部分 kylin.properties 中的配置，你可以在这里定义覆盖的属性。如果你没有要配置的，点击 Next 按钮
![kylin031](https://github.com/BeanCookie/note-images/blob/main/kylin031.png)

- 概览 & 保存
![kylin032](https://github.com/BeanCookie/note-images/blob/main/kylin032.png)