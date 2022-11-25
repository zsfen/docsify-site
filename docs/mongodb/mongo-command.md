[mongo命令](https://www.mongodb.org.cn/manual/197.html)
# 1、命令

### 1、查看MongoDB的连接信息：
```
db.serverStatus().connections
```

### 2、查看进程信息：

```
db.currentOP(<operations>)
1、operations定义为布尔值true，db.currentOp(true)--返回全部连接,**operations**定义为文档，返回符合条件的连接.包括空闲连接和系统操作
2、查看等待获取锁的会话db.currentOP({"waitingForLock" : true})
3、查看db数据库执行时间超过3秒的活动进程:db.currentOP(  {      "active" : true,      "secs_running":{"$gt":3},      "ns":/^db./  })
currentOP返回值
> 各个关键词的含义：
>opid：进程号
>active：是否活跃状态
>secs_running：操作运行秒数
>microsecs_running：操作运行微秒数
>op:操作类型，包括(insert/update/query/remove/getmore/command)
>ns：命名空间
>query：查询语句
>client：连接的客户端信息
>desc：描述信息
>threadId：线程id
>connectionId：连接id
>waitingForLock：是否等待获取锁
>lockStats.timeLockedMicros.r：持有读锁的时间(微秒)
>lockStats.timeLockedMicros.w：持有写锁的时间(微秒)
>lockStats.timeAcquiringMicros.r：请求读锁的时间(微秒)
>lockStats.timeAcquiringMicros.2：请求写锁的时间(微秒)
```
### 3、打印mongo语句进程信息

```
/data/mongodb/bin/mongo ip:port -ufamily -password  --authenticationDatabase=admin --eval "printjson(db.currentOp())" >> currentOp.txt
```
### 4、杀死正在执行的进程:
db.killOp(opid是);
--opid是currentOP返回值的opid

### 5、查看mongo操作状态：
> 是mongodb自带的状态检测工具，在命令行下使用。它会间隔固定时间获取mongodb的当前运行状态，并输出。如果你发现数据库突然变慢或者有其他问题的话，你第一手的操作就考虑采用mongostat来查看mongo的状态

```
mongostat
1、mongo命令行输入
2、linux的命令行：mongostat --host localhost:27017 -uroot -p123456 --authenticationDatabase admin
说明：
inserts/s 每秒插入次数
query/s 每秒查询次数
update/s 每秒更新次数
delete/s 每秒删除次数
getmore/s 每秒执行
getmore次数
command/s 每秒的命令数，比以上插入、查找、更新、删除的综合还多，还统计了别的命令flushs/s 每秒执行fsync将数据写入硬盘的次数。
mapped/s 所有的被mmap的数据量，单位是MB，
vsize 虚拟内存使用量，单位MB
res 物理内存使用量，单位MB
faults/s 每秒访问失败数（只有Linux有），数据被交换出物理内存，放到swap。不要超过100，否则就是机器内存太小，造成频繁swap写入。此时要升级内存或者扩展
locked % 被锁的时间百分比，尽量控制在50%以下吧
idx miss % 索引不命中所占百分比。如果太高的话就要考虑索引是不是少了
q t|r|w 当Mongodb接收到太多的命令而数据库被锁住无法执行完成，它会将命令加入队列。这一栏显示了总共、读、写3个队列的长度，都为0的话表示mongo毫无压力。高并发时，一般队列值会升高。
conn 当前连接数
time 时间戳

注：MongoDB为每一个连接创建一个线程，线程的创建与释放也会有开销，所以尽量要适当配置连接数的启动参数，
   maxIncomingConnections，阿里工程师建议在5000以下，基本满足多数场景
set: 副本集的名称
repl: 节点的复制状态
 M     ---master
 SEC     ---secondary
 REC     ---recovering
 UNK     ---unknown
 SLV     ---slave
 RTR     ---mongs process("router')
 ARB     ---arbiter
```
### 5、mongotop
> mongotop提供了一个方法，用来跟踪一个MongoDB的实例，查看哪些大量的时间花费在读取和写入数据。 mongotop提供每个集合的水平的统计数据。默认情况下，mongotop返回值的每一秒
默认情况下，mongotop每一秒刷新一次。

```
1、linux的shell: mongotop --host ip:port -u "用户名" -p "密码"  --authenticationDatabase "admin"  60

参数说明ip：必须是分片集群中主从库的IP地址
port：必须是分片集群中主从库的端口号
用户名和密码：分片集群中副本集管理员权限的账号，若没有添加任何用户的话，那么可以不用设置用户名和密码
authenticationDatabase：必须是admin库
60：表示每隔一分钟就输出一组数据，若不设置这个参数的话，那么默认是每隔一秒就输出一组数据
2、mongo命令行 mongotop  60
3、每个数据库的锁的使用情况
mongotop --locks  

输出字段说明：ns：数据库命名空间，后者结合了数据库名称和集合。
db：数据库的名称。名为 . 的数据库针对全局锁定，而非特定数据库。
total：mongod在这个命令空间上花费的总时间。
read：在这个命令空间上mongod执行读操作花费的时间。
write：在这个命名空间上mongod进行写操作花费的时间。
```

### 6、查看mongo超时语句日志：
```
tail -100000 /mongod/Tools/mongodb/log/shard/mongodb.log | grep -E '[0-9]{5,}ms$' > /tmp/test_query_mongo.txt
```
### 7、主从同步延迟时间：
```
db.printSlaveReplicationInfo()
```
### 8、查索引：
```
db.getCollection("表名").getIndexes() 查看集合的所有索引
如
db.test.getIndexes()
[
 {
 "v" : 2,          #索引版本
  "key" : {    #索引的字段及排序方向
  "_id" : 1  #根据"_id"字段升序所以
 },
"name" : "_id_",            #索引名      
"ns" : "db_msg_track.test"  #集合名
  },
getIndexKeys() ：查看索引键
db.test.getIndexKeys()

totalIndexSize()：查看集合索引的总大小(当前集合所有索引所占用的空间大小)
db.test.totalIndexSize()

getIndexSpecs()：查看集合各索引的详细信息

```

### 9、删除索引
```

dropIndex(索引名)方法用于删除指定的索引
db.test.dropIndex("_id_")


dropIndexes()方法用于删除全部的索引

db.test.dropIndexes()


```

### 10、创建索引

```
db.collection.createIndex(keys, options)
#更具体的例子如下：
（1）单字段索引
db.集合名.createIndex( {"字段名": 1 },{"name":'idx_字段名'})
 
如：db.test.createIndex({"duration":1})

1）索引命令规范这里建议是 idx_<构成索引的字段名>。
当然name只是可选项系统会默认生成索引名的
2）字段名后面 1代表升序  -1代表降序。当然对于单字段索引没有任何区别。
 
（2）组合索引
db.集合名.createIndex({"字段名1":-1,"字段名2":1},{"name":'idx_字段名1_字段名2',background:true})
 如：db.test.createIndex({"duration":1, "name": 1})
 字段名后面 1代表升序  -1代表降序
（3）后台创建索引
db.集合名.createIndex({"字段名":1},{"name":'idx_字段名',background:true})
 db.test.createIndex({"duration":1, "name": 1}, {

    "background": true

})

（4）为内嵌字段添加索引
db.集合名.createIndex({"字段名.内嵌字段名":1},{"name":'idx_字段名_内嵌字段名'})
 {"name":"zhangsan", "class":2, "duration": 988, habbit:["football","running"], family:{number:4,addr:"zhejiang"}}
 
db.test1.createIndex({"family.number":1})

（5）TTL索引
db.集合名.createIndex( { "字段名": 1 },{ "name":'idx_字段名',expireAfterSeconds: 定义的时间,background:true} )

db.test1.createIndex({createDate:1}, {expireAfterSeconds:30,background:true})

 （6）唯一索引
db.collection.createIndex( <key and index type specification>, { unique: true } )
唯一单索引
db.test1.createIndex({"name":1},{unique:true})
 唯一复合索引
db.members.createIndex( { groupNumber: 1, lastname: 1, firstname: 1 }, { unique: true } )

（7）分片数据库，建立片键值
sh.shardCollection('cmhigateway.pon_rx_s',{'date':1, 
'deviceId':1})


```

### 11、获取当前数据库的信息，比如Obj总数、数据库总大小、平均Obj大小等
```
db.stat()


"db" : "test" ,表示当前是针对"test"这个数据库的描述。想要查看其他数据库，可以先运行 use databasename(e.g  $use admiin).　
"collections" :  表示当前数据库有多少个collections.可以通过运行show collections查看当前数据库具体有哪些collection.　
"objects" : 13，表示当前数据库所有collection总共有多少行数据。显示的数据是一个估计值，并不是非常精确。　
"avgObjSize" : 36,表示每行数据是大小，也是估计值，单位是bytes　
"dataSize" : 468,表示当前数据库所有数据的总大小，不是指占有磁盘大小。单位是bytes　
"storageSize" : 13312,表示当前数据库占有磁盘大小，单位是bytes,因为mongodb有预分配空间机制，为了防止当有大量数据插入时对磁盘的压力,因此会事先多分配磁盘空间。　
"numExtents" : 3,似乎没有什么真实意义。我弄明白之后再详细补充说明。　
"indexes" : 1 ,表示system.indexes表数据行数。　
"indexSize" : 8192,表示索引占有磁盘大小。单位是bytes   
"fileSize" : 201326592，表示当前数据库预分配的文件大小，例如test.0,test.1，不包括test.ns。
```
### 12、删除表
```
db.表名.drop()
```
# mongo语句
### 插入：
```
db.collection.insert(
<document or array of documents>,
{
    writeConcern: <document>,    //可选字段
    ordered: <boolean>    //可选字段
    }
)
db 为数据库名，collection 为集合名，insert() 为插入文档命令，三者之间用连接。
document or array of documents> 参数表示可设置插入一条或多条文档。
writeConcern:<document> 参数表示自定义写出错的级别，是一种出错捕捉机制。
ordered:<boolean> 是可选的，默认为 true。
如果为 true，在数组中执行文档的有序插入，并且如果其中一个文档发生错误，MongoDB 将返回而不处理数组中的其余文档；如果为 false，则执行无序插入，若其中一个文档发生错误，则忽略错误，继续处理数组中的其余文档。

插入不指定 _id 字段的文档的代码如下：
db.表名.insert( { item : "card", qty : 15 })

插入指定 _id 字段的文档，值 _id 必须在集合中唯一，以避免重复键错误，
db.test.insert(
    { _id: 10, item: "box", qty: 20 }
)

插入的多个文档无须具有相同的字段

db.test.insert(
    [
        { _id: 11, item: "pencil", qty: 50, type: "no.2" },
        { item: "pen", qty: 20 },
        { item: "eraser", qty: 25 }
    ]
)

有序地插入多条文档的
db.test.insert([
        {_id:10, item:"pen", price:"20" },
        {_id:12, item:"redpen", price: "30" },
        {_id:11, item:"bluepen", price: "40" }
    ],
    {ordered:true}
```
### 查找
```
db.表名.find(query, projection)
query 为可选项，设置查询操作符指定查询条件；projection 也为可选项，表示指定返回的字段，如果忽略此选项则返回所有字段
等于：
db.test.find( {price : 24} )

大于（>）
db.test.find( {price : {$gt : 24}} )

小于（<）
db.test.find( {price : {$lt : 24}} )

大于等于（>=）
db.test.find( {price : {$gte : 24}} )

小于等于（<=）
db.test.find( {price : {$lte : 24}} )

不等于（!=）
db.test.find( {price : {$ne : 24}} )

与（and）
db.test.find( {name : "《MongoDB 入门教程》", price : 24} )

或（or）
db.test.find( {$or:[{name : "《MongoDB 入门教程》"},{price : 24}]} )

限制结果数
db.test.find().limit(3)

Skip() 函数用于略过指定个数的文档
db.test.find().skip(1)

sort() 函数用于对查询结果进行排序，1 是升序，-1 是降序
db.test.find().sort({"price" : 1})

只返回某一列
db.test.find().sort({"price" : 1},{"name":1})

是否存在查询：
db.test.find({"date":{$exists:true}})

模糊查询
以xxxx开头的
db.test.find({"name":/^xxxx/})
以xxxx结尾的
db.test.find({"name":/xxxx^/})
包含xxxx
db.test.find({"name":/xxxx/})
忽略大小写
db.test.find({"name":/xxxx/i})
```
### 删除
```
db.collection.remove(    <query>,    {        justOne: <boolean>, writeConcern: <document>    })

参数说明：query：必选项，是设置删除的文档的条件。
justOne：布尔型的可选项，默认为false，删除符合条件的所有文档，如果设为 true，则只删除一个文档。

writeConcem：可选项，设置抛出异常的级别。
如：
移除 title 为“MongoDB”的文档
db.test.remove({'title': 'MongoDB'})
```

### 更新

```
db.collection.update(    
	<query>, 
	<update>, 
	{       
		upsert: <boolean>,   
		multi: <boolean>,  
		writeConcern: <document>
	}
)

# 参数解释；
query : update的查询条件，类似sql update查询内where后面的。
update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
writeConcern :可选，抛出异常的级别。


1、批量更新：
db.表名.updateMany({"countycode":"eeee","CommunityInfo":{$in:["test"]}},{$set:{"countycode":"230181"}})

db.表名.update({   "date" : NumberLong(20210429)}, {    $set: {    "date" : NumberLong(202104291)   }}, {upsert:false, multi:true});
2、单条更新，  不存在插入
db.表名.update({   "date" : NumberLong(20210429)}, {    $set: {    "date" : NumberLong(202104291)   }}, {upsert:true, multi:false});

3、删除字段：
db.表名.update({"date":NumberLong(20210829)},{$unset:{'periodic_count':''}},{upsert:false, multi:true})

4、替换文档
replaceOne()
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
db.表名.replaceOne(
   { name: "abc" },
   { name: "amy", age: 34, type: 2, status: "P"}
)

```
### 聚合
```

db.collection.aggregate(pipeline, options)　　　　[
pipeline参数]　
$group : 将集合中的文档分组，可用于统计结果，$group首先将数据根据key进行分组；　　　　
$project：可以对输入文档进行添加新字段或删除现有的字段，可以自定哪些字段显示与不显示;　　　　
$match ：根据条件用于过滤数据，只输出符合条件的文档，如果放在pipeline前面，根据条件过滤数据，传输到下一个阶段管道，可以提高后续的数据处理效率。还可以放在out之前，对结果进行再一次过滤;　　　　
$redact ：字段所处的document结构的级别;　　　　
$limit ：用来限制MongoDB聚合管道返回的文档数;　　　
$skip :在聚合管道中跳过指定数量的文档，并返回余下的文档;　　　　
$unwind :将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值;　　　　
$sample :随机选择从其输入指定数量的文档。如果是大于或等于5%的collection的文档，$sample进行收集扫描，进行排序，然后选择顶部的文件。因此，$sample 在收集阶段是受排序的内存限制 语法：{ $sample: { size: <positive integer> } }　　　　
$sort :将输入文档排序后输出；　　　　
$geoNear:用于地理位置数据分析；　　　　
$out :必须为pipeline最后一个阶段管道，因为是将最后计算结果写入到指定的collection中；         　
$indexStats :返回数据集合的每个索引的使用情况；语法：{ $indexStats: { } }

1、对city分组，并求每组num的平均值
db.data.aggregate({$group:{_id:'$city',avg_num:{$avg:'$num'}}})

2、统计多组求平均
db.表名.aggregate({$group:{_id:'$city',avg_num:{$avg:'$num'},avg_age:{$avg:'$age'}}})
3、统计多组求和
db.表名.aggregate({$group:{_id:'$city',count1:{$sum:'$num'},count2:{$sum:'$age'}}})
4、过滤，求和
db.表名.aggregate([
  { $match: {"datetime":NumberLong(2021020821)},
  {$group:{_id:{"datetime": "$datetime"}, count:{$sum:"$pon"}}}
])

例子：
db表名.aggregate([{$match:{"task":NumberLong(1620720591),"date":"20210521"}},{$group:{"_id":"$td",total:{$sum:"$task"}}}])
db.表名.aggregate([{$match:{"date":NumberLong(20211201)}},{$group:{"_id":"$areacode",total:{$sum:"$pon"},total2:{$sum:"$Count"}}}])

db.表名.aggregate([{$match:{"date":NumberLong(20210430)}},{$group:{"_id":"$areacode",total:{$sum:"$pon"}}}])
db.表名.aggregate([{$match:{"date":{$lte:NumberLong(20210429),$gte:NumberLong(20210429)}}},{$group:{"_id":"$areacode",total:{$sum:"$Count"}}}])
db.表名.aggregate([{$group:{_id:{"BrasIp":"$BrasIp","countycode":"$countycode"}}}])
排序：
db.表名.aggregate([{$match:{"date_time" :{$gte:20221021,$lte:20221030}}},{$group:{_id:{"time":"$date_time", "area_code" : "$code",},count:{$sum:"$users"}}},{$sort : { "_id.date_time" : -1}}])
加上sort，注意：sort要排在group后面或者聚合函数的最后，排序才能生效
```
### 统计总量

```
1、是否存在统计：
db.表名.count({"date":{$exists:true}})
2、db.表名.count({"date":NumberLong(20210430)})
```

### 导入导出
```
导出
存储为csv 文件，指定导出列，过滤条件
./mongoexport -d cmhigateway  --port  端口  -u f用户名 -p 密码 --authenticationDatabase admin -c 表名 --type=csv -o DeviceDetailInfo_20220412.csv -f "deviceId,PPPOEUser,areacode,countycode,manufacturer,model,periodic_count,periodic_wifi_count,periodic_wifi_below_count,sub_wifi_count,wifi_radio_power,subdevbelow_count" -q '{"isWeakSignalDevice":"true","areacode":""}'


参数说明：-h:指明数据库宿主机的IP
-u:指明数据库的用户名
-p:指明数据库的密码
-d:指明数据库的名字
-c:指明collection的名字
-f:指明要导出那些列
-o:指明到要导出的文件名
-q:指明导出数据的过滤条件
 -k [ --slaveOk ] arg (=1) use secondaries for export if available, default 
                            true
使用--noHeaderLine 进行不显示字段名

--limit=10  限制导出条数

导入
csv文件导入
字段名就是文件表头名

./mongoimport -d 库名  -c old_model --type csv  --headerline --file old_model.csv --host ip地址 --port 端口 -u 用户名 -p 密码 --authenticationDatabase admin 
先删除再导入
./mongoimport -u 用户名 -p 密码 --host ip地址 --port 端口 --authenticationDatabase admin --d 库名 -c old_model  --type=csv  --headerline  --drop  --file old_model.csv
指定导入部分字段
./mongoimport -u 用户名 -p 密码 --host ip地址 --port 端口 --authenticationDatabase admin --d 库名 -c old_model  --type=csv  --headerline  --file old_model.csv 
--upsertFields uid,name,sex
导入，冲突则覆盖
./mongoimport -u 用户名 -p 密码 --host ip地址 --port 端口 --authenticationDatabase admin --d 库名 -c old_model  --type=csv  --headerline  --file old_model.csv 
--mode=upsert

导入时必须指定列名称（如果备份文件第一行是列名称，也会被当成数据导入到数据库中）
./mongoimport -u 用户名 -p 密码 --host ip地址 --port 端口 --authenticationDatabase admin --d 库名 -c old_model  --type=csv  --headerline  --file old_model.csv 
--fields=id,name

指定字段类型进行导入 (注意：csv文件第一行不要有字段名称)
./mongoimport -u 用户名 -p 密码 --host ip地址 --port 端口 --authenticationDatabase admin --d 库名 -c old_model  --type=csv  --headerline  --file old_model.csv 
--columnsHaveTypes --fields="id.int32(),name.string()"

-h(--host):指明数据库宿主机的IP
-u(--username):指明数据库的用户名
-p（--password）:指明数据库的密码
-d（--db）:指明数据库的名字
-c（--collection）:指明collection的名字
-f（--fields）:指明要导入那些列

--type:指明要导入的文件格式，默认json，（json,csv,dat,tsv）
--headerline:指明第一行是列名，不需要导入
--file：指明要导入的文件
--drop 先删除原表，再导入
 --ignoreBlanks 字段为空的不插入 
-upsertFields 部分字段导入 ，存在就更新，不存在就插入
--upsert，存在就更新，不存在就插入

--stopOnError 在出现第一个错误时停止导入，而不是继续
--jsonArray load a json array, not one item per line. Currentlylimited to 4MB.



```
# JS脚本
1.执行js脚本命令:
```
./mongo localhost:27017/db query.js> result.txt
/usr/local/mongodb4/bin/mongo -u 用户 -p 密码 --authenticationDatabase=admin localhost:20036/库名  querys1.js >r.csv
```
2.js脚本--[mongo-js脚本](https://zhangshenjia.com/it/mongodb-js/)
```
var col = [];
db.getCollection('A表').find({}).forEach(function (x) {
    var deviceId=x._id;
    col.push(deviceId);
    if(col.length>=1000){
       var cursor= db.getCollection("B表").find({"_id":{$in:col}})
       while(cursor.hasNext()){
        var obj=cursor.next();
        print(obj.deviceId);
        }
        col=[];
    }
   if (col.length > 0) {
      var cursor= db.getCollection("B表").find({"_id":{$in:col}})
       while(cursor.hasNext()){
        var obj=cursor.next();
        print(obj.deviceId+"");
        }
        col=[];
   }});
```

3. 模糊更新--js脚本
```
db.getCollection('city1').find({"code":/^2327/}).forEach(
   function(item){                
   db.getCollection('city1').update({"_id":item._id},{$set:{"parentcode":"0457"}})
   }
)
```
4.js脚本连接mongo

```
var testDB = db.getSisterDB('testSrcDB')//或者db.getSiblingDB('testSrcDB');

var result = testDB.getCollection('testSrcCollection').find({"version" : "-"});

while(result.hasNext()){

testDB.testDestCollection_20211120.insert(result.next());
var testDB = connect("192.168.1.101:27027,192.168.1.102:27017,192.168.1.103:27027/test?replicaSet=xxx").getSisterDB("test");

userDB.auth("usernmae", "passwd");

var t_user = userDB.getCollection("T_USER");
var testDB = connect("user:passwd@192.168.1.101:27027,192.168.1.102:27017,192.168.1.103:27027/test?replicaSet=xxx&authSource=test");

var t_user = userDB.getCollection("T_USER");

var ts=userDB.getCollectionNames();
```

# mongo和java连接使用问题：





