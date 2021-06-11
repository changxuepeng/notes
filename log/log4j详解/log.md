https://www.cnblogs.com/godtrue/p/6444158.html

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration>

    <!-- 将日志信息输出到控制台 -->
    <appender name="ConsoleAppender" class="org.apache.log4j.ConsoleAppender">
        <!-- 设置日志输出的样式 -->
        <layout class="org.apache.log4j.PatternLayout">
            <!-- 设置日志输出的格式 -->
            <param name="ConversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%-5p] [method:%l]%n%m%n%n" />
        </layout>
        <!--过滤器设置输出的级别-->
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <!-- 设置日志输出的最小级别 -->
            <param name="levelMin" value="WARN" />
            <!-- 设置日志输出的最大级别 -->
            <param name="levelMax" value="ERROR" />
            <!-- 设置日志输出的xxx，默认是false -->
            <param name="AcceptOnMatch" value="true" />
        </filter>
    </appender>

    <!-- 将日志信息输出到文件，但是当文件的大小达到某个阈值的时候，日志文件会自动回滚 -->
    <appender name="RollingFileAppender" class="org.apache.log4j.RollingFileAppender">
        <!-- 设置日志信息输出文件全路径名 -->
        <param name="File" value="D:/log4j/RollingFileAppender.log" />
        <!-- 设置是否在重新启动服务时，在原有日志的基础添加新日志 -->
        <param name="Append" value="true" />
        <!-- 设置保存备份回滚日志的最大个数 -->
        <param name="MaxBackupIndex" value="10" />
        <!-- 设置当日志文件达到此阈值的时候自动回滚，单位可以是KB，MB，GB，默认单位是KB -->
        <param name="MaxFileSize" value="10KB" />
        <!-- 设置日志输出的样式 -->
        <layout class="org.apache.log4j.PatternLayout">
            <!-- 设置日志输出的格式 -->
            <param name="ConversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%-5p] [method:%l]%n%m%n%n" />
        </layout>
    </appender>

    <!-- 将日志信息输出到文件，可以配置多久产生一个新的日志信息文件 -->
    <appender name="DailyRollingFileAppender" class="org.apache.log4j.DailyRollingFileAppender">
        <!-- 设置日志信息输出文件全路径名 -->
        <param name="File" value="D:/log4j/DailyRollingFileAppender.log" />
        <!-- 设置日志每分钟回滚一次，即产生一个新的日志文件 -->
        <param name="DatePattern" value="'.'yyyy-MM-dd-HH-mm'.log'" />
        <!-- 设置日志输出的样式 -->
        <layout class="org.apache.log4j.PatternLayout">
            <!-- 设置日志输出的格式 -->
            <param name="ConversionPattern" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%-5p] [method:%l]%n%m%n%n" />
        </layout>
    </appender>


    <!--
     注意：
     1：当additivity="false"时，root中的配置就失灵了，不遵循缺省的继承机制
     2：logger中的name非常重要，它代表记录器的包的形式，有一定的包含关系，试验表明
        2-1：当定义的logger的name同名时，只有最后的那一个才能正确的打印日志
        2-2：当对应的logger含有包含关系时，比如：name=test.log4j.test8 和 name=test.log4j.test8.UseLog4j，则2-1的情况是一样的        2-3：logger的name表示所有的包含在此名的所有记录器都遵循同样的配置，name的值中的包含关系是指记录器的名称哟！注意啦！
     3：logger中定义的level和appender中的filter定义的level的区间取交集
     4：如果appender中的filter定义的 levelMin > levelMax ，则打印不出日志信息
     -->

    <!-- 指定logger的设置，additivity指示是否遵循缺省的继承机制-->
    <logger name="test.log4j.test8.UseLog4j" additivity="false">
        <level value ="WARN"/>
        <appender-ref ref="DailyRollingFileAppender"/>
    </logger>

    <!--指定logger的设置，additivity指示是否遵循缺省的继承机制 -->
    <logger name="test.log4j.test8.UseLog4j_" additivity="false">
        <level value ="ERROR"/>
        <appender-ref ref="RollingFileAppender"/>
    </logger>

    <!-- 根logger的设置-->
    <root>
        <level value ="INFO"/>
        <appender-ref ref="ConsoleAppender"/>
        <!--<appender-ref ref="DailyRollingFileAppender"/>-->
    </root>

</log4j:configuration>
```

相关资源：

1.https://logging.apache.org/log4j/2.x/

2.http://www.tutorialspoint.com/log4j/

3.https://baike.baidu.com/item/log4j/480673?fr=aladdin

4.https://zh.wikipedia.org/wiki/Log4j

5.https://www.iteye.com/topic/378077

6.https://my.oschina.net/xianggao/blog/523401

7.http://blog.sina.com.cn/s/blog_5ed94d710101go3u.html

8.https://my.oschina.net/zjds/blog/365155

9.https://www.iteye.com/blog/willow-na-347340

10.https://www.iteye.com/blog/zhangxiang390-258455

