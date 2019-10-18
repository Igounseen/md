# InfluxDB

## Linux 上安装

#### Centos7 上安装

配置yum源

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```

安装并启动服务

```bash
sudo yum install influxdb
sudo service influxdb start
```

#### Ubuntu上安装

配置repo

```bash
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

安装并启动服务

```bash
sudo apt-get update && sudo apt-get install influxdb
sudo service influxdb start
```

#### Docker上安装

```bash
docker run -d -p 8083:8083 -p8086:8086 --expose 8090 --expose 8099 --name influxDbService influxdb
```

> -d：deamon，后台启动
> -p：port, 端口映射，宿主机端口:容器内端口；8083是influxdb的web管理工具端口，8086是influxdb的HTTP API端口
> --expose：允许容器接受外部传入的数据
> --name：容器名称，此处为influxDbService
> influxdb：镜像名



##  使用Cli（Shell）

进入容器：

```bash
docker container exec -it influxDbService  /bin/bash
```

进入Cli:


```bash
 influx -precision rfc3339 
```

>-precision:  指定任何返回时间戳的格式/精度



## 创建数据库

创建数据库：

```sql
CREATE DATABASE mydb
```

查询已有的数据库：

```sql
SHOW DATABASES
```



## 数据基本读写

#### 写入数据

```sql
 > INSERT cpu,host=serverA,region=us_west value=0.64
 
 > INSERT stock,symbol=AAPL bid=127.46,ask=127.48
 
 > INSERT payment,device=mobile,product=Notepad,method=credit billed=33,licenses=3i 1434067467100293230
 
 > INSERT temperature,machine=unit42,type=assembly external=25,internal=37 1434067467000000000
 
```

#### 读取数据

```sql
> SELECT "host", "region", "value" FROM "cpu"
name: cpu
time                         host    region  value
----                         ----    ------  -----
2019-09-24T06:08:37.9181655Z serverA us_west 0.64
```

```sql
> select * from payment
name: payment
time                          billed device licenses method product
----                          ------ ------ -------- ------ -------
2015-06-12T00:04:27.10029323Z 33     mobile 3        credit Notepad
```



## 导入外部数据

数据来源：

```bash
curl https://s3.amazonaws.com/noaa.water-database/NOAA_data.txt -o NOAA_data.txt
```

数据样例：

```
# DDL
CREATE DATABASE NOAA_water_database
# DML
# CONTEXT-DATABASE: NOAA_water_database

h2o_feet,location=coyote_creek water_level=8.120,level\ description="between 6 and 9 feet" 1439856000
h2o_feet,location=coyote_creek water_level=8.005,level\ description="between 6 and 9 feet" 1439856360
h2o_feet,location=coyote_creek water_level=7.887,level\ description="between 6 and 9 feet" 1439856720
```

导入：

```bash
influx -import -path=NOAA_data.txt -precision=s -database=NOAA_water_database	
```

导入结果：

```
> show measurements
name: measurements
name
----
average_temperature
h2o_feet
h2o_pH
h2o_quality
h2o_temperature
```

>influxdb 中的 measurement 关键字相当于mysql 中的 table



## 关键概念

数据表例: **census**

| time                 | **butterflies** | **honeybees** | **location** | **scientist** |
| -------------------- | --------------- | ------------- | ------------ | ------------- |
| 2015-08-18T00:00:00Z | 12              | 23            | 1            | langstroth    |
| 2015-08-18T00:00:00Z | 1               | 30            | 1            | perpetua      |
| 2015-08-18T00:06:00Z | 11              | 28            | 1            | langstroth    |
| 2015-08-18T00:06:00Z | 3               | 28            | 1            | perpetua      |
| 2015-08-18T05:54:00Z | 2               | 11            | 2            | langstroth    |
| 2015-08-18T06:00:00Z | 1               | 10            | 2            | langstroth    |
| 2015-08-18T06:06:00Z | 8               | 23            | 2            | perpetua      |
| 2015-08-18T06:12:00Z | 7               | 22            | 2            | perpetua      |

