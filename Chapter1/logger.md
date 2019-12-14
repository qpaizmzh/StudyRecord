# 好用的监控jdbc日志模块

#### 前提：

在翻修老项目的过程中，发现springboot项目配置的彩色日志模块，在显示sql语句打印的时候竟然分成了预编译的sql语句和要传入的参数，意味着每次调试sql语句，还要把这两个东西复制出来，再按照格式一个一个参数往sql语句套，这对日常调试特别不方便。后来经过高人的指点，决定把日志模块升级，达到的效果就是把sql语句完整显示出来，以后调试的时候直接复制到sql查询窗口查看就完事

#### 步骤：

- 在maven的pom.xml中，添加新的监控sql日志模块：

```xml
        <!--监控sql日志-->
        <dependency>
            <groupId>org.bgee.log4jdbc-log4j2</groupId>
            <artifactId>log4jdbc-log4j2-jdbc4.1</artifactId>
            <version>1.16</version>
        </dependency>
```

- 然后添加log4jdbc.log4j2.properties配置文件：

```properties
# If you use SLF4J. First, you need to tell log4jdbc-log4j2 that you want to use the SLF4J logger
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```

- 在applications.properties或者是applications.yml的文件中进行地址和驱动的更改：

```yaml
driver-class-name: net.sf.log4jdbc.sql.jdbcapi.DriverSpy
url: jdbc:log4jdbc:mysql://localhost:3306/database
```

之后就可以开始运行并观察日志，在对数据库进行操作的时候，就会有打印好的sql语句和表格辉在命令行窗口中打印出来，像这样：

![img](../img/logger-1)

但是在日志中也可以看的到，有很多信息是不需要的，像是resultSet这些是不需要的，为此，我们需要添加在日志文件中进行一些配置，筛选一些没用的日志打印：

- 创建logback.xml，并在logback配置文件中添加如下的配置（一些配置的细节后续再重新进行学习）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true">
    <!-- 从application.yml 中注入变量  -->
    <!-- <springProperty scope="context" name="LOG_PATH" source="log.home"/> -->
    <!-- <springProperty scope="context" name="APPDIR" source="spring.application.name"/> -->
    <property name="LOG_PATH" value="./logs"/>
    <property name="APPDIR" value="graceLogs"/>
 
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>1-%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger - %msg%n</pattern>
            <charset>GBK</charset>
        </encoder>
    </appender>
 
 
    <!-- error级别日志文件输出,按日期时间滚动记录输出 -->
    <appender name="FILEERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APPDIR}/log_error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${APPDIR}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>500MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger Line:%-3L - %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
 
    <!-- warn级别日志文件输出,按日期时间滚动记录输出 -->
    <appender name="FILEWARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APPDIR}/log_warn.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${APPDIR}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>2MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger Line:%-3L - %msg%n</pattern>
            <charset>utf-8</charset>        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
 
 
    <!-- info级别日志文件输出,按日期时间滚动记录输出 -->
    <appender name="FILEINFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${APPDIR}/log_info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${APPDIR}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>2MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger Line:%-3L - %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
 
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger Line:%-3L - %msg%n</pattern>
        </encoder>
    </appender>
 
    <!--设置为OFF,即屏蔽; 留下sqltiming作为INFO级别输出-->
    <logger name="jdbc.connection" level="OFF"/>
    <logger name="jdbc.resultset" level="OFF"/>
    <logger name="jdbc.resultsettable" level="OFF"/>
    <logger name="jdbc.audit" level="OFF"/>
    <logger name="jdbc.sqltiming" level="INFO"/>
    <logger name="jdbc.sqlonly" level="OFF"/>
 
    <!--设置日志打印级别为INFO-->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILEINFO"/>
        <appender-ref ref="FILEWARN"/>
        <appender-ref ref="FILEERROR"/>
    </root>
 
</configuration>
```

重新启动项目，就会发现打印的日志少了结果集的打印，同时也在特定的目录中记录了某些日志，并进行了分类：

![img](../img/logger-2)

![img](../img/logger-3)

#### 遇到的坑：

- 第一次添加的时候，因为留下来的是原来的配置文件，也没有修改其他的东西，也没有对特定的日志进行分类储存，于是启动项目进行测试打印日志的时候，出现了疯狂打印日志的情况，而且运行项目比没改的时候要慢了很多，一度以为是这个日志模块的问题，后来更换新的日志配置文件也不行，本来想退回原来的版本的时候，高人给我发现了问题：**log4jdbc.log4j2.properties配置文件这个配置文件并没有被读取编译到**，什么原因呢，maven配置文件限制了：

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <filtering>true</filtering>//这里开启了过滤资源文件
                <includes>
                    <include>**/*.xml</include>
                    <include>这里包含了很多的properties文件，唯独没有包含我添加的log4jdbc.log4j2.properties，所以要在这里进行一个添加
                    </include>
                    
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
```

把这个资源文件添加为包含之后，重新用mvn命令compile，然后开始启动项目：终于有上面图片的效果啦~~

