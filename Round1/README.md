
     1.安装虚拟机
        注意最好四台机器，1个master2个slaver，所有的安装的目录要一致，使用桥接模式
     2.安装securecrt，进行ssh免秘钥连接
     3.安装jdk环境
        下载jdk1.8安装包，进行解压后，进行环境配置（a在主目录下的.bashrc下最后加入就行）
     4.安装hadoop
       下载hadoop2.6.5的安装包，进行解压
       设置好各个虚拟机的名称（在etc/hostname中修改）
       设置映射关系（/etc/hosts中设置，把所有机器的ip地址与主机名写上）
       配置hadoop的文件：
         $HADOOP_HOME/etc/hadoop/hadoop-env.sh
         $HADOOP_HOME/etc/hadoop/yarn-env.sh
         $HADOOP_HOME/etc/hadoop/core-site.xml
         $HADOOP_HOME/etc/hadoop/hdfs-site.xml
         $HADOOP_HOME/etc/hadoop/mapred-site.xml
         $HADOOP_HOME/etc/hadoop/yarn-site.xml
         $HADOOP_HOME/etc/hadoop/slaves
         
         以下是详细的配置：
         1) hadoop-env.sh 、yarn-env.sh，mapred_env.sh
             这二个文件主要是修改JAVA_HOME后的目录，改成实际本机jdk所在目录位置
             vi etc/hadoop/hadoop-env.sh （及 vi etc/hadoop/yarn-env.sh）
               在这一行上添加：
             export JAVA_HOME=/home/hadoop/jdk1.8(按具体情况修改)
          2) core-site.xml 参考下面的内容修改：
              1 <?xml version="1.0" encoding="UTF-8"?>
              2 <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
              3 <configuration>
              4   <property>
              5     <name>fs.defaultFS</name>
              6     <value>hdfs://master:9000</value>
              7   </property>
              8   <property>
              9     <name>hadoop.tmp.dir</name>
             10     <value>/home/hadoop/tmp</value>
             11   </property>
             12 </configuration>   
                 注：/home/hadoop/tmp 目录如不存在，则先mkdir手动创建     
         3) hdfs-site.xml
              1 <?xml version="1.0" encoding="UTF-8"?>
              2 <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
             <configuration>
                 <property>
                     <name>dfs.namenode.secondary.http-address</name>
                     <value>slave1:50090</value>
                  </property>
                 <property>
                     <name>dfs.replication</name>
                     <value>3</value>
                 </property>
                  <property>
                     <name>dfs.namenode.name.dir</name>
                     <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/name</value>
                  </property>
                  <property>
                     <name>dfs.datanode.data.dir</name>
                     <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/data</value>
                  </property>
             </configuration>
  
                               
         4) mapred-site.xml
             <configuration>
                 <property>
                     <name>mapreduce.framework.name</name>
                     <value>yarn</value>
                 </property>
                 <property>
                     <name>mapreduce.jobhistory.address</name>
                     <value>master:10020</value>
                 </property>
                 <property>
                     <name>mapreduce.jobhistory.webapp.address</name>
                     <value>master:19888</value>
                 </property>
             </configuration>
  
                              
 
         5)yarn-site.xml
            <configuration>
                 <property>
                     <name>yarn.resourcemanager.hostname</name>
                     <value>master</value>
                 </property>
                 <property>
                     <name>yarn.nodemanager.aux-services</name>
                     <value>mapreduce_shuffle</value>
              </property>
             </configuration>
             
         6) 修改slaves
             vi slaves 编辑该文件，输入
             slave01
             slave02

 
        7：将master上的hadoop目录复制到slave01,slave02，slave03
             在在master机器的主目录下运行拷贝
             scp -r hadoop-2.6.5 hadoop@slave01:/home/hadoop/
             scp -r hadoop-2.6.5 hadoop@slave02:/home/hadoop/
             scp -r hadoop-2.6.5 hadoop@slave03:/home/hadoop/
 
         8：验证是否成功启动
             master节点上，重新启动
             $HADOOP_HOME/sbin/start-all.sh
             检查master节点上有几下3个进程：
                  ResourceManager
                  SecondaryNameNode
                  NameNode
 
             slave01、slave02，slave03上有几下2个进程：
                 DataNode
                 NodeManager
 
 1、UV：（1）在map中对日志数据正则表达式筛选处理，得到ip，时间（年、月、日）
      （2）时间为key ip为value，
      （3）combiner先处理下日志数据减少数据
      （4）reduce中去重掉ip输出ip数量就是UV(访问用户量)
      （5）通过对时间的规定，选择来统计每日、每周、每月的UV数量
      
 2、PV：（1）在map中对日志数据正则表达式筛选处理，得到时间，访问地址
      （2）将时间作为key，访问地址作为values
      （3）combiner先处理下数据减少数据
      （4）reduce中进行统计，统计values的总数量即PV
      
 3、留存：（1）统计次日就把二日的数据输入进来，统计次月就把二月的数据输入进来
       （2）把数据在map里进行正则处理同时是从哪个文件里给予标记，得到的得到ip，标记
       （3）在reduce中判断是否同时存在二个文件标记，存在就加一
       （4）输出出数目就是留存
       
 4、跳转率（1）map读取数据正则分出ip values为show或musician
         （2）reduce统计只有一条values值的IP数目输出出来
         （3）除以总的UV即为跳转率
         
 5、baidu跳转的PV：（1）map读取数据正则分割出哪里跳转的只截取百度跳转的作为key，values定义1
                  （2）reduce中统计values数目
                  （3）输的数目即为baidu跳转的PV
 6、每天iOS和Android 
           （1）map中读取数据，正则处理数据
           （2）截取是ios和Android的用户作为key值 ip作为values值
           （3）reduce中把values值放入set集合中去重，计算出数量
           （4）即为每个设备的UV数
           
     实时查询：（1）当前show的访问数量：把访问show的放到hbase中存储，查询直接从Hbase中查询但钱存储的数量
              （2）当前musician的访问数量： 把访问musician的放到hbase中存储，查询直接从Hbase中查询但钱存储的数量
            
  项目拆解：（1）12月9号完成方案设计和时间规划
           （2） 12月10、11、12号完成UV、pv、留存
           （3） 12月13、14、15号完成跳转率、baidu跳转的PV、ios和Androif的UV数