>**measurement**  : 	是一个容器，包含了列time，field和tag。概念上类似表。例：census
>
>**timestamp** :			时间戳。以`RFC3339`格式展示的相关联的UTC日期和时间。例：2015-08-18T00:00:00Z
>
>**field key** ： 			 在InfluxDB中不能没有field，field 没有索引 。例: honeybees, butterflies
>
>**field value** ：          各个field key 的值。 例：12， 1， 11， 3，....
>
>**tag key** : 		          在InfluxDB中可以没有tag，tag是索引起来的。例：location, scientist
>
>**tag value ** : 			 各个tag key 的值。 例： 23，30，28，...
>
>**retention policy** :  描述infloxdb保留数据的时间。 measurement默认永久保存。
>
>**point** :                      表示一个简单数记录，类似与sql数据库中的行。	                                                                    



## 查询样例

1 查询表 h2o_feet 的5行数据：

```sql
> select * from h2o_feet limit 5;
name: h2o_feet
time                 level description    location     water_level
----                 -----------------    --------     -----------
2015-08-18T00:00:00Z below 3 feet         santa_monica 2.064
2015-08-18T00:00:00Z between 6 and 9 feet coyote_creek 8.12
2015-08-18T00:06:00Z below 3 feet         santa_monica 2.116
2015-08-18T00:06:00Z between 6 and 9 feet coyote_creek 8.005
2015-08-18T00:12:00Z below 3 feet         santa_monica 2.028
```

2 查询表 h2o_feet 中 location 不是 santa_monica 并且water_level 在（-0.59，9.95）间的数据：

```sql
> SELECT "water_level" FROM "h2o_feet" WHERE "location" <> 'santa_monica' AND (water_level < -0.59 OR water_level > 9.95)
name: h2o_feet
time                 water_level
----                 -----------
2015-08-29T07:18:00Z 9.957
2015-08-29T07:24:00Z 9.964
2015-08-29T07:30:00Z 9.954
2015-08-29T14:30:00Z -0.61
2015-08-29T14:36:00Z -0.591
2015-08-30T15:18:00Z -0.594
```

3  查询表h2o_feet 前1469天到当前的所有记录

```sql
>  SELECT * FROM "h2o_feet" WHERE time > now() - 1469d
name: h2o_feet
time                 level description    location     water_level
----                 -----------------    --------     -----------
2015-09-18T06:54:00Z below 3 feet         coyote_creek 2.52
2015-09-18T06:54:00Z between 3 and 6 feet santa_monica 4.131
2015-09-18T07:00:00Z below 3 feet         coyote_creek 2.641
2015-09-18T07:00:00Z between 3 and 6 feet santa_monica 4.137
2015-09-18T07:06:00Z below 3 feet         coyote_creek 2.769
```
4  以location 分组，计算每组water_level 的均值：

```sql
> SELECT MEAN("water_level") FROM "h2o_feet" GROUP BY "location"
name: h2o_feet
tags: location=coyote_creek
time                 mean
----                 ----
1970-01-01T00:00:00Z 5.359342451341401

name: h2o_feet
tags: location=santa_monica
time                 mean
----                 ----
1970-01-01T00:00:00Z 3.530863470081006
```

5  每12分钟为一组，聚合查询一个时间段内各location 的water_level 统计次数：

```sql
> SELECT COUNT("water_level") FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' GROUP BY time(12m),"location"
name: h2o_feet
tags: location=coyote_creek
time                 count
----                 -----
2015-08-18T00:00:00Z 2
2015-08-18T00:12:00Z 2
2015-08-18T00:24:00Z 2

name: h2o_feet
tags: location=santa_monica
time                 count
----                 -----
2015-08-18T00:00:00Z 2
2015-08-18T00:12:00Z 2
2015-08-18T00:24:00Z 2
```



