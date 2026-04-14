# docker
1. 先卸载旧版本（如果有）

```bash
sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

2. 安装依赖

```bash
sudo dnf install -y dnf-plugins-core
```

3. 添加 Docker 官方源（关键）

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

4. 安装 Docker 引擎

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

5. 启动并设置开机自启

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

6. 验证安装成功

```bash
sudo docker run hello-world
```
出现 `Hello from Docker!` 就代表安装成功 ✅

---

## 常用 Docker 命令（方便你用）
```bash
docker --version        # 查看版本
systemctl status docker # 查看运行状态
docker ps               # 查看运行容器
docker images           # 查看镜像
```



# nacos安装

一、创建挂载目录（必执行）

```
# 创建 nacos 三大挂载目录
mkdir -p /data-mars/nacos/conf
mkdir -p /data-mars/nacos/logs
mkdir -p /data-mars/nacos/data
```

二、临时启动 nacos 复制文件

```
# 临时启动容器（用于复制配置文件）
docker run -d --name nacos -p 8848:8848 -e MODE=standalone nacos/nacos-server:v2.2.1

# 复制配置文件到宿主机
docker cp nacos:/home/nacos/conf/ /data-mars/nacos/

# 复制日志目录
docker cp nacos:/home/nacos/logs/ /data-mars/nacos/

# 复制数据目录
docker cp nacos:/home/nacos/data/ /data-mars/nacos/

# 删除临时容器
docker rm -f nacos
```

三、最终启动 nacos（正式运行）

```
docker run -d --name nacos -p 8848:8848 -p 9848:9848 -p 9849:9849 --env MODE=standalone --env NACOS_AUTH_ENABLE=true -v /data-mars/nacos/conf:/home/nacos/conf -v /data-mars/nacos/logs:/home/nacos/logs -v /data-mars/nacos/data:/home/nacos/data --restart=always nacos/nacos-server:v2.4.3
```

上面的启动开启了认证所以需要下面的配置

```
# spring
server.servlet.contextPath=${SERVER_SERVLET_CONTEXTPATH:/nacos}
server.contextPath=/nacos
server.port=${NACOS_APPLICATION_PORT:8848}
server.tomcat.accesslog.max-days=30
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i %{Request-Source}i
server.tomcat.accesslog.enabled=${TOMCAT_ACCESSLOG_ENABLED:false}
server.error.include-message=ALWAYS
# default current work dir
server.tomcat.basedir=file:.
#*************** Config Module Related Configurations ***************#
### Deprecated configuration property, it is recommended to use `spring.sql.init.platform` replaced.
#spring.datasource.platform=${SPRING_DATASOURCE_PLATFORM:}
spring.sql.init.platform=${SPRING_DATASOURCE_PLATFORM:}
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
db.num=${MYSQL_DATABASE_NUM:1}
db.url.0=jdbc:mysql://${MYSQL_SERVICE_HOST}:${MYSQL_SERVICE_PORT:3306}/${MYSQL_SERVICE_DB_NAME}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}
db.user.0=${MYSQL_SERVICE_USER}
db.password.0=${MYSQL_SERVICE_PASSWORD}
## DB connection pool settings
db.pool.config.connectionTimeout=${DB_POOL_CONNECTION_TIMEOUT:30000}
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2
### The auth system to use, currently only 'nacos' and 'ldap' is supported:
nacos.core.auth.system.type=${NACOS_AUTH_SYSTEM_TYPE:nacos}
### worked when nacos.core.auth.system.type=nacos
### The token expiration in seconds:
### If turn on auth system:
nacos.core.auth.system.type=nacos
nacos.core.auth.enabled=true
nacos.core.auth.plugin.nacos.token.expire.seconds=${NACOS_AUTH_TOKEN_EXPIRE_SECONDS:18000}
### The default token:
nacos.core.auth.plugin.nacos.token.secret.key=${NACOS_AUTH_TOKEN:YWJjZGVmZ2hpamtsbW5vcHFyc3R1dnd4eXpBQkNERUZHSElKS0xNTk9QUVJTVFVWV1hZWg==}
### Turn on/off caching of auth information. By turning on this switch, the update of auth information would have a 15 seconds delay.
nacos.core.auth.caching.enabled=${NACOS_AUTH_CACHE_ENABLE:false}
nacos.core.auth.enable.userAgentAuthWhite=${NACOS_AUTH_USER_AGENT_AUTH_WHITE_ENABLE:false}
nacos.core.auth.server.identity.key=${NACOS_AUTH_IDENTITY_KEY:itzhouli}
nacos.core.auth.server.identity.value=${NACOS_AUTH_IDENTITY_VALUE:itzhouli}
## spring security config
### turn off security
nacos.security.ignore.urls=${NACOS_SECURITY_IGNORE_URLS:/,/error,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/**,/v1/console/health/**,/actuator/**,/v1/console/server/**}
# metrics for elastic search
management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false
nacos.naming.distro.taskDispatchThreadCount=10
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.console.ui.enabled=true
nacos.core.param.check.enabled=true

```



# nginx安装

```
# 1. 创建新的挂载目录
mkdir -p /data-mars/nginx/conf
mkdir -p /data-mars/nginx/log
mkdir -p /data-mars/nginx/html

# 2. 临时启动 Nginx 容器（用于复制配置文件）
docker run --name nginx -p 80:80 -d nginx:1.26.3

# 3. 复制容器内文件到宿主机新挂载目录
docker cp nginx:/etc/nginx/nginx.conf /data-mars/nginx/conf/nginx.conf
docker cp nginx:/etc/nginx/conf.d /data-mars/nginx/conf/conf.d
docker cp nginx:/usr/share/nginx/html /data-mars/nginx/

# 4. 删除临时容器
docker rm -f nginx

# 5. 正式启动 Nginx（目录挂载到 /data-mars/nginx）
docker run -p 80:80 -p 443:443 --name nginx --restart=always -v /data-mars/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /data-mars/nginx/conf/conf.d:/etc/nginx/conf.d -v /data-mars/nginx/log:/var/log/nginx -v /data-mars/nginx/html:/usr/share/nginx/html -d nginx:1.26.3
```



# coturn安装

```
mkdir -p /data-mars/coturn/config
vim  /data-mars/coturn/config/turnserver.conf


# 启动容器
docker run -d --name coturn --restart always --network host -v /data-mars/coturn/config/turnserver.conf:/etc/turnserver.conf -e TZ=Asia/Shanghai coturn/coturn:4.9.0-r0 -c /etc/turnserver.conf
```

firewall-cmd --add-port=3478/tcp --permanent
firewall-cmd --add-port=3478/udp --permanent
firewall-cmd --add-port=49152-65535/udp --permanent
firewall-cmd --reload

```
# 公网IP映射（公网IP/内网IP）
external-ip= xx.xxx.xx.234/192.168.90.144

# 监听主机（内网IP） 0.0.0.0  # 监听所有IP
listening-ip=0.0.0.0
# 监听端口
listening-port=3478
# 标识 Coturn 你的域名（或服务器IP）
realm=


# 认证
lt-cred-mech
# 用户名:密码
user=user:123

# 媒体端口范围
min-port=49152
max-port=65535
# 输出日志到 stdout
log-file=stdout
# 详细日志
verbose
```

