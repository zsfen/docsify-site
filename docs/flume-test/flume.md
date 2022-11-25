# **flume代码介绍**
[flume文档](https://flume.liyifeng.org/ "flume文档")
## **flume原理**
- 如图所示，Event是Flume定义的一个数据流传输的最小单元。Agent就是一个Flume的实例，本质是一个JVM进程，该JVM进程控制Event数据流从外部日志生产者那里传输到目的地（或者是下一个Agent）;

![flume流程图](./flume-process.jpg "flume流程图")
- Event在Flume里表示数据传输的一个最小单位（被Flume收集的一条条日志又或者一个个的二进制文件，不管你在外面叫什么，进入Flume之后它就叫event）。参照下图可以看得出Agent就是Flume的一个部署实例， 一个完整的Agent中包含了必须的三个组件Source、Channel和Sink，Source是指数据的来源和方式，Channel是一个数据的缓冲池，Sink定义了数据输出的方式和目的地。
- Source获取由外部（如Web服务器）传递给它的Event。外部以Flume Source识别的格式向Flume发送Event。
- 当Source接收Event时，它将其存储到一个或多个channel。该channel是一个被动存储器（或者说叫存储池），可以存储Event直到它被Sink消耗。Agent中的source和sink与channel存取Event是异步的。Flume的Source负责消费外部传递给它的数据（比如web服务器的日志）。外部的数据生产方以Flume Source识别的格式向Flume发送Event.

## **配置详解**
- ### 配置文件参数
- **文件基本参数**
- 定义flume需要配置source、sink和channel各个组件的属性。配置的方式是以相同的分级命名空间的方式，你可以设置各个组件的类型以及基于其类型特有的属性

```
# properties for sources
<Agent>.sources.<Source>.<someProperty> = <someValue>

# properties for channels
<Agent>.channel.<Channel>.<someProperty> = <someValue>

# properties for sinks
<Agent>.sources.<Sink>.<someProperty> = <someValue>
```
- **样例**

```
# example.conf: 一个单节点的 Flume 实例配置

# 配置Agent a1各个组件的名称
a1.sources = r1    #Agent a1 的source有一个，叫做r1
a1.sinks = k1      #Agent a1 的sink也有一个，叫做k1
a1.channels = c1   #Agent a1 的channel有一个，叫做c1

# 配置Agent a1的source r1的属性
a1.sources.r1.type = netcat       #数据来源，这里定义的是，没有别名填全限定类名
a1.sources.r1.bind = localhost    #NetCat TCP Source监听的hostname，这个是本机
a1.sources.r1.port = 44444        #监听的端口

# 配置Agent a1的sink k1的属性
a1.sinks.k1.type = logger         # sink使用的是Logger ,数据传出的接收者，这里定义为loger,还可以是HDFS,hive,file

# 配置Agent a1的channel c1的属性，channel是用来缓冲Event数据的
a1.channels.c1.type = memory           #channel的类型是内存channel，顾名思义这个channel是使用内存来缓冲数据
a1.channels.c1.capacity = 1000         #内存channel的容量大小是1000，注意这个容量不是越大越好，配置越大一旦Flume挂掉丢失的event也就越多
a1.channels.c1.transactionCapacity = 100    #source和sink从内存channel每次事务传输的event数量

# 把source和sink绑定到channel上
a1.sources.r1.channels = c1       #与source r1绑定的channel有一个，叫做c1
a1.sinks.k1.channel = c1          #与sink k1绑定的channel有一个，叫做c1
```
- 在一个配置文件里可以有设置多个不同的组件，每个组件都应该有一个 type 属性，这样Flume才能知道它是什么类型的组件。每个组件类型都有它自己的一些属性。所有的这些都是根据需要进行配置。
- 上述配置文件中定义了一个Agent叫做a1，a1有一个source监听本机44444端口上接收到的数据、一个缓冲数据的channel还有一个把Event数据输出到控制台的sink。这个配置文件给各个组件命名，并且设置了它们的类型和其他属性。通常一个配置文件里面可能有多个Agent，当启动Flume时候通常会传一个Agent名字来做为程序运行的标记。

- ### 启动命令

```
bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console 
这里就是通过name指定了组件a1进行启动，

```


## **代码详解**
> flume逻辑是 获取kafka作为source,文件file作为最终的输出，中间对数据进行了处理，包括上传到FTP、写入到mongo。

- ### kafkaSource--Sink
- #### （1).继承kafkaSource,覆盖源码配置

>重新加载配置文件里kafka相关配置,这里configure方法是配置相关的方法

```java
public class testSource extends KafkaSource {
    @Override
	
    public synchronized void configure(Context context) {
        String apiConfig = context.getString("api", "test");
        
        super.configure(context);
    }
}

```

- #### （2).Sink继承AbstractSink
> 负责读取channel里的event数据，存储为CSV格式文件，并上传到FTP上。

**类包含为 configure(Context)、start（）、process()方法**
- configure(Context)
> 加载配置文件的内容，初始化配置，如下代码所示

```java
    @Override
    public void configure(Context context) {
        String api = context.getString("api", "apis");
        
```
- start
> 定义数据读取的时间间隔、方式和写入地址等

```java
   @Override
    public void start() {
        loggers.info("Start {}...", this);
        this.sinkCounter.start();
        super.start();
        this.pathController.setBaseDirectory(this.directory);
       
            this.rollService.scheduleAtFixedRate(new Runnable() {
			}
```

- process方法
> 该方法就是对Channel数据进行读取处理

```java
    public Status process() throws EventDeliveryException {
        if (this.shouldRotate) {
            logger.debug("time to rotate {}", this.pathController.getCurrentFile());
           
```
s