## HTTP endpoints

| endpoint        | m描述                    |
| --------------- | ------------------------ |
| /write          | 写入数据                 |
| /query          | 查询和管理数据           |
| /debug/requests | 显示最近的写入和查询次数 |
| /ping           | 检查influxdb的版本和状态 |
| /debug/vars     | 运行时的统计信息         |

####  POST 将数据写入表

```bash
curl -i -XPOST "http://localhost:8086/write?db=mydb&precision=s" --data-binary 'stock,symbol=AAPE ask=128.34,bid=128.77  1463683075'
```
Response: 
```http
HTTP/1.1 204 No Content
Content-Type: application/json
Request-Id: f545baf9-e034-11e9-805c-0242ac110002
X-Influxdb-Build: OSS
X-Influxdb-Version: 1.7.8
X-Request-Id: f545baf9-e034-11e9-805c-0242ac110002
Date: Thu, 26 Sep 2019 08:09:26 GMT
```

####  GET  查询库mydb中表stock的所有数据

```shell
curl -G http://localhost:8086/query?db=mydb --data-urlencode "q=SELECT * FROM stock"
```
Response:
```json
{"results":[{"statement_id":0,"series":[{"name":"stock","columns":["time","ask","bid","symbol"],"values":[["2016-05-19T18:37:55Z",128.34,128.77,"AAPE"],["2016-05-20T22:24:35Z",128.34,128.77,"AAPE"],["2019-09-24T06:12:38.861384Z",127.48,127.46,"AAPL"]]}]}]}
```

####  查询最近60s的读写情况

```bash
curl http://localhost:8086/debug/requests?seconds=60
```

Response:

```json
{
"172.17.0.1": {"writes":0,"queries":4}
}
```

####  查询influxdb的状态

```bash
 curl -sl -I http://localhost:8086/ping
```

Response:

```http
HTTP/1.1 204 No Content
Content-Type: application/json
Request-Id: 0035ea28-e0c5-11e9-8003-0242ac110002
X-Influxdb-Build: OSS
X-Influxdb-Version: 1.7.8
X-Request-Id: 0035ea28-e0c5-11e9-8003-0242ac110002
Date: Fri, 27 Sep 2019 01:20:32 GMT
```

####  查询influxdb运行时的统计信息

```bash
curl http://localhost:8086/debug/vars
```



## Java API

[参考](<https://github.com/influxdata/influxdb-java>)

#### 查询

```java
public class QueryTest {
    public static void main(String[] args) {
        InfluxDB influxDB = InfluxDBFactory.connect("http://127.0.0.1:8086", "root", "");
        String dbName = "NOAA_water_database";
        influxDB.setDatabase(dbName);
        QueryResult result = influxDB.query(new Query("select mean(water_level) from h2o_feet group by location"));
        List<QueryResult.Result> results = result.getResults();
        for (QueryResult.Result res : results) {
            List<QueryResult.Series> series = res.getSeries();
            for (QueryResult.Series s: series) {
                System.out.println(s);
            }
        }
        influxDB.close();
    }
}
```

相关结果：

```
Series [name=h2o_feet, tags={location=coyote_creek}, columns=[time, mean], values=[[1970-01-01T00:00:00Z, 5.359342451341401]]]
Series [name=h2o_feet, tags={location=santa_monica}, columns=[time, mean], values=[[1970-01-01T00:00:00Z, 3.530863470081006]]]
```

#### 写入

```java
public class WriteTest {
    public static void main(String[] args) {
        InfluxDB influxDB = InfluxDBFactory.connect("http://127.0.0.1:8086", "root", "");
        String dbName = "mydb";
        influxDB.setDatabase(dbName);
        Point point = Point.measurement("stock")
                .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
                .tag("symbol", "AAPL")
                .addField("ask", 314.23)
                .addField("bid", 123.12)
                .build();
        influxDB.write(point);
        influxDB.close();
    }
}
```



## 日志

配置日志输出文件：/etc/default/influxdb

```shell
STDOUT=/var/log/influxdb/influxd.log 
STDERR=/dev/null
```

http 访问日志

默认情况下，http日志与系统日志混在一起，可以分开配置。修改配置文件 /etc/influxdb/influxdb.conf

```bash
[http]
    # 配置http日志路径
	access-log-path = "/var/log/influxdb/http.log"
```



##  数据的备份和恢复

//TODO



## 可视化工具

[Grafana](<https://grafana.com/docs/features/datasources/influxdb/>)



## 性能测试

#### 数据特征

- 单行有9列，其中1个time列，4各tag列；
- 数据中共有9个PE，每个PE各有3个端口；
- 数据时间跨度为2019-09-27T08:10:40Z 到 2024-06-28T10:50:30Z，总量为1千5百万。

#### 数据导入

1 秒导入的外部数据量(PPS)约为： >3w  。导入1千5百万条总花费时长7分30秒。

```bash
2019/09/27 16:21:11 Processed 14000000 lines.  Time elapsed: 7m0.2640446s.  Points per second (PPS): 33312
2019/09/27 16:21:15 Processed 14100000 lines.  Time elapsed: 7m3.4700159s.  Points per second (PPS): 33296
2019/09/27 16:21:18 Processed 14200000 lines.  Time elapsed: 7m6.793429s.  Points per second (PPS): 33271
2019/09/27 16:21:20 Processed 14300000 lines.  Time elapsed: 7m9.3614129s.  Points per second (PPS): 33305
2019/09/27 16:21:24 Processed 14400000 lines.  Time elapsed: 7m12.5565537s.  Points per second (PPS): 33290
2019/09/27 16:21:27 Processed 14500000 lines.  Time elapsed: 7m15.5405537s.  Points per second (PPS): 33291
2019/09/27 16:21:29 Processed 14600000 lines.  Time elapsed: 7m18.3886669s.  Points per second (PPS): 33303
2019/09/27 16:21:32 Processed 14700000 lines.  Time elapsed: 7m21.450517s.  Points per second (PPS): 33299
2019/09/27 16:21:36 Processed 14800000 lines.  Time elapsed: 7m24.7644169s.  Points per second (PPS): 33276
2019/09/27 16:21:39 Processed 14900000 lines.  Time elapsed: 7m27.7159335s.  Points per second (PPS): 33280
2019/09/27 16:21:42 Processed 15000000 lines.  Time elapsed: 7m30.505427s.  Points per second (PPS): 33295
2019/09/27 16:21:42 Processed 1 commands
2019/09/27 16:21:42 Processed 15000000 inserts
2019/09/27 16:21:42 Failed 0 inserts

```

表的数据总量：

```sql
> select count(*) from statistics
name: statistics
time count_rxbps count_rxpps count_txbps count_txpps
---- ----------- ----------- ----------- -----------
0    15000000    15000000    15000000    15000000

```

表的数据样例：

```sql
> select * from statistics limit 3
name: statistics
time                desc_interface       desc_pe rxbps       rxpps     src_interface        src_pe txbps       txpps
----                --------------       ------- -----       -----     -------------        ------ -----       -----
1569571840000000000 GigabitEthernet3/0/2 PE9     0.38911033  1.9797122 GigabitEthernet3/0/2 PE6    0.38911033  1.9797122
1569571850000000000 GigabitEthernet3/0/2 PE2     0.029660702 1.1622765 GigabitEthernet3/0/2 PE3    0.029660702 1.1622765
1569571860000000000 GigabitEthernet3/0/0 PE3     0.7706396   1.3228071 GigabitEthernet3/0/0 PE7    0.7706396   1.3228071
```



#### 聚合查询性能

1 查询 从PE3到PE9 在 2019-09-27 至 2024-06-28 日之间已130天为一组的rxbps均值。TimeCost: 390ms

```bash
> select mean(rxbps),mean(txbps),mean(rxpps),mean(txpps) from statistics where "src_pe" = 'PE1' and "desc_pe" = 'PE3' and time > '2019-09-27T08:10:40Z' and time < '2024-06-28T10:50:30Z' group by time(130d)
name: statistics
time                 mean                mean_1              mean_2             mean_3
----                 ----                ------              ------             ------
2019-06-23T00:00:00Z 0.49985659845703856 0.49985659845703856 1.5087383645185557 1.5087383645185557
2019-10-31T00:00:00Z 0.4981432853717296  0.4981432853717296  1.4989413382869783 1.4989413382869783
2020-03-09T00:00:00Z 0.5018830508307139  0.5018830508307139  1.5006637754412049 1.5006637754412049
2020-07-17T00:00:00Z 0.4994071772885532  0.4994071772885532  1.5017010598345006 1.5017010598345006
2020-11-24T00:00:00Z 0.4997291149222085  0.4997291149222085  1.4999344556048835 1.4999344556048835
2021-04-03T00:00:00Z 0.5012818284335312  0.5012818284335312  1.4975732364618157 1.4975732364618157
2021-08-11T00:00:00Z 0.5006100803056361  0.5006100803056361  1.497144697552581  1.497144697552581
...

```

2  全表已src_pe 和dest_pe 做聚合。 TimeCost:23s

```sql
> select mean(rxbps),mean(txbps),mean(rxpps),mean(txpps) from statistics where time > '2019-09-27T08:10:40Z' and time < '2024-06-28T10:50:30Z' group by time(150d),src_pe,desc_pe

name: statistics
tags: desc_pe=PE2, src_pe=PE1
time                 mean                mean_1              mean_2             mean_3
----                 ----                ------              ------             ------
2019-09-11T00:00:00Z 0.5014492742023874  0.5014492742023874  1.4992754726398112 1.4992754726398112
2020-02-08T00:00:00Z 0.49732351371270345 0.49732351371270345 1.5047455242870587 1.5047455242870587
2020-07-07T00:00:00Z 0.49989105670990636 0.49989105670990636 1.500591039739087  1.500591039739087
2020-12-04T00:00:00Z 0.49890493821321746 0.49890493821321746 1.5020985893686216 1.5020985893686216
....

name: statistics
tags: desc_pe=PE2, src_pe=PE3
time                 mean                mean_1              mean_2             mean_3
----                 ----                ------              ------             ------
2019-09-11T00:00:00Z 0.49986103550376004 0.49986103550376004 1.5026672888972428 1.5026672888972428
2020-02-08T00:00:00Z 0.49874006212543615 0.49874006212543615 1.498660459530955  1.498660459530955
2020-07-07T00:00:00Z 0.5007407901621606  0.5007407901621606  1.5037903671133508 1.5037903671133508
2020-12-04T00:00:00Z 0.5003577775028557  0.5003577775028557  1.4990856717820962 1.4990856717820962
...

```

3 查询所有src_pe = PE1 的 的聚合情况。    TimeCost:2508ms

```bash
> select mean(rxbps),mean(txbps),mean(rxpps),mean(txpps) from statistics where "src_pe"='PE1' and  time > '2019-09-27T08:10:40Z' and time < '2024-06-28T10:50:30Z' group by time(150d),src_pe,desc_pe


name: statistics
tags: desc_pe=PE7, src_pe=PE1
time                 mean                mean_1              mean_2             mean_3
----                 ----                ------              ------             ------
2019-09-11T00:00:00Z 0.5008025060603762  0.5008025060603762  1.5011987892633814 1.5011987892633814
2020-02-08T00:00:00Z 0.4977871879851161  0.4977871879851161  1.4991703814311947 1.4991703814311947
2020-07-07T00:00:00Z 0.49920901818730856 0.49920901818730856 1.5004719125599075 1.5004719125599075
2020-12-04T00:00:00Z 0.4959039498678229  0.4959039498678229  1.4976820554902723 1.4976820554902723
2021-05-03T00:00:00Z 0.4979796320540383  0.4979796320540383  1.498317549758266  1.498317549758266
2021-09-30T00:00:00Z 0.4971971971574478  0.4971971971574478  1.5035878230037811 1.5035878230037811
2022-02-27T00:00:00Z 0.4998497940623678  0.4998497940623678  1.5033950087120362 1.5033950087120362
2022-07-27T00:00:00Z 0.5016586592852955  0.5016586592852955  1.5024103218985796 1.5024103218985796
2022-12-24T00:00:00Z 0.5015504309956803  0.5015504309956803  1.5026238993052465 1.5026238993052465
2023-05-23T00:00:00Z 0.5018607831898219  0.5018607831898219  1.5029084591845758 1.5029084591845758
2023-10-20T00:00:00Z 0.49760529257729375 0.49760529257729375 1.4951298651649676 1.4951298651649676
2024-03-18T00:00:00Z 0.49940482820729826 0.49940482820729826 1.4992534534775888 1.4992534534775888

name: statistics
tags: desc_pe=PE8, src_pe=PE1
time                 mean                mean_1              mean_2             mean_3
----                 ----                ------              ------             ------
2019-09-11T00:00:00Z 0.500086091425502   0.500086091425502   1.4994384408933    1.4994384408933
2020-02-08T00:00:00Z 0.4987737219683152  0.4987737219683152  1.5008715776125219 1.5008715776125219
2020-07-07T00:00:00Z 0.4969123869316141  0.4969123869316141  1.501812173029646  1.501812173029646
2020-12-04T00:00:00Z 0.4996774135467364  0.4996774135467364  1.5017176515186592 1.5017176515186592
2021-05-03T00:00:00Z 0.5041683520025542  0.5041683520025542  1.5034453854321408 1.5034453854321408
2021-09-30T00:00:00Z 0.5017972350598597  0.5017972350598597  1.500228396630465  1.500228396630465
2022-02-27T00:00:00Z 0.49979503265512737 0.49979503265512737 1.4986825584168837 1.4986825584168837
2022-07-27T00:00:00Z 0.5005004443471129  0.5005004443471129  1.4985617421664827 1.4985617421664827
2022-12-24T00:00:00Z 0.497631457526195   0.497631457526195   1.4975610171017553 1.4975610171017553
2023-05-23T00:00:00Z 0.4980566736833629  0.4980566736833629  1.4959507482689074 1.4959507482689074
2023-10-20T00:00:00Z 0.5008333952070481  0.5008333952070481  1.4982229180561772 1.4982229180561772
2024-03-18T00:00:00Z 0.5056742070416563  0.5056742070416563  1.501268259456371  1.501268259456371


```

## 认证和授权

1. 创建admin用户

   ```sql
   CREATE USER paul WITH PASSWORD 'admin' WITH ALL PRIVILEGES
   ```

2. 修改配置文件 /etc/influxdb/influxdb.conf

   ```
   [http]
   	 auth-enabled = true
   ```

3. 重启服务

   ```bash
   service influxdb restart	
   ```

4. 使用认证

- curl 

   ```bash
   curl -G http://localhost:8086/query -u admin:admin --data-urlencode "q=SHOW DATABASES"
   ```

   ```bash
   curl -G "http://localhost:8086/query?u=admin&p=admin" --data-urlencode "q=SHOW DATABASES"
   ```

- cli

   ```bash
   export INFLUX_USERNAME=admin
   export INFLUX_PASSWORD=admin
   echo $INFLUX_USERNAME $INFLUX_PASSWORD
   admin admin
   
   influx
   Connected to http://localhost:8086 version 1.4.x
   InfluxDB shell 1.4.x
   ```

   ```bash
   influx -username admin -password admin
   Connected to http://localhost:8086 version 1.4.x
   InfluxDB shell 1.4.x	
   ```

   

```bash

```

