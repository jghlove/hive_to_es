通过Hive查询语句查出结果集，并向Elasticsearch导入的小工具

方便从hive导查询结果到ES的python脚本，采用分页查询机制，结果集合过多时不会撑爆内存<br>


公司的数据分析经常需要看各种报表，多是分析统计类需求，类SQL语言适合做具有筛选逻辑的数据
（有时候有的数据无法从业务主库中查出来，只能直接前端埋点），
Elasticsearch适合做统计，而且结合Kibana可以直接生成报表！

耗时查询不宜直接在线上查，还好公司已经实现每天在访问低谷期同步线上数据到Hadoop大数据中心。
    
对这类常有的统计类需求，我的做法是先用HQL做筛选逻辑，ES拿到数据再进行聚合统计，如每天、每月、某人的数据。
结合ES其实更多是因为需求方喜欢Kibana的图表。

该脚本也可作为hive数据同步到Elasticsearch的工具用，不经HQL筛选数据直接进Elasticsearch
力求简洁的配置，方便使用

<br>
脚本使用说明<br>

环境: Python2 Python3 <br>
命令 #python hive_to_es.py config=<配置文件目录><br>


配置文件使用说明： 使用.ini后缀的配置文件<br>

```ini
;Elasticsearch地址、用户名、密码
[es]
hosts = 192.168.3.100:9200
username = elastic
password = 888888

;默认存入的es的index，若导出表没配置es_index，默认存入该index
default_index = tqc_ttt

;Hive地址、端口、数据库名、用户等配置
[hive]
host = 127.0.0.1
port = 10000
user = hiveuser
auth_mechanism = PLAIN
database = dbname


;需要导到ES的各个表的名称，同时也是导到ES的默认type名；
;如果是通过HQL筛选出结果集合再导入ES，结果表名称可自定义，且必须再下面给出HQL文件路径的配置
[table]
tables = student,score,teacher,my_result_a,my_result_b

[my_result_a]
;HQL文件位置，目前还不支持带有注释的HQL
sql_path = ./sql/hql_test1.sql

;再定义另一想要导出到ES的结果集合
[my_result_b]
sql_path = ./sql/hql_test2.sql

# 如需要对结果集作出更多配置，可进行如下操作

;配置头为对应要导出的表的名称
;[student]

;若不使用默认index，则配置此目标index
;es_index = tqc_test
;若不使用默认type，则配置此目标type；默认type与表名一致
;es_type = tqc_test_type

;字段名映射，这里hive表中的name字段映射为ES中的name_in_es，sex字段映射为ES中的sex_in_es...
;column_mapping = name=name_in_es,sex=sex_in_es

;限定导出的字段
;columns = name,address,sex

;通过编写HQL获得结果集导入ES时的HQL文件路径
;sql_path = ./sql/hql_test1.sql

;分页查询配置，为了防止一次查询出所有数据，导致结果集过大，导致查询时内存吃不消，无分页配置时默认分页大小30000
;page_size = 1000
;导入数据前是否清空该type下所有数据，相当于全量导入结果集，默认=false，清空原有type中数据，全量导入ES。
;overwrite = false


```

**TODO 引入impala渠道导数据，提升查询速度**