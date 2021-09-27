---
title: "Запуск Spark2 из Oozie"
date: 2021-09-24T17:02:00+03:00
tags:
  - spark2
  - oozie
topics:
  - hadoop
type: "post"
---
# Настройка Oozie Spark Action для Spark2
Для настройки Oozie Spark для работы с Spark2 задачами, нужно создать директорию spark2 в папке ShareLib и скопировать туда Spark2. Oozie ShareLib это папка с библиотеками которые доступы для задач на всех нодах в кластере.

1. Создать  spark2 ShareLib директрию в Oozie ShareLib и скопировать туда Spark2
```s
hdfs dfs -mkdir /user/oozie/share/lib/lib_<ts>/spark2
hdfs dfs -put /usr/hdp/current/spark2-client/jars/* /user/oozie/share/lib/lib_<ts>/spark2/
```

2. Скопировать туда же jar oozie-sharelib-spark из папки spark
```s
hdfs dfs -cp /user/oozie/share/lib/lib_<ts>/spark/oozie-sharelib-spark-*.jar /user/oozie/share/lib/lib_<ts>/spark2/
```

3. Скопировать туда же hive-site.xml
```s
hdfs dfs -put /usr/hdp/current/spark2-client/conf/hive-site.xml /user/oozie/share/lib/lib_<ts>/spark2/
```

4. Скопировать туда же библиотеки Python
```s
hdfs dfs -put /usr/hdp/current/spark2-client/python/lib/py* /user/oozie/share/lib/lib_<ts>/spark2/
```

5. Обновить ShareLib 
```s
oozie admin –sharelibupdate
```

6. Проверить что все ок
```s
oozie admin –shareliblist spark2
```
все теперь в Oozie есть Spark2

# Запуск Spark2 из Oozie 
1. Создаем структуру директорий, файлы и копируем примеры

```s
mkdir -p demo/lib
cp /opt/oozie/spark/lib/oozie-examples.jar ./demo/lib/
touch ./demo/job.properties
touch ./demo/workflow.xml
mkdir demo/input-data
cp /opt/data/data.txt ./demo/input-data/
cd demo
```

2. Создаем файл workflow.xml

```xml
<workflow-app name="project-demoapp" xmlns="uri:oozie:workflow:0.5">
	<start to="project-demoapp" />
	<action name="project-demoapp">
		<spark xmlns="uri:oozie:spark-action:0.1">
			<job-tracker>${jobTracker}</job-tracker>
			<name-node>${nameNode}</name-node>
			<prepare>
				<delete path="${nameNode}${saveDir}" />
			</prepare>
			<master>${master}</master>
			<mode>${mode}</mode>
			<name>project-demoapp</name>
			<class>org.apache.oozie.example.SparkFileCopy</class>
			<jar>${nameNode}${jobDir}/lib/oozie-examples.jar</jar>
			<arg>${nameNode}${dataFile}</arg>
			<arg>${nameNode}${saveFile}</arg>
		</spark>
		<ok to="end" />
		<error to="fail" />
	</action>
	<kill name="fail">
		<message>Statistics job failed [${wf:errorMessage(wf:lastErrorNode())}]</message>
	</kill>
	<end name="end" />
</workflow-app>
```

3. Создаем файл job.properties

```ini
nameNode=hdfs://demohdp
jobTracker=yarnRM
oozie.use.system.libpath=true
oozie.action.sharelib.for.spark=spark2
oozie.wf.application.path=/user/a.kostetskiy/oozie_spark
jobDir=/user/a.kostetskiy/oozie_spark
dataDir=/user/a.kostetskiy/oozie_spark/input-data
saveDir=/user/a.kostetskiy/oozie_spark/out
master=yarn
mode=client
```

4. Загружаем все в кластер
```s
hdfs dfs -copyFromLocal -f ./ hdfs://demohdp/user/a.kostetskiy/
```
5. проверяем и запускаем 
```s
-bash-4.2$ oozie job --dryrun -config job.properties
OK
-bash-4.2$ oozie job --run -config job.properties
job: 0000007-210923215237401-oozie-oozi-W
-bash-4.2$ 
```

