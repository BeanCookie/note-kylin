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