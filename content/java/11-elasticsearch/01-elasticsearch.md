---
title: Elastic Search
date: 2021-11-02
categories: [Elasticsearch]
toc: true
---

## Elastic Search 的安装与配置

#### Elasticsearch 的文件目录结构

| 目录    | 配置文件          | 描述                                                       |
| :------ | :---------------- | :--------------------------------------------------------- |
| bin     |                   | 脚本文件，包括启动 elasticsearch，安装插件、运行统计数据等 |
| config  | elasticsearch.yml | 集群配置文件，user，role based 相关配置                    |
| JDK     |                   | Java 运行环境                                              |
| data    | path.data         | 数据文件                                                   |
| lib     |                   | Java 类库                                                  |
| logs    | path.log          | 日志文件                                                   |
| modules |                   | 包含所有ES模块                                             |
| plugins |                   | 包含所有已安装插件                                         |

#### JVM 配置

- 修改 JVM - config/jvm.options
  - 7.1 下载的默认设置是 1GB
- 配置的建议
  - Xmx 和 Xms 设置成一样
  - Xmx 不要超过机器内存的50%
  - 不要超过30GB

#### 安装与查看插件

![image-20211102222810666](../../.vuepress/public/images/image-20211102222810666.png)

```bash
# 查看插件list
elasticsearch-plugin list
```

![image-20211102222941086](../../.vuepress/public/images/image-20211102222941086.png)

- Elasticsearch 提供插件的机制对系统进行扩展
  - Discovery Plugin
  - Analysis Plugin
  - Security Plugin
  - Management Plugin
  - Ingest Plugin
  - Mapper Plugin
  - Backup Plugin

#### 如何在开发机上运行多个实例

```bash
# 进程
bin/elasticsearch -E node.name=node1 -E cluster.name=allen -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E cluster.name=allen -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E cluster.name=allen -E path.data=node3_data -d

# 删除进程
ps | grep elasticsearch

kill pid
```



## Kibana的安装



