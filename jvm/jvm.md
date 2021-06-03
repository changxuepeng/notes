java -XX:+PrintCommandLineFlags

```
-XX:InitialHeapSize=131709120 -XX:MaxHeapSize=2107345920 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
```

1.PrintFlagFinal

java -XX:+PrintFlagsFinal -version



2.jps

![img](jvm.assets/20180723165900610)

![img](jvm.assets/20180723165900610)

- Bootstrap代表tomcat
- 25687 代表jps命令本身

3.jinfo

查看运行中的java实例参数,如下设置的tomcat的最大内存

jinfo -flag MaxHeapSize  3556

![img](jvm.assets/20180723170544642)

jinfo -flag UseG1GC 7208

查看垃圾回收器





