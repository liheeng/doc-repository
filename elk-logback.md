小阔学堂-代码异常自动抓取和邮件通知配置
目录
1	配置Logback
1.1	Logback.xml
1.2	Logstash配置
2	安装配置ELK
2.1	安装ELK
2.2	安装Logstash
2.3	安装配置Kibana
3	附：CentOS 7.3下Elasticsearch 2.4.1+Kibana 4.6.6+Logstash 2.4.1及插件安装配置
配置Logback
Logback.xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Define properties -->
    <property name="PROJECT" value="kxt-integration"/>
    <property name="LOG_PATH" value="./logs"/>

    <!-- 彩色日志 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <!-- 控制台输出 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}] [%le] [%lo] [%thread]: %msg '%ex'\n</pattern>
        </encoder>
    </appender>

    <!-- 系统默认日志 -->
    <!-- 时间滚动输出 level为 TRACE、DEBUG、INFO、WARN 日志 -->
    <appender name="file—default"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- deny all events with a level below TRACE and above WARN -->
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
            <evaluator class="ch.qos.logback.classic.boolex.GEventEvaluator">
                <expression>
                    e.level.toInt() &gt;= TRACE.toInt() &amp;&amp; e.level.toInt() &lt;= WARN.toInt()
                </expression>
            </evaluator>
            <OnMatch>NEUTRAL</OnMatch>
            <OnMismatch>DENY</OnMismatch>
        </filter>
        <file>${LOG_PATH}/${PROJECT}-default.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_PATH}/${PROJECT}-default.%d{yyyy-MM-dd}.log</FileNamePattern>
            <MaxHistory>10</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %le %lo [%thread]: ## '%msg' '%ex'\n</pattern>
        </encoder>
    </appender>

    <!-- 错误日志 -->
    <!-- 时间滚动输出 level为 ERROR、FAULT 日志 -->
    <appender name="file—error"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <file>${LOG_PATH}/${PROJECT}-error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_PATH}/${PROJECT}-error.%d{yyyy-MM-dd}.log</FileNamePattern>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %le %lo [%thread]: ## '%msg' '%ex'\n</pattern>
        </encoder>
    </appender>
    <!--elk登录日志-->
    <appender name="login-info"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <file>${LOG_PATH}/${PROJECT}-login-info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_PATH}/${PROJECT}-login-info.%d{yyyy-MM-dd}.log</FileNamePattern>
            <MaxHistory>10</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}] [%le] [%lo] [%thread] %msg \n</pattern>
        </encoder>
    </appender>

    <!-- 接口输入输出日志 -->
    <!-- 时间滚动输出 level为 TRACE、DEBUG、INFO、WARN 日志 -->
    <appender name="api-log"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
            <evaluator class="ch.qos.logback.classic.boolex.GEventEvaluator">
                <expression>
                    e.level.toInt() &gt;= TRACE.toInt() &amp;&amp; e.level.toInt() &lt;= WARN.toInt()
                </expression>
            </evaluator>
            <OnMatch>NEUTRAL</OnMatch>
            <OnMismatch>DENY</OnMismatch>
        </filter>
        <file>${LOG_PATH}/${PROJECT}-api.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_PATH}/${PROJECT}-api.%d{yyyy-MM-dd}.log</FileNamePattern>
            <MaxHistory>10</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %le %lo [%thread]: ## '%msg' '%ex'\n</pattern>
        </encoder>
    </appender>

    <!--&lt;!&ndash; Logstash TCP appender. &ndash;&gt;-->
    <!--<appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">-->
        <!--<destination>${LOGSTASH_HOST}:${LOGSTASH_PORT}</destination>-->
        <!--<keepAliveDuration>5 minutes</keepAliveDuration>-->
        <!--&lt;!&ndash; encoder is required &ndash;&gt;-->
        <!--<encoder class="net.logstash.logback.encoder.LogstashEncoder">-->
            <!--<customFields>{"project": "${PROJECT}"}</customFields>-->
            <!--<throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">-->
                 <!--&lt;!&ndash;正则匹配的日志信息不输出&ndash;&gt;-->
                 <!--<exclude>sun\.reflect\..*\.invoke.*</exclude>-->
                 <!--<exclude>net\.sf\.cglib\.proxy\.MethodProxy\.invoke</exclude>-->
                 <!--<exclude>org.*</exclude>-->
                 <!--<rootCauseFirst>true</rootCauseFirst>-->
            <!--</throwableConverter>-->
        <!--</encoder>-->
        <!-- Logstash UDP appender. -->
    <appender name="stash" class="net.logstash.logback.appender.LogstashSocketAppender">
        <!--logstash服务器IP-->
        <host>${LOGSTASH_HOST}</host>
        <!-- port is optional (default value shown) -->
        <port>${LOGSTASH_PORT}</port>
        <customFields>{"project": "${PROJECT}"}</customFields>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>

    <!-- Root logger -->
    <!-- 记录系统一般日志，Level values: DEBUG, ERROR, WARNING, INFO, ... -->
    <root level="INFO">
        <appender-ref ref="stdout"/>
        <appender-ref ref="file—default"/>
        <appender-ref ref="file—error"/>
        <appender-ref ref="stash" />
        <!--<appender-ref ref="file-str"/>-->
    </root>

    <!-- controller请求日志, 记录接口调用的日志 -->
    <logger name="com.codyy.cloudschool.integration.common.log" level="INFO" additivity="false">
        <appender-ref ref="api-log"/>
        <appender-ref ref="file—error"/>
        <appender-ref ref="stash" />
    </logger>

    <!-- 登录日志-->
    <logger name="login-logger" level="INFO" additivity="false">
        <appender-ref ref="login-info"/>
    </logger>

