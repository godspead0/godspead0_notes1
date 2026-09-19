大家在连接mysql的时候，启动项目，会警告你推荐使用com.mysql.cj.jdbc.Driver 而不是com.mysql.jdbc.Driver 

那么这两者到底有什么区别呢

本质区别：

com.mysql.jdbc.Driver 是 mysql-connector-java 5中的， 
com.mysql.cj.jdbc.Driver 是 mysql-connector-java 6以及以上中的

在使用com.mysql.jdbc.Driver时，配置是需要下面这样的：

driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
AI写代码
java
运行
在使用com.mysql.cj.jdbc.Driver时，则是需要下面这样的配置的：

driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&?useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
AI写代码
java
运行
注意：

需要指定时区（serverTimezone=UTC）和 使用SSL （useSSL=false）

另外还需注意：

在设定时区的时候，如果设定serverTimezone=UTC，会比中国时间早8个小时，如果在中国，可以选择Asia/Shanghai或者Asia/Hongkong，像下面这样配置：

driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?serverTimezone=Shanghai&?useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
AI写代码
java
运行
情况分析：

如果你maven使用的是6版本以及以上版本的mysql驱动：

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.16</version>
        </dependency>
AI写代码
java
运行
这是使用的是8.0.16版本的Mysql驱动，那么会报一下的错误：

Loading class 'com.mysql.jdbc.Driver'. This is deprecated. The new 
driver class is 'com.mysql.cj.jdbc.Driver'. 
The driver is automatically registered via the SPI 
and manual loading of the driver class is generally unnecessary.
AI写代码
java
运行
上面报错翻译：

正在加载类'com.mysql.jdbc.Driver'。 这已被弃用。 新的
驱动程序类是'com.mysql.cj.jdbc.Driver'。
驱动程序通过SPI自动注册
并且通常不需要手动加载驱动程序类。
AI写代码
java
运行
这时候你就要把com.mysql.jdbc.Driver 改为 com.mysql.cj.jdbc.Driver

但是你改完之后还是会报错：

WARN: Establishing SSL connection without server’s identity verification is not recommended. 
According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection 
must be established by default if explicit option isn’t set. 
For compliance with existing applications not using SSL the verifyServerCertificate property is set to ‘false’. 
You need either to explicitly disable SSL by setting useSSL=false, 
or set useSSL=true and provide truststore for server certificate verification.
AI写代码
java
运行
上面报错翻译：

警告：建议不要在没有服务器身份验证的情况下建立SSL连接。
根据MySQL 5.5.45 +，5.6.26+和5.7.6+要求SSL连接
如果未设置显式选项，则必须默认建立。
为了符合不使用SSL的现有应用程序，verifyServerCertificate属性设置为“false”。
您需要通过设置useSSL = false显式禁用SSL，
或者设置useSSL = true并为服务器证书验证提供信任库。
AI写代码
java
运行
这个时候如果不需要SSL验证，就在url后面加useSSL=false

这个时候就不会报警告了.

使用mysql  8.0.16 版本的驱动的时候解决如下报错：

java.sql.SQLException: The server time zone value '???ú±ê×??±??' is unrecognized or represents more than one time zone.
AI写代码
java
运行
这是由于数据库和系统时区差异所造成的，在jdbc连接的url后面加上serverTimezone=GMT即可解决问题，如果需要使用gmt+8时区，需要写成GMT%2B8，否则会被解析为空。

再一个解决办法就是使用低版本的MySQL jdbc驱动，5.1.28不会存在时区的问题。

————————————————
版权声明：本文为CSDN博主「DayFight_DayUp」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/a907691592/article/details/96876030