6. Ждем и проверяем задачу

```s
-bash-4.2$ oozie job -info 0000007-210923215237401-oozie-oozi-W
Job ID : 0000007-210923215237401-oozie-oozi-W
------------------------------------------------------------------------------------------------------------------------------------
Workflow Name : project-demoapp
App Path      : /user/a.kostetskiy/demo
Status        : SUCCEEDED
Run           : 0
User          : a.kostetskiy
Group         : -
Created       : 2021-09-24 14:55 GMT
Started       : 2021-09-24 14:55 GMT
Last Modified : 2021-09-24 14:56 GMT
Ended         : 2021-09-24 14:56 GMT
CoordAction ID: -

Actions
------------------------------------------------------------------------------------------------------------------------------------
ID                                                                            Status    Ext ID                 Ext Status Err Code  
------------------------------------------------------------------------------------------------------------------------------------
0000007-210923215237401-oozie-oozi-W@:start:                                  OK        -                      OK         -         
------------------------------------------------------------------------------------------------------------------------------------
0000007-210923215237401-oozie-oozi-W@project-demoapp                          OK        job_1630614195653_0710 SUCCEEDED  -         
------------------------------------------------------------------------------------------------------------------------------------
0000007-210923215237401-oozie-oozi-W@end                                      OK        -                      OK         -         
------------------------------------------------------------------------------------------------------------------------------------
(base) -bash-4.2$ 
```

7. Проверяем, что файлы скопировались
```s
-bash-4.2$ hdfs dfs -ls hdfs://demohdp/user/a.kostetskiy/demo/out/
Found 2 items
-rw-r--r--   3 a.kostetskiy a.kostetskiy          0 2021-09-24 17:56 hdfs://demohdp/user/a.kostetskiy/demo/out/_SUCCESS
-rw-r--r--   3 a.kostetskiy a.kostetskiy       1410 2021-09-24 17:56 hdfs://demohdp/user/a.kostetskiy/demo/out/part-00000
-bash-4.2$ 
```

# Ошибки
## Если ошибка вида "Caused by: org.apache.spark.SparkException: Exception when registering SparkListener"
```s
Caused by: org.apache.spark.SparkException: Exception when registering SparkListener
	at org.apache.spark.SparkContext.setupAndStartListenerBus(SparkContext.scala:2371)
	at org.apache.spark.SparkContext.<init>(SparkContext.scala:554)
	at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2493)
	at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:933)
	at org.apache.spark.sql.SparkSession$Builder$$anonfun$7.apply(SparkSession.scala:924)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:924)
	at org.apache.spark.examples.SparkPi$.main(SparkPi.scala:31)
	at org.apache.spark.examples.SparkPi.main(SparkPi.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:688)
Caused by: java.lang.ClassNotFoundException: com.cloudera.spark.lineage.ClouderaNavigatorListener
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:348)
	at org.apache.spark.util.Utils$.classForName(Utils.scala:239)
	at org.apache.spark.util.Utils$$anonfun$loadExtensions$1.apply(Utils.scala:2738)
	at org.apache.spark.util.Utils$$anonfun$loadExtensions$1.apply(Utils.scala:2736)
	at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
	at scala.collection.TraversableLike$$anonfun$flatMap$1.apply(TraversableLike.scala:241)
	at scala.collection.mutable.ArraySeq.foreach(ArraySeq.scala:74)
	at scala.collection.TraversableLike$class.flatMap(TraversableLike.scala:241)
	at scala.collection.AbstractTraversable.flatMap(Traversable.scala:104)
	at org.apache.spark.util.Utils$.loadExtensions(Utils.scala:2736)
	at org.apache.spark.SparkContext$$anonfun$setupAndStartListenerBus$1.apply(SparkContext.scala:2360)
	at org.apache.spark.SparkContext$$anonfun$setupAndStartListenerBus$1.apply(SparkContext.scala:2359)
	at scala.Option.foreach(Option.scala:257)
	at org.apache.spark.SparkContext.setupAndStartListenerBus(SparkContext.scala:2359)
```

Если в логе приложения есть ошибка вида, это скорее всего значит, что в Oozie включен Spark on Yarn.  