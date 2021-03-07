#### 准备Vagrant

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
    node.vm.box = "ubuntu18"

    node.vm.network "private_network", ip: "192.168.33.#{i}"

    node.vm.synced_folder "./data", "/vagrant_data"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "node#{i}"
      vb.cpus = 4
      vb.memory = "2048"
    end
    end
  end
end
```

#### 设置SSH免密登录

```sh
cp ~/.ssh/id_rsa.pub ./data # 将宿主机的公钥拷贝到虚拟机共享目录
vagrant ssh node1 # 登录到node1机器
sudo adduser hadoop # 创建hadoop用户
sudo su - hadoop
# 添加sudo权限
ssh-keygen -t rsa # 生成秘钥
cat /vagrant_data/id_rsa.pub >> ~/.ssh/authorized_keys # 配置vagrant免密登录
sudo chmod 777 -R /opt/

vagrant ssh node2 # 登录到node2机器
sudo adduser hadoop # 创建hadoop用户
sudo su - hadoop
ssh-keygen -t rsa # 生成秘钥
cat /vagrant_data/id_rsa.pub >> ~/.ssh/authorized_keys # 配置vagrant免密登录
sudo chmod 777 -R /opt/


vagrant ssh node3 # 登录到node3机器
sudo adduser hadoop # 创建hadoop用户
sudo su - hadoop
ssh-keygen -t rsa # 生成秘钥
cat /vagrant_data/id_rsa.pub >> ~/.ssh/authorized_keys # 配置vagrant免密登录
sudo chmod 777 -R /opt/
```

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
    node.vm.box = "ubuntu18"

    node.vm.network "private_network", ip: "192.168.33.#{i}"
    node.vm.network "public_network", ip: "192.168.44.#{i}", bridge: "Intel(R) Wi-Fi 6 AX200 160MHz"

    node.vm.synced_folder "./data", "/vagrant_data"
    node.ssh.username = "hadoop"
    node.ssh.private_key_path = "C:/Users/LuZhong/.ssh/id_rsa"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "node#{i}"
      vb.cpus = 4
      vb.memory = "2048"
    end
    end
  end
end
```

```shell
# 下载hadoop二进制包
mkdir Download
cd Download
wget https://cdn.azul.com/zulu/bin/zulu8.52.0.23-ca-jdk8.0.282-linux_x64.tar.gz
sudo tar -zxvf zulu8.52.0.23-ca-jdk8.0.282-linux_x64.tar.gz -C /opt/


wget https://apache.claz.org/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz
sudo tar -zxvf hadoop-3.2.2.tar.gz -C /opt/
cd /opt
mv zulu8.52.0.23-ca-jdk8.0.282-linux_x64 jdk8

mv hadoop-3.2.2 hadoop

cd hadoop
vim ./etc/hadoop/core-site.xml

<configuration>
  <!-- 指定HDFS访问的域名地址  -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node1:8020</value>
	</property>
    <!-- 临时文件存储目录  -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/hadoop/hadoopDatas/tempDatas</value>
	</property>
	<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>
	</property>

	<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
	<property>
		<name>fs.trash.interval</name>
		<value>10080</value>
	</property>
</configuration>

vim ./etc/hadoop/hdfs-site.xml
<configuration>
  <property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>node01:50090</value>
	</property>
	<!--  hdfs的集群的web  访问地址  -->
	<property>
		<name>dfs.namenode.http-address</name>
		<value>node1:50070</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///opt/hadoop/hadoopDatas/namenodeDatas</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用逗号进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///opt/hadoop/hadoopDatas/hadoopDatas/datanodeDatas</value>
	</property>
	  <!-- edits产生的文件存放路径 -->
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///opt/hadoop/hadoopDatas/hadoopDatas/dfs/nn/edits</value>
	</property>
    
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///opt/hadoop/hadoopDatas/hadoopDatas/dfs/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///opt/hadoop/hadoopDatas/hadoopDatas/dfs/nn/snn/edits</value>
	</property>
    <!-- 默认文件存储的副本数量 -->
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
    <!-- 关闭hdfs的文件权限 -->
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
    <!-- 默认 文件的存储的块的大小 -->
	<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
</configuration>

vim ./etc/hadoop/yarn-site.xml

<configuration>
  <!-- yarn resourcesmanager 的主机地址 -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>node1</value>
	</property>
  <!-- 逗号隔开的服务列表，列表名称应该只包含a-zA-Z0-9_,不能以数字开始-->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<!-- 启用日志聚合功能，应用程序完成后，收集各个节点的日志到一起便于查看 -->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
  <property>
		 <name>yarn.log.server.url</name>
		 <value>http://node1:19888/jobhistory/logs</value>
  </property>
  <!--多长时间将聚合删除一次日志 此处 30 day -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>2592000</value>
	</property>
  <!--时间在几秒钟内保留用户日志。只适用于如果日志聚合是禁用的-->
  <property>
    <name>yarn.nodemanager.log.retain-seconds</name>
    <value>604800</value><!--7 day-->
  </property>
    
  <!--指定文件压缩类型用于压缩汇总日志-->
  <property>
    <name>yarn.nodemanager.log-aggregation.compression-type</name>
    <value>gz</value>
  </property>
  <!-- nodemanager本地文件存储目录-->
  <property>
    <name>yarn.nodemanager.local-dirs</name>
    <value>/opt/hadoop/hadoopDatas/yarn/local</value>
  </property>
  <!-- resourceManager  保存最大的任务完成个数 -->
  <property>
    <name>yarn.resourcemanager.max-completed-applications</name>
    <value>1000</value>
  </property>
</configuration>


scp scp -r jdk8/ node2:$PWD
scp scp -r jdk8/ node3:$PWD

scp scp -r hadoop/ node2:$PWD
scp scp -r hadoop/ node3:$PWD

# node1
vim ./etc/hadoop/workers
node1
node2
node3

vim ~/.profile
export JAVA_HOME=/opt/jdk8
export HADOOP_HOME=/opt/hadoop

PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin/:$JAVA_HOME/bin:$PATH"

# node2
vim ~/.profile
export JAVA_HOME=/opt/jdk8
export HADOOP_HOME=/opt/hadoop

PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin/:$JAVA_HOME/bin:$PATH"

# node3
vim ~/.profile
export JAVA_HOME=/opt/jdk8
export HADOOP_HOME=/opt/hadoop

PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin/:$JAVA_HOME/bin:$PATH"
```