</configuration>
Logstash配置
先添加stash appender，filter level设为ERROR及以上，默认采用UDP模式，也可以设置为TCP模式
在系统默认（root）日志里添加stash 应用
<!--&lt;!&ndash; Logstash TCP appender. &ndash;&gt;-->
    <!--<appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">-->
        <!--<destination>${LOGSTASH_HOST}:${LOGSTASH_PORT}</destination>-->
        <!--<keepAliveDuration>5 minutes</keepAliveDuration>-->
        <!--&lt;!&ndash; encoder is required &ndash;&gt;-->
        <!--<encoder class="net.logstash.logback.encoder.LogstashEncoder">-->
            <!--<customFields>{"project": "${PROJECT}"}</customFields>-->
            <!--<throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">-->
                 <!--&lt;!&ndash;正则匹配的日志信息不输出&ndash;&gt;-->
                 <!--<exclude>sun\.reflect\..*\.invoke.*</exclude>-->
                 <!--<exclude>net\.sf\.cglib\.proxy\.MethodProxy\.invoke</exclude>-->
                 <!--<exclude>org.*</exclude>-->
                 <!--<rootCauseFirst>true</rootCauseFirst>-->
            <!--</throwableConverter>-->
        <!--</encoder>-->
        <!-- Logstash UDP appender. -->
    <appender name="stash" class="net.logstash.logback.appender.LogstashSocketAppender">
        <!--logstash服务器IP-->
        <host>${LOGSTASH_HOST}</host>
        <!-- port is optional (default value shown) -->
        <port>${LOGSTASH_PORT}</port>
        <customFields>{"project": "${PROJECT}"}</customFields>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>

    <!-- Root logger -->
    <!-- 记录系统一般日志，Level values: DEBUG, ERROR, WARNING, INFO, ... -->
    <root level="INFO">
        <appender-ref ref="stdout"/>
        <appender-ref ref="file—default"/>
        <appender-ref ref="file—error"/>
        <appender-ref ref="stash" />
        <!--<appender-ref ref="file-str"/>-->
    </root>
安装配置ELK
安装ELK
下载Elasticsearch v2.4.1安装，并启动elasticsearch，es不用修改配置，直接运行即可：./bin/elasticsearch。
Elasticsearch安装在10.10.20.34机器上，路径为/var/local/elasticsearch-2.4.1

修改es目录下config/elasticsearch.yml文件，修改host和port
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
 network.host: 10.10.20.34
#
# Set a custom port for HTTP:
#
 http.port: 9200
