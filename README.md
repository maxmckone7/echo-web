# Echo Web
[![Slack](https://img.shields.io/badge/slack-join-D60051.svg)](https://hbchen.slack.com/messages/CE6S4HJN6)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fmaxmckone7%2Fecho-web.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fmaxmckone7%2Fecho-web?ref=badge_shield)

Go web framework Echo example. <br/>`master`为`v4`版本，`v3`切换到`v3`分支。
> Requires

[![Slack](https://img.shields.io/badge/Echo-v4+-41DAFF.svg)](https://github.com/labstack/echo)
[![Slack](https://img.shields.io/badge/go-1.11+-2A489A.svg)](https://golang.org)
[![Slack](https://img.shields.io/badge/GO111MODULE-on-brightgreen.svg)](https://golang.org)

> Echo中文文档 [go-echo.org](http://go-echo.org/)

> [在线演示](http://echo.www.hbchen.com/)

#### 目录
- [环境配置](#环境配置)
- [运行](#运行)
- [打包](#打包)
- [目录结构](#目录结构)
- [框架功能](#框架功能)
- [Supervisord部署](#supervisord%E9%83%A8%E7%BD%B2)
- [Confd管理配置](#confd%e7%ae%a1%e7%90%86%e9%85%8d%e7%bd%ae)
- [OpenTracing](#opentracing)
- [Metrics](#metrics)
- [Docker部署](#docker%e9%83%a8%e7%bd%b2)
- [pprof](#pprof)

## 环境配置

##### 1.源码下载
```shell
$ go get github.com:hb-go/echo-web
```

##### 2.依赖安装
###### go mod
> GFW[goproxy.io](https://goproxy.io/)

###### ~~Glide~~
> [glide工具安装](https://github.com/Masterminds/glide#install)

> .glide/mirrors.yaml[设置参考](glide.mirrors.yaml)
```shell
# 编辑mirrors.yaml，或命令行glide mirror set [original] [replacement] --vcs [type]
$ vi ~/.glide/mirrors.yaml

# 查看glide mirror
$ glide mirror list

$ cd $GOPATH/src/github.com/hb-go
$ glide install
```

##### 3.MySQL配置
```shell
# ./conf/conf.toml
[database]
name = "goweb_db"
user_name = "goweb_dba"
pwd  = "123456"
host = "127.0.0.1"
port = "3306"

# 测试数据库SQL脚本
./echo-web/res/db_structure.sql
```

##### 4.Redis、Memcached配置，可选

> 可选需修改session、cache的store配置
- session_store = "FILE"或"COOKIE"
- cache_store = "IN_MEMORY"


```shell
# ./conf/conf.toml
[redis]
server = "127.0.0.1:6379"
pwd = "123456"

[memcached]
server = "localhost:11211"
```

##### 5.子域名
```shell
# ./conf/conf.toml
[server]
addr = ":8080"
domain_api = "echo.api.localhost.com"
domain_web = "echo.www.localhost.com"
domain_socket = "echo.socket.localhost.com"

# 改host
$ vi /etc/hosts
127.0.0.1       echo.api.localhost.com
127.0.0.1       echo.www.localhost.com
127.0.0.1       echo.www.localhost.com

# Nginx配置，可选
server{
    listen       80;
    server_name  echo.www.localhost.com echo.api.localhost.com echo.socket.localhost.com;

    charset utf-8;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

##### 6.Bindata打包工具，可选(运行可选，打包必选)
> [Bindata安装](https://github.com/jteeuwen/go-bindata#installation)

## 运行
```shell
# 首次运行必须带 -a -t，更新assets.go、template.go的资源路径为本地
$ ./run.sh [-a] [-t]        # -a -t 可选(须安装Bindata)，以debug方式更新assets、template的Bindata资源包

# 浏览器访问
http://echo.www.localhost.com      # Nginx代理
http://echo.www.localhost.com:8080 # 无代理

# OpenTracing
http://localhost:8700/traces
```

## 打包
> 打包静态资源及模板文件须[安装Bindata](https://github.com/jteeuwen/go-bindata#installation)

```shell
# 打包
$ ./build.sh 		    # 默认本机
$ ./build.sh -l		    # 打包Linux平台

# 运行
$ ./echo-web -c conf/conf.toml      # -c指定配置文件，默认conf/conf.toml
```
## 目录结构
```sh
assets          Web服务静态资源
conf            项目配置
middleware      中间件
model           模型，数据库连接&ORM
  └ orm         ORM扩展
module          模块封装
  ├ auth        Auth授权
  ├ cache       缓存
  ├ log         日志
  ├ render      渲染
  ├ session     Session
  └ tmpl        Web模板
res             项目资源
  └ db          数据
router          路由
  └ api         接口路由
    ├context    自定义Context，便于扩展API层扩展
    └router     路由
  ├ socket      socket示范
  └ web         Web路由
    ├context    自定义Context，便于扩展Webb层扩展
    └router     路由        
template        模板
  └ pongo2      pongo2模板
util            公共工具
  ├ conv        类型转换
  ├ crypt       加/解密
  └ sql         SQL
```

## 框架功能

功能 | 描述
:--- | :---
[配置](https://github.com/hb-go/echo-web/tree/master/conf) | [toml](http://github.com/BurntSushi/toml)配置文件
[子域名部署](https://github.com/hb-go/echo-web/blob/master/router/router.go) | 子域名区分模块
[缓存](https://github.com/hb-go/echo-web/blob/master/module/cache) | Redis、Memcached、Memory
[Session](https://github.com/hb-go/echo-web/blob/master/module/session) | Redis、File、Cookie，支持Flash
[ORM](https://github.com/hb-go/echo-web/tree/master/model) | Fork [gorm](http://github.com/jinzhu/gorm)，`FirstSQL`、`LastSQL`、`FindSQL`、`CountSQL`支持构造查询SQL
[查询缓存](https://github.com/hb-go/echo-web/tree/master/model/orm) | 支持`First`、`Last`、`Find`、`Count`的查询缓存
[模板](https://github.com/hb-go/echo-web/tree/master/module/render) | 支持html/template(~~示例模板未维护~~)、[pongo2](http://github.com/flosch/pongo2)，模板支持打包[bindata](https://github.com/jteeuwen/go-bindata#installation)
静态 | 静态资源，支持打包[bindata](https://github.com/jteeuwen/go-bindata#installation)
安全 | CORS、CSRF、XSS、HSTS、验证码等
[OpenTracing](http://opentracing.io/) | Tracer支持Jaeger、Appdash，在Request、ORM层做跟踪，可在conf配置开启)
监控 | `Prometheus` `Grafana` `Graphite` `InfluxDB` 
其他 | JWT、Socket演示

## Supervisord部署
```bash
# Max open files问题，修改supervisord配置
$ vi /etc/supervisor/supervisord.conf
[supervisord]
minfds = 10240

$ vi /etc/supervisor/conf.d/echo-web.conf
```

##### 配置参考
```bash
[program:echo-web]
directory = /{path}/echo-web                                            ; 程序的启动目录
command =  /{path}/echo-web/echo-web -c /{path}/echo-web/conf/conf.toml ; 启动命令，与手动在命令行启动的命令是一样的
process_name=%(program_name)s                                           ; process_name expr (default %(program_name)s)
numprocs=1           ; number of processes copies to start (def 1)
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = root          ; 用哪个用户启动
redirect_stderr = true          ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 7      ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /var/log/echo-web/stdout.log
; 这一配置项的作用是：如果supervisord管理的进程px又产生了若干子进程，使用supervisorctl停止px进程，停止信号会传播给px产生的所有子进程，确保子进程也一起停止。这一配置项对希望停止所有进程的需求是非常有用的。
stopasgroup=false
killasgroup=false
environment=GOGC=1000
```

## Confd管理配置
```bash
# 安装confd
$ go get github.com/kelseyhightower/confd
```
```bash
# 将配置写入etcd，统一前缀echo-web
$ etcdctl ls --recursive --sort /echo-web
/echo-web/app
/echo-web/app/name
......
/echo-web/tmpl/suffix
/echo-web/tmpl/type

$ cd {pwd}/echo-web
$ confd -onetime -confdir conf  -backend etcd -node http://127.0.0.1:4001 -prefix echo-web
```

## OpenTracing
> 在conf.toml开启或禁用
- Appdash可直接使用，查看[http://localhost:8700](http://localhost:8700)
- Jaeger需要搭建服务，可在[Docker搭建开发环境](http://jaeger.readthedocs.io/en/latest/getting_started/#all-in-one-docker-image)，查看[http://localhost:16686](http://localhost:16686)
```bash
[opentracing]
disable = false

# jaeger or appdash
type = "jaeger"

# jaeger serviceName
service_name = "echo-web"

# jaeger-agent 127.0.0.1:6831
# appdash http://localhost:8700
address = "127.0.0.1:6831"
```

## Metrics
> 在conf.toml开启或禁用
```bash
[metrics]
disable = false
freq_sec = 10
address = "127.0.0.1:2003"  # Graphite
```
#### Grafana
浏览器:localhost:3000
```bash
docker run --name grafana -d -p 3000:3000 grafana/grafana
```

#### Prometheus
浏览器:localhost:9090
```bash
docker run -d --name prometheus -p 9090:9090 -v ~/tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
              prom/prometheus
              
# Dashboard JSON
# middleware/metrics/prometheus/grafana.json
```
[更多参考middleware/metrics/README.md](/middleware/metrics/README.md)

## Docker部署
> Dockerfile含两种方式，源码方式镜像包偏大
```bash
# 创建镜像，注意修改配置
$ docker build -t hbchen/echo-web:v0.0.1 .

# 运行容器
$ docker run  \
     -p 8080:8080 \
     --name=echo-web \
     hbchen/echo-web:v0.0.1
```

#### *MySQL、Redis、Memcached等服务配置问题
```bash
# 如果是服务在宿主机需要配置服务IP为主机IP，127.0.0.1/localhost网络不通

# hbchen/echo-web使用的配置，在宿主机host上做个映射(192.168.1.8为主机IP)
# 192.168.1.8 mysql.localhost.com
# 192.168.1.8 redis.localhost.com
# 192.168.1.8 memcached.localhost.com
[database]
host = "mysql.localhost.com"

[redis]
server = "redis.localhost.com:6379"

[memcached]
server = "memcached.localhost.com:11211"
```

##### 自动修改/etc/hosts，映射自定义域名到主机IP
```bash
$ vi ~/.profile

# 添加
grep -v "etcd.localhost.com\|consul.localhost.com\|mysql.localhost.com\|redis.localhost.com\|memcached.localhost.com" /etc/hosts > ~/hosts_temp
cat ~/hosts_temp > /etc/hosts
LC_ALL=C ifconfig en0 | grep 'inet ' | cut -d ' ' -f2 | awk '{print $1 " etcd.localhost.com\n" $1 " consul.localhost.com\n" $1 " mysql.localhost.com\n"$1     " redis.localhost.com\n" $1 " memcached.localhost.com"}' >> /etc/hosts

# 修改配置或切换网络时更新
$ source ~/.profile
```

## pprof

```go
import "github.com/hb-go/echo-web/middleware/pprof"

echo.Pre(pprof.Serve())
/*或*/
echo.Use(pprof.Serve())
```
Web浏览prof信息
> http://{HOST}/debug/pprof/

[google/pprof](https://github.com/google/pprof)
- [Building pprof](https://github.com/google/pprof#building-pprof)
- [Run pprof via a web interface](https://github.com/google/pprof#run-pprof-via-a-web-interface)
```bash
$ go get -u github.com/google/pprof

$ pprof -http=[host]:[port] [main_binary] profile.pb.gz

# 使用echo-web演示
$ pprof -http=localhost:8080 --alloc_space http://echo.www.hbchen.com/debug/pprof/heap
```



## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fmaxmckone7%2Fecho-web.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fmaxmckone7%2Fecho-web?ref=badge_large)