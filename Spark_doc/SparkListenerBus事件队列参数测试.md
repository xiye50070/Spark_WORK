---
title: SparkListenerBus事件队列参数测试 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


一、本次测试背景
1、测试参数：
spark.scheduler.listenerbus.eventqueue.size
2、参数介绍：
此参数用于修改SparkListenerBus中，阻塞队列eventQueue的大小，默认为10000。在Spark早期版本中属于静态属性，固定位10000，这导致队列满时，只得移除一些最老的事件，最终导致各种问题与bug

二、测试过程
1、测试代码：
（1）估值项目中，中小企业过户业务中，股份转让系统库业务（离线）
（2）wordCountOnline（实时批处理）
2、测试数据：
离线：hdfs://192.168.102.120:8020/yss/guzhi/interface/20181215/bjsjg  大小：2.29k
实时：http://192.168.102.120:50070/explorer.html#/yss/aml/basicTable/20190103  大小：325.41m
2、执行模式：standalone，Yarn-cluster，Yarn-client  
3、测试过程：
离线 ，参数设置为1，三种模式分别执行10次
实时， 参数设置为1，三种模式分别执行10次

三、测试结果
standalone,Yarn-cluster,Yarn-client模式：
三种模式结果相同
离线：10次报错相同，其中3次数据库写入失败，7次数据库写入成功
报错为：
[ERROR] 2019-01-18 13:56:02,216 method:org.apache.spark.internal.Logging$class.logError(Logging.scala:70) Dropping event from queue appStatus. This likely means one of the listeners is too slow and cannot keep up with the rate at which tasks are being started by the scheduler.
[ERROR] 2019-01-18 13:56:02,218 method:org.apache.spark.internal.Logging$class.logError(Logging.scala:70) Dropping event from queue executorManagement. This likely means one of the listeners is too slow and cannot keep up with the rate at which tasks are being started by the scheduler.
Exception in thread "main" org.apache.spark.sql.AnalysisException: Path does not exist: hdfs://192.168.102.120:8020/yss/guzhi/interface/20181215/bjsjg;
	at org.apache.spark.sql.execution.datasources.DataSource$.org$apache$spark$sql$execution$datasources$DataSource$$checkAndGlobPathIfNecessary(DataSource.scala:715)
	at org.apache.spark.sql.execution.datasources.DataSource$$anonfun$15.apply(DataSource.scala:389)
	at org.apache.spark.sql.execution.datasources.DataSource$$anonfun$15.apply(DataSource.scala:389)
	at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
	at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
	at scala.collection.immutable.List.foreach(List.scala:381)
	at scala.collection.TraversableLike$class.flatMap(TraversableLike.scala:241)
	at scala.collection.immutable.List.flatMap(List.scala:344)
	at org.apache.spark.sql.execution.datasources.DataSource.resolveRelation(DataSource.scala:388)
	at org.apache.spark.sql.DataFrameReader.loadV1Source(DataFrameReader.scala:239)
	at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:227)
	at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:174)
	at com.yss.scala.util.BasicUtils$.readCSV(BasicUtils.scala:58)
	at com.yss.scala.core.BJSJG$.main(BJSJG.scala:28)
	at com.yss.scala.core.BJSJG.main(BJSJG.scala)
[ERROR] 2019-01-18 13:56:14,671 method:org.apache.spark.internal.Logging$class.logError(Logging.scala:91) Listener AppStatusListener threw an exception
java.lang.NullPointerException
	at org.apache.spark.status.AppStatusListener.onApplicationEnd(AppStatusListener.scala:157)
	at org.apache.spark.scheduler.SparkListenerBus$class.doPostEvent(SparkListenerBus.scala:57)
	at org.apache.spark.scheduler.AsyncEventQueue.doPostEvent(AsyncEventQueue.scala:37)
	at org.apache.spark.scheduler.AsyncEventQueue.doPostEvent(AsyncEventQueue.scala:37)
	at org.apache.spark.util.ListenerBus$class.postToAll(ListenerBus.scala:82)
	at org.apache.spark.scheduler.AsyncEventQueue.org$apache$spark$scheduler$AsyncEventQueue$$super$postToAll(AsyncEventQueue.scala:89)
	at org.apache.spark.scheduler.AsyncEventQueue$$anonfun$org$apache$spark$scheduler$AsyncEventQueue$$dispatch$1.apply(AsyncEventQueue.scala:89)
	at scala.util.DynamicVariable.withValue(DynamicVariable.scala:58)
	at org.apache.spark.scheduler.AsyncEventQueue.org$apache$spark$scheduler$AsyncEventQueue$$dispatch(AsyncEventQueue.scala:83)
	at org.apache.spark.scheduler.AsyncEventQueue$$anon$1$$anonfun$run$1.apply$mcV$sp(AsyncEventQueue.scala:79)
	at org.apache.spark.util.Utils$.tryOrStopSparkContext(Utils.scala:1319)
	at org.apache.spark.scheduler.AsyncEventQueue$$anon$1.run(AsyncEventQueue.scala:78)

