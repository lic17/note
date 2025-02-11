我们在使用Log4j的时候，总是出现： 

```
log4j:WARN No appenders could be found for logger (org.apache.ibatis.logging.LogFactory).  
log4j:WARN Please initialize the log4j system properly.  
```

这个问题是因为我们的log4j.properties文件配置不够完整，所以我们给它配置齐了就不会再出现这个问题。 
log4j.properties不完整配置如下： 


1.在resources下新建一个 log4j.properties 文件

```
#这里控制日志输出到控制台，还是文件中（CONSOLE，可选的还有其他，如FILE等）；DEBUG是指定日志的级别，如果是文件，在对应的FILE中指定文件的目录
log4j.rootLogger=DEBUG,CONSOLE,FILE    
log4j.addivity.org.apache=true  
```

2.在maven中添加依赖
3.在Java中使用如下的代码


maven 的依赖如下：

```
<!-- Logging -->  		
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>${org.slf4j-version}</version>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>jcl-over-slf4j</artifactId>
		<version>${org.slf4j-version}</version>
		<scope>compile</scope>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-log4j12</artifactId>
		<version>${org.slf4j-version}</version>
		<scope>compile</scope>
	</dependency>
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.15</version>
		<exclusions>
			<exclusion>
				<groupId>javax.mail</groupId>
				<artifactId>mail</artifactId>
			</exclusion>
			<exclusion>
				<groupId>javax.jms</groupId>
				<artifactId>jms</artifactId>
			</exclusion>
			<exclusion>
				<groupId>com.sun.jdmk</groupId>
				<artifactId>jmxtools</artifactId>
			</exclusion>
			<exclusion>
				<groupId>com.sun.jmx</groupId>
				<artifactId>jmxri</artifactId>
			</exclusion>
		</exclusions>
		<scope>compile</scope>  
	</dependency>
```


```
log4j.rootLogger=CONSOLE,FILE  
log4j.addivity.org.apache=true  
  
# 应用于控制台  
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender  
log4j.appender.CONSOLE.Threshold=INFO  
log4j.appender.CONSOLE.Target=System.out  
log4j.appender.CONSOLE.Encoding=GBK  
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout  
log4j.appender.CONSOLE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
  
# 每天新建日志  
log4j.appender.A1=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.A1.File=C:/log4j/log  
log4j.appender.A1.Encoding=GBK  
log4j.appender.A1.Threshold=DEBUG  
log4j.appender.A1.DatePattern='.'yyyy-MM-dd  
log4j.appender.A1.layout=org.apache.log4j.PatternLayout  
log4j.appender.A1.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L : %m%n  
  
#应用于文件  
log4j.appender.FILE=org.apache.log4j.FileAppender  
log4j.appender.FILE.File=C:/log4j/file.log  
log4j.appender.FILE.Append=false  
log4j.appender.FILE.Encoding=GBK  
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout  
log4j.appender.FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
  
# 应用于文件回滚  
log4j.appender.ROLLING_FILE=org.apache.log4j.RollingFileAppender  
log4j.appender.ROLLING_FILE.Threshold=ERROR  
log4j.appender.ROLLING_FILE.File=rolling.log  
log4j.appender.ROLLING_FILE.Append=true  
log4j.appender.CONSOLE_FILE.Encoding=GBK  
log4j.appender.ROLLING_FILE.MaxFileSize=10KB  
log4j.appender.ROLLING_FILE.MaxBackupIndex=1  
log4j.appender.ROLLING_FILE.layout=org.apache.log4j.PatternLayout  
log4j.appender.ROLLING_FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
  
#自定义Appender  
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender  
log4j.appender.im.host = mail.cybercorlin.net  
log4j.appender.im.username = username  
log4j.appender.im.password = password  
log4j.appender.im.recipient = yyflyons@163.com  
log4j.appender.im.layout=org.apache.log4j.PatternLayout  
log4j.appender.im.layout.ConversionPattern =[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n  
  
#应用于socket  
log4j.appender.SOCKET=org.apache.log4j.RollingFileAppender  
log4j.appender.SOCKET.RemoteHost=localhost  
log4j.appender.SOCKET.Port=5001  
log4j.appender.SOCKET.LocationInfo=true  
# Set up for Log Facter 5  
log4j.appender.SOCKET.layout=org.apache.log4j.PatternLayout  
log4j.appender.SOCET.layout.ConversionPattern=[start]%d{DATE}[DATE]%n%p[PRIORITY]%n%x[NDC]%n%t[THREAD]%n%c[CATEGORY]%n%m[MESSAGE]%n%n  
# Log Factor 5 Appender  
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender  
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000  
  
# 发送日志给邮件  
log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender  
log4j.appender.MAIL.Threshold=FATAL  
log4j.appender.MAIL.BufferSize=10  
log4j.appender.MAIL.From=yyflyons@163.com  
log4j.appender.MAIL.SMTPHost=www.wusetu.com  
log4j.appender.MAIL.Subject=Log4J Message  
log4j.appender.MAIL.To=yyflyons@126.com  
log4j.appender.MAIL.layout=org.apache.log4j.PatternLayout  
log4j.appender.MAIL.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n
```