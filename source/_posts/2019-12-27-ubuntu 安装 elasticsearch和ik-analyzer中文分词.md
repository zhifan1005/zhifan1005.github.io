---
title:  ubuntu 安装 elasticsearch和ik-analyzer中文分词
date: 2019-12-27 20:15:37
tags: 
- ubuntu 
- elasticsearch 
- ik-analyzer
categories: Server
---

@[toc]

>***收集的好文链接(长期更新)***

---
# 安装java环境

>我们首先要配置java环境，这个需要Java8或者更新的包。一般执行以下命令时可以看看Java版本是否符合：

```bash
sudo apt install default-jdk
```

# 安装Elasticsearch

## 安装公钥
在安装之前我们需要下载和安装公钥，否则没有办法使用apt安装Elasticsearch。
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

## 添加源

```bash
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```
## 安装es

```bash
sudo apt-get update && sudo apt-get install elasticsearch
```

>这样就安装好了。

## 命令管理

### systemd

>设置开机启动：

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
```

### 启动

```bash
sudo systemctl start elasticsearch
```
### 关闭

```bash
sudo systemctl stop elasticsearch
```


### 端口与查看

```bash
netstat -plntu  
```

## 访问试下

```bash
curl -XGET '127.0.0.1:9200/?pretty'
```

>如果返回的数据如下所示，即表示安装成功了：

```
{
  "name" : "fi_nt2e",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "mO1Ly_8YSnWm1gYNPIL-MA",
  "version" : {
    "number" : "6.8.6",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "3d9f765",
    "build_date" : "2019-12-13T17:11:52.013738Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.2",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

> 注意：Elasticsearch需要启动一会。如果启动完成立马执行上面的命令，可以会提示拒绝连接，多试几次就好了。

## 配置 Elasticsearch

### 配置参数
>Elasticsearch 默认情况下从 /etc/elasticsearch/elasticsearch.yml 文件中加载它的配置。

RPM也又一个系统配置文件（/etc/sysconfig/elasticsearch），它允许你设置以下参数：


参数  | 解释
---|---
JAVA_HOME | 设置要使用的自定义Java路径
MAX_OPEN_FILES  |  打开文件的最大数量，默认65536
MAX_LOCKED_MEMORY|最大锁内存大小。如果你在elasticsearch.yml中使用bootstrap.memory_lock选项，请设置unlimited
MAX_MAP_COUNT|进程可能拥有的内存映射区域的最大值。如果使用 mmapfs 作为索引存储类型，请确认将其设置为较高的值。请检查linux内核文档关于max_map_count的更多信息。这是在elasticsearch启动之前通过sysctl设置的。默认是262144
ES_PATH_CONF|配置文件目录（需要包含elasticsearch.yml和log4j2.properties文件），默认路径是：/etc/elasticsearch
ES_JAVA_OPTS|	Any additional JVM system properties you may want to apply.
RESTART_ON_UPGRADE | 配置软件包升级时将会重新启动，默认是false。这意味着你在安装软件包之后手动重启elasticsearch实例。这样做的原因是为了保障, 在集群中更新时，在高流量网络和减少你集群的响应时间的情况下导致分片的重新分配。

### 包的目录布局


类型 | 描述| 默认路径 | 设置
---|---|---|---
home | Elasticsearch家目录或者$ES_HOME | /usr/share/elasticsearch |
bin | 二进制脚本，包括elasticsearch去启动一个节点和elasticsearch-plugin安装插件 |/etc/elasticsearch|conf|配置文件，包含elasticsearch.yml|/etc/elasticsearch|ES_PATH_CONF
conf|	环境变量，包含 heap 大小，文件描述符。|/etc/default/elasticsearch|
data|	在节点上分配的每个索引/分片的数据文件的位置。可以容纳多个位置。|/var/lib/elasticsearch|path.data
logs|	日志文件位置。|/var/log/elasticsearch|	path.logs
plugins|插件文件位置. 每个插件将包含在一个子目录中.|/usr/share/elasticsearch/plugins|
repo|共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放置在指定目录中任何子目录中。|Not configured|path.repo

# 安装 ik-analyzer

>注意这里一定要选择和你的es版本匹配的ik版本

## 安装

```bash
/usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.6/elasticsearch-analysis-ik-6.8.6.zip
```
>我的安装过程如下

```bash
root@JD:/etc/nginx/sites-available# /usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.6/elasticsearch-analysis-ik-6.8.6.zip
warning: Falling back to java on path. This behavior is deprecated. Specify JAVA_HOME
-> Downloading https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.6/elasticsearch-analysis-ik-6.8.6.zip
[=================================================] 100%   
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.net.SocketPermission * connect,resolve
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed analysis-ik
```
>出现以上显示 表示安装成功

## 重启服务

```bash
sudo systemctl restart elasticsearch  
```
>重启之后要等待一下

## 测试

### 测试不使用分词

```bash
root@JD:/var/www/bbs# curl -H 'Content-Type: application/json'  -XGET 'localhost:9200/_analyze?pretty' -d '{"text":"北京欢迎你"}' 
{
  "tokens" : [
    {
      "token" : "北",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "京",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "欢",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "迎",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "你",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    }
  ]
}
```

### 测试使用分词

```bash
root@JD:/var/www/bbs# curl -H 'Content-Type: application/json'  -XGET 'localhost:9200/_analyze?pretty' -d '{"analyzer":"ik_max_word","text":"北京欢迎你"}'  
{
  "tokens" : [
    {
      "token" : "北京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "欢迎",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "你",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 2
    }
  ]
}
```

