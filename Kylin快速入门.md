# Kylin快速入门

#### 使用Docker运行Kylin

因为Kylin依赖 JDK、Hadoop、Hive、Hbase、Spark、Kafka和MySQL等大量服务，对于初学者来说有相当大的难度。因此Kylin面向初学用户提供了包含上诉所有服务的docker镜像。

```shell
# 拉去镜像
docker pull apachekylin/apache-kylin-standalone:3.1.0
```

```shell
# 启动容器
docker run -d \
-m 8G \
-p 7070:7070 \
-p 8088:8088 \
-p 50070:50070 \
-p 8032:8032 \
-p 8042:8042 \
-p 16010:16010 \
apachekylin/apache-kylin-standalone:3.1.0
```

在容器启动时会自动启动NameNode、DataNode、ResourceManager, NodeManager、HBase、Kafka和Kylin服务，并自动运行 `$KYLIN_HOME/bin/sample.sh`，以及在 Kafka 中创建 “kylin_streaming_topic” topic 并持续向该 topic 中发送数据。这是为了让用户启动容器后，就能体验以批和流的方式的方式构建 Cube 并进行查询。

容器启动后，我们可以通过 “docker exec -it <container_id> bash” 命令进入容器内。当然，由于我们已经将容器内指定端口映射到本机端口，我们可以直接在本机浏览器中打开各个服务的页面，如：

- Kylin 页面：http://127.0.0.1:7070/kylin/login
- HDFS NameNode 页面：[http://127.0.0.1:50070](http://127.0.0.1:50070/)
- YARN ResourceManager 页面：[http://127.0.0.1:8088](http://127.0.0.1:8088/)
- HBase 页面：[http://127.0.0.1:16010](http://127.0.0.1:16010/)

#### 使用样例数据构建Cube

1. 在浏览器中访问http://127.0.0.1:7070/kylin/login

输入用户名: ADMIN，密码：KYLIN登入Kylin

![image-20210310211858543](https://github.com/BeanCookie/note-images/blob/main/kylin001.png)

2. 选择名为 kylin_sales_cube 的样例 Cube，点击 “Actions” -> “Build” 
![image-20210310212101546](https://github.com/BeanCookie/note-images/blob/main/kylin002.png)

3. 选择一个在 2014-01-01 之后的日期
![image-20210310212227074](https://github.com/BeanCookie/note-images/blob/main/kylin003.png)

4. 点击 “Monitor” 标签，查看 build 进度直至 100%

![image-20210310212415834](https://github.com/BeanCookie/note-images/blob/main/kylin004.png)

5. 当kylin_sales_cube的状态变为READY后
![image-20210310213439230](https://github.com/BeanCookie/note-images/blob/main/kylin005.png)
点击 “Insight” 标签，执行 SQLs，例如

```sql
select part_dt, sum(price) as total_sold, count(distinct seller_id) as sellers from kylin_sales group by part_dt order by part_dt
```

![image-20210310213646296](https://github.com/BeanCookie/note-images/blob/main/kylin006.png)

可以看到90多MB的数据聚合查询只需要0.14秒

6. 在Hive执行同样的SQL

```shell
hive
hive> use default;
select part_dt, sum(price) as total_sold, count(distinct seller_id) as sellers from kylin_sales group by part_dt order by part_dt;
```

![image-20210310214235543](https://github.com/BeanCookie/note-images/blob/main/kylin007.png)

同样的结果Hive中要执行将近30秒，整整快了200多倍。