运行成功后，在浏览器打开网址：http://127.0.0.1:9200，如果看到如下信息，说明启动成功
{
  "name" : "uAPjUu9",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "tSq1s58NQdCBfJ2Aoo_LtA",
  "version" : {
    "number" : "6.3.2",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "053779d",
    "build_date" : "2018-07-20T05:20:23.451332Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
安装Logstash
下载Logstach v2.4.1安装（Logstash安装在10.10.20.34机器上，路径为/home/shdev/app/logstash-2.4.1）
新建配置文件：touch config/my-logstash.conf, 由于要接入logback，因此可做如下配置：
input {
    stdin {
    }
    udp {
        port => 514
        codec => "json"
        type => "syslog"
        buffer_size => 20000
    }
    tcp {
        port => 514
        codec => json
    }
}
output {
    elasticsearch {
        hosts => ["10.10.20.34:9200"]
    }
    stdout {
        codec => rubydebug
    }
    if [level] == "ERROR" and "kxt-" in [project] and "Exception" in [stack_trace] {
        email {
                to => "shdev.list@codyy.com,liheng@codyy.com"
                from => "codyysh@codyy.com"
                subject => "Exception: %{host} - %{project} - %{message}"
                body => "%{project} happened error in Host %{HOSTNAME}(%{host})\n\n%{stack_trace}"
                domain => "mail.codyy.com"
                port => 25
        }
    }
}
udp配置，监听端口为514， 日志类型为syslog，编码器为json
tcp配置，监听端口为514，编码器为json
修改elasticsearch的hosts配置，指定elasticsearch的地址和监听端口
发送java系统异常日志邮件通过22-31行的配置
以root用户启动logstash
> ./bin/logstash -c conf/logstash.conf

安装配置Kibana
下载Kibana v4.6.6安装（Kibana安装在10.10.20.34机器上，路径为 /home/shdev/app/kibana-4.6.6-linux-x86_64）
修改config/kibana.yml配置
设置elasticsearch.url、elasticsearch.username
# The Elasticsearch instance to use for all your queries.
elasticsearch.url: "http://10.10.20.34:9200"

# preserve_elasticsearch_host true will send the hostname specified in `elasticsearch`. If you set it to false,
# then the host you use to connect to *this* Kibana instance will be sent.
# elasticsearch.preserveHost: true

# Kibana uses an index in Elasticsearch to store saved searches, visualizations
# and dashboards. It will create a new index if it doesn't already exist.
# kibana.index: ".kibana"

# The default application to load.
# kibana.defaultAppId: "discover"

# If your Elasticsearch is protected with basic auth, these are the user credentials
# used by the Kibana server to perform maintenance on the kibana_index at startup. Your Kibana
# users will still need to authenticate with Elasticsearch (which is proxied through
# the Kibana server)
elasticsearch.username: "es_admin"
# elasticsearch.password: "123456"
启动kabana： ./bin/kibana
在浏览器打开kibana系统：http://localhost:5601
附：CentOS 7.3下Elasticsearch 2.4.1+Kibana 4.6.6+Logstash 2.4.1及插件安装配置
https://blog.csdn.net/mawenwu1983/article/details/78142935

Elasticsearch 2.4.1

下载地址

https://www.elastic.co/downloads/past-releases/elasticsearch-2-4-1

https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.1/elasticsearch-2.4.1.tar.gz

要求：

1 不能以ROOT身份运行，所以需要新建一个用户

adduser 新用户名

passwd 新用户名

2 因为不能以ROOT身份运行，所以需要更改文件夹权限

sudo chown -R elasticsearch新用户名

3 /elasticsearch/config/elasitcsearch.yml配置

cluster.name: 集群名

node.name: 节点名

network.host: 0.0.0.0 #设置非本机访问

http.port: 9200 #端口号 默认9200

4 运行

./elasticsearch

5 后台运行

./elasticsearch -d

Kibana 4.6.6

https://www.elastic.co/downloads/past-releases/kibana-4-6-6

https://download.elastic.co/kibana/kibana/kibana-4.6.6-linux-x86_64.tar.gz

配置/kibana/config/kibana.yml

elasticsearch.url: "http://localhost:9200" #修改为你的elasticsearch的地址和端口

elasticsearch.username: "es_admin" #安装shield插件需要开启这个验证

elasticsearch.password: "安装shield时候设置的密码" #安装shield插件需要开启这个验证

Logstash 2.4.1

https://www.elastic.co/downloads/past-releases/logstash-2-4-1

https://download.elastic.co/logstash/logstash/logstash-2.4.1.tar.gz

在bin目录下新建一个logstash.conf，并输入下列内容

input { stdin { } }output { elasticsearch { hosts => ["localhost:9200"] } stdout { codec => rubydebug }}

然后运行./logstash -f logstash.conf

Marvel

https://www.elastic.co/downloads/marvel

安装

./elasticsearch/bin/plugin install license

./elasticsearch/bin/plugin install marvel-agent

./kibana/bin/kibana plugin --install elasticsearch/marvel/latest

Shield

https://www.elastic.co/downloads/shield

安装

./elasticsearch/bin/plugin install license

./elasticsearch/bin/plugin install shield

./elasticsearch/bin/shield/esusers useradd es_admin -r admin

Sense

安装

https://www.elastic.co/guide/en/elasticsearch/guide/2.x/running-elasticsearch.html#sense

./kibana/bin/kibana plugin --install elastic/sense
