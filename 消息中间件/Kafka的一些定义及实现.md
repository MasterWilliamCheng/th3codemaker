### 一些专业术语基本定义
**Topic**：主题，相当于为某一类业务发生消息数据。\
**Broker**：相当于服务端，接收处理客户端消息请求。同一台linux机器可以通过server.properties配置多个broker，也可以在多台机器上分布多个broker，避免单机宕机造成服务不可用。\
**Partition**：分区，topic可以分成多个分区，消息分布在不同分区里，全局唯一。\
**leader副本**：领导者副本。对外提供服务，同时一个broker里可能会有不同分区的leader和follower，但是不会同时存在同一个分区的leader和follower。\
**follower副本**：追随者副本。不对外提供服务，只从leader副本拉取数据做同步工作。在leader挂掉之后可以参与选举。\
**offset**：消息位移，需要主动或者被动的提交，用于记录上一次消费位置。\
**ISR**：分区中维护了一个动态集合ISR，其中会保存leader副本以及一些在一定时间内（replica.lag.time.max.ms，默认10秒）与leader副本数据相对同步的follower副本，follower会因为这个前提动态进出ISR。\
**Consumer Group**：消费者组，consumer实例可以配置groupid，对于一个消费者组的多个consumer来说，它们会一起协调来瓜分一个主题下的所有分区（换言之，一个topic的单个分区的消息，只有一个consumer能获取到）。不同消费者组是互相隔离的，不受彼此影响。一般来说，消费者组内的消费者实例不能大于主题下的分区总数，否则会出现有的实例分配不到分区变成空闲状态。\
**Rebalance**：重平衡。消费者组内的某个实例挂掉之后，需要对剩余的consumer实例重新进行topic下的分区的分配，这个过程可能会很久，此时所有的实例都需要停止工作，类似JVM中的“STOP THE WORLD”。发生重平衡的条件有，组内消费者实例数量发生变化，订阅topic数变化，分区数发生变化。\
**Coordinator**:协调者，协调Consumer Group中的服务，包括执行Rebalance以及提供位移管理和组成员管理等


**消息流转流程图**\
![image](https://user-images.githubusercontent.com/31581862/130718033-60fad85f-3c71-4889-9d5f-c53da441ecb1.png)

### broker及leader选举机制
**broker leader选举**\
在kafka集群中，会有多个broker节点，第一个在zk上成功创建临时节点/controller的就是leader broke，其余的follower broker则会通过controller path在zk上注册watch，当leader broker退出时，所有的follower broker都通过watch感知到，它们会去竞争创建新的/controller。zk会保证只有一个broker成功创建临时节点，其他的broker会收到异常通知，只能去注册新的watch。\
个人感觉这是一种类似分布式锁+监听器的模式。

**分区leader副本选举**\
一般情况下，分区的leader副本只会从ISR中选举，按照AR集合（所有副本集合）的顺序默认第一个follower副本成为新的leader，除非配置unclean.leader.election.enable进行unclean领导者选举，也就是非同步副本参与选举。

### 关键配置信息（consumer）
**enable.auto.commit** 手动提交或者自动提交，自动提交是在调用 poll 方法时，提交上次 poll 处理过后的位移\
**auto.commit.interval.ms** 自动提交间隔\
**heartbeat.interval.ms** 心跳间隔\
**session.timeout.ms** consumer过期时间，超过这个时间没有收到请求Coordinator就会认为consumer挂了\
**max.poll.interval.ms** 两次poll的最大间隔时间

### kafka常用linux命令
linux下启动zookeeper服务和kafka，就可以进行消息的生产与消费了

```
后台启动 nohup ./bin/kafka-server-start.sh ./config/server.properties >>/usr/local/kafka/kafka_2.12-2.8.0/logs/kafka.log 2>&1 &cd ../ 
停止服务 ./kafka_2.12-2.8.0/bin/kafka-server-stop.sh
查询端口号 netstat -anlpt|grep 9092
生产者 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic
消费者 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --from-beginning
查看topics ./kafka-topics.sh --list --zookeeper localhost:2181
```

