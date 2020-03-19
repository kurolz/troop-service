# Troop
Troop是运维自动化的底层基础设施，一套完整的服务器集中化管理系统。

[基础架构图](https://www.processon.com/view/link/5dc23dace4b04913a28be048)

## 需要部署的服务（4）：
- 服务端：General
- 客户端（Agent）：Scout
- 文件服务器：FileManager
- 命令行工具：Client

## 安装：
### 环境准备：
#### 1.安装MySQL：
```
yum install -y mysql-server
```
#### 2.创建`general`数据库
#### 3.安装RabbitMQ
```
docker pull rabbitmq:management
docker run -d --hostname my-rabbit -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 --name rabbitmq rabbitmq:management
```
#### 4.安装File服务(文件服务)
```
wget https://github.com/kurolz/troop-service/raw/master/troop_file_linux_1.0.0_amd64.tar.gz
tar -zxvf troop_file_linux_1.0.0_amd64.tar.gz
cd troop_file_linux_1.0.0_amd64
./install.sh

# 修改配置文件
/usr/local/troop-scout/conf/config.ini

# 重启服务
service troop-scoutd restart
```

#### 5.安装General服务(服务端)
保证服务器必须拥有General配置文件中的plugin-git地址的clone和pull权限，如未填写git地址或无权限拉取，所有Scout的自更新和插件功能将无法使用
```
wget https://github.com/kurolz/troop-service/raw/master/troop_general_linux_1.0.0_amd64.tar.gz
tar -zxvf troop_general_linux_1.0.0_amd64.tar.gz
cd troop_general_linux_1.0.0_amd64
./install.sh

# 修改配置文件
/usr/local/troop-general/conf/config.ini

# 重启服务
service troop-generald restart
```

#### 6.安装Client服务(命令行工具)
```
wget https://github.com/kurolz/troop-service/raw/master/troop_client_linux_1.0.0_amd64.tar.gz
tar -zxvf troop_client_linux_1.0.0_amd64.tar.gz
cd troop_client_linux_1.0.0_amd64
./install.sh

# 修改配置文件
/usr/local/troop-client/conf/config.ini
```

#### 7.安装scout服务(Agent客户端)

##### Linux
```
wget https://github.com/kurolz/troop-service/raw/master/troop_scout_linux_1.0.0_amd64.tar.gz
tar -zxvf troop_scout_linux_1.0.0_amd64.tar.gz
cd troop_scout_linux_1.0.0_amd64
./install.sh

# 修改配置文件
/usr/local/troop-scout/conf/config.ini

# 重启服务
service troop-scoutd restart
```

##### Windows ##
点击链接下载

[https://github.com/kurolz/troop-service/raw/master/troop-scout.msi](https://github.com/kurolz/troop-service/raw/master/troop-scout.msi)

双击安装，最好直接安装到C:\

修改配置文件

```
# 配置文件

安装目录\troop-scout\config.ini
```

重启服务
```
net stop troop-scout
net start troop-scout
```

---
### 配置说明：
#### General（服务端）
```
[debug]
enabled = true  # debug输出信息，当设置为false时，会将输出信息打印到log文件

[mysql]
host = 127.0.0.1  # MySQL连接Host
port = 3306  # MySQL连接端口
user = root  # MySQL连接用户名
password =  # MySQL连接密码
db = general  # 使用的数据库
charset = utf8  # 字符集
max_idle_connection = 10  # MySQL连接池允许的最大闲置的连接数
max_open_connection = 100  # MySQL连接池允许的最大打开的连接数

[rpc]
listen = 6954  # 监听的RPC端口

[rabbit_mq]
user = guest  # RabbitMQ连接用户名
password = guest  # RabbitMQ连接密码
host = 127.0.0.1  # RabbitMQ连接Host
port = 5672  # RabbitMQ连接端口
vhost = /  # RabbitMQ使用的vhost
max_connection_num = 3  # RabbitMQ连接池允许的最大连接数
max_channel_num = 5  # RabbitMQ连接池允许的最大channel数

[http]
listen = 6858  # 监听的HTTP端口
token = default-token  # HTTP请求验证token

[file]
address = http://127.0.0.1:6859  # 文件服务器的连接信息，应使用所有Scout都能访问的地址

[log]
logfile = "general.log"  # 输出日志文件名

[scout]
auto_accept = false  # 是否自动接受Scout握手、交换密钥

[plugin]
git = ssh://git@gitlab.com/troop/plugins/plugin.git  
# 插件的git地址，保证服务器必须拥有此git地址的clone和pull权限，
# 如不填写或无权限拉取，所有Scout的自更新和插件功能将无法使用
```

#### Scout（Agent）
```
[debug]
enabled = true  # debug输出信息，当设置为false时，会将输出信息打印到log文件

[host]
hostname =  # 服务端识别本机器的唯一值，如未配置，默认通过`hostname`命令获取
ip =  # 服务端会保存每台机器的IP，如未配置，默认自动探测本机IP
tag =  # 给机器打上标签，服务端可通过标签识别机器，支持多个标签，通过英文逗号`,`分隔

[general]
addresses = 127.0.0.1:6954  # 服务端的地址，格式: Ip:Port
timeout = 5000  # 连接服务端的超时时间, 格式: 秒*1000，如5秒，则填5000

[log]
logfile = scout.log  # 输出日志文件名

[rabbit_mq]
user = guest  # RabbitMQ连接用户名
password = guest  # RabbitMQ连接密码
host = 127.0.0.1  # RabbitMQ连接Host
port = 5672  # RabbitMQ连接端口
vhost = /  # RabbitMQ使用的vhost

[plugin]
dir = plugins  # 存放插件的目录
plugins =  # 需要安装的插件，多个插件用英文逗号`,`分隔，可指定版本，如: pluginName:v1.3.4
```

#### Client（命令行客户端）
```
[general]
addresses = 127.0.0.1:6954  # 服务端的地址，格式: Ip:Port
token = default-token  # 访问服务端使用的token
```

#### File（文件服务）
```
[debug]
enabled = true

[http]
listen = 6859
token = default-token  # 和服务端交互使用的token

[log]
logfile = "file.log"
```

---
## 使用说明

### 获取命令行帮助手册
```
troop --help
```

### Scout客户端信任操作
```
troop scout -l  # 查看scout列表
troop scout -a  # 接受scout握手
troop scout -r  # 拒绝scout握手
troop scout -d  # 删除scout
```

### 任务推送
```
###################################
# 通用参数说明
-d, --detach      指定任务在后台运行，默认false
--device string   指定执行任务的机器设备类型，默认：server
-o, --os string   指定执行任务的机器操作系统类型, 如：linux 或 windows
-t, --tag string  通过执行任务的机器标签
###################################

# 获取后台运行的任务结果
troop result <task_id>

# 测试连通性
troop ping.pong '*'

# 执行Linux命令:
troop command '*' ifconfig -o linux
troop command '*' "/bin/bash -c ifconfig" -o linux

# 执行Windows命令:
troop command '*' ipconfig -o windows
troop command '*' "cmd /c ipconfig" -o windows

# 分发文件:
troop file.send '*' /opt/test.txt /tmp/

# 插件:
troop plugin.update '*'  # 指定目标机器更新插件
troop plugin.version '*'  # 获取目标机器的安装的插件版本列表
troop plugin '*' <plugin_name> deploy  # 让目标插件执行`部署服务`任务
troop plugin '*' <plugin_name> start   # 让目标插件执行`启动服务`任务
troop plugin '*' <plugin_name> stop    # 让目标插件执行`停止服务`任务
troop plugin '*' <plugin_name> restart # 让目标插件执行`重启服务`任务
troop plugin '*' <plugin_name> status  # 获取目标插件管理的`服务状态`

# 更新scout
troop scout.update '*' http://example.com/scout.tar.gz
```