实时：10次均报错，每次执行其中某一批次数据会处理失败

报错为：
[ERROR] 2019-01-18 13:59:01,783 method:org.apache.spark.internal.Logging$class.logError(Logging.scala:70) Dropping event from queue appStatus. This likely means one of the listeners is too slow and cannot keep up with the rate at which tasks are being started by the scheduler.
[ERROR] 2019-01-18 13:59:01,785 method:org.apache.spark.internal.Logging$class.logError(Logging.scala:70) Dropping event from queue executorManagement. This likely means one of the listeners is too slow and cannot keep up with the rate at which tasks are being started by the scheduler.
[ERROR] 2019-01-21 09:57:47,601 method:org.apache.spark.internal.Logging$class.logError(Logging.scala:91) Listener AppStatusListener threw an exception
java.lang.NullPointerException
	at org.spark_project.guava.base.Preconditions.checkNotNull(Preconditions.java:191)
	at org.spark_project.guava.collect.MapMakerInternalMap.putIfAbsent(MapMakerInternalMap.java:3507)
	at org.spark_project.guava.collect.Interners$WeakInterner.intern(Interners.java:85)
	at org.apache.spark.status.LiveEntityHelpers$.weakIntern(LiveEntity.scala:592)
	at org.apache.spark.status.LiveRDDDistribution.toApi(LiveEntity.scala:478)
	at org.apache.spark.status.LiveRDD$$anonfun$2.apply(LiveEntity.scala:536)
	at org.apache.spark.status.LiveRDD$$anonfun$2.apply(LiveEntity.scala:536)
	at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:234)
	at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:234)
	at scala.collection.mutable.HashMap$$anon$2$$anonfun$foreach$3.apply(HashMap.scala:108)
	at scala.collection.mutable.HashMap$$anon$2$$anonfun$foreach$3.apply(HashMap.scala:108)
	at scala.collection.mutable.HashTable$class.foreachEntry(HashTable.scala:230)
	at scala.collection.mutable.HashMap.foreachEntry(HashMap.scala:40)
	at scala.collection.mutable.HashMap$$anon$2.foreach(HashMap.scala:108)
	at scala.collection.TraversableLike$class.map(TraversableLike.scala:234)
	at scala.collection.AbstractTraversable.map(Traversable.scala:104)
	at org.apache.spark.status.LiveRDD.doUpdate(LiveEntity.scala:536)
	at org.apache.spark.status.LiveEntity.write(LiveEntity.scala:49)
	at org.apache.spark.status.AppStatusListener.org$apache$spark$status$AppStatusListener$$update(AppStatusListener.scala:816)
	at org.apache.spark.status.AppStatusListener$$anonfun$updateRDDBlock$2.apply(AppStatusListener.scala:755)
	at org.apache.spark.status.AppStatusListener$$anonfun$updateRDDBlock$2.apply(AppStatusListener.scala:695)
	at scala.Option.foreach(Option.scala:257)
	at org.apache.spark.status.AppStatusListener.updateRDDBlock(AppStatusListener.scala:695)
	at org.apache.spark.status.AppStatusListener.onBlockUpdated(AppStatusListener.scala:624)
	at org.apache.spark.scheduler.SparkListenerBus$class.doPostEvent(SparkListenerBus.scala:73)
	at org.apache.spark.scheduler.AsyncEventQueue.doPostEvent(AsyncEventQueue.scala:37)
	at org.apache.spark.scheduler.AsyncEventQueue.doPostEvent(AsyncEventQueue.scala:37)
	at org.apache.spark.util.ListenerBus$class.postToAll(ListenerBus.scala:82)
	at org.apache.spark.scheduler.AsyncEventQueue.org$apache$spark$scheduler$AsyncEventQueue$$super$postToAll(AsyncEventQueue.scala:89)
	at org.apache.spark.scheduler.AsyncEventQueue$$anonfun$org$apache$spark$scheduler$AsyncEventQueue$$dispatch$1.apply(AsyncEventQueue.scala:89)
	at scala.util.DynamicVariable.withValue(DynamicVariable.scala:58)
	at org.apache.spark.scheduler.AsyncEventQueue.org$apache$spark$scheduler$AsyncEventQueue$$dispatch(AsyncEventQueue.scala:83)
	at org.apache.spark.scheduler.AsyncEventQueue$$anon$1$$anonfun$run$1.apply$mcV$sp(AsyncEventQueue.scala:79)
	at org.apache.spark.util.Utils$.tryOrStopSparkContext(Utils.scala:1319)
	at org.apache.spark.scheduler.AsyncEventQueue$$anon$1.run(AsyncEventQueue.scala:78)

四、总结
spark.scheduler.listenerbus.eventqueue.size = 10000，此参数处理中小数据量时已足够，在处理大数据量业务时需根据情况情况调大该参数，该参数的作用为事件总线中，事件队列的大小，队列满后将已GC方式回收溢出事件，参数过小会导致频繁GC，cpu飙升，以及其他一些无法预料的问题。本次测试数据量较小，后续有大批量数据时再对此文档进行修改。

