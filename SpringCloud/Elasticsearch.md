# Elasticsearch 学习笔记

## 一、简介

**Elasticsearch** 是一个 **分布式搜索与分析引擎**，用于快速地存储、搜索和分析海量数据。
 官网：https://www.elastic.co/cn/elasticsearch/

------

## 二、 Docker 安装

### 1、启动 Elasticsearch

```
docker run -d \
  --name es \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -e "discovery.type=single-node" \
  -v es-data:/usr/share/elasticsearch/data \
  -v es-plugins:/usr/share/elasticsearch/plugins \
  --privileged \
  --network hm-net \
  -p 9200:9200 \
  -p 9300:9300 \
  elasticsearch:7.12.1
```

📍访问地址：http://192.168.195.131:9200

------

### 2、启动 Kibana

```
docker run -d \
  --name kibana \
  -e ELASTICSEARCH_HOSTS=http://es:9200 \
  --network=hm-net \
  -p 5601:5601  \
  kibana:7.12.1
```

📍访问地址：[http://192.168.195.131:5601](http://192.168.195.131:5601/)

<img width="1876" height="932" alt="kibana" src="https://github.com/user-attachments/assets/cefb4cac-1821-4005-91af-d4e2ee5429a6" />

<img width="659" height="748" alt="dev_tools" src="https://github.com/user-attachments/assets/d3ee44c1-49b9-40ca-a2df-803734494643" />


------

## 三、 倒排索引

- 传统数据库是 **从 ID 找内容**
- **倒排索引（Inverted Index）** 是 **从词条找文档 ID**

例如：

| 词条   | 文档ID    |
| ------ | --------- |
| 程序员 | [1, 3, 5] |
| 开发   | [2, 3]    |
| Java   | [1, 4]    |

------

## 四、IK 分词器（中文分词）

### 1、下载地址

https://release.infinilabs.com/analysis-ik/stable/

------

### 2、安装步骤

1. 查看所有数据卷

   ```
   docker volume ls
   ```

2. 查看数据卷位置

   ```
   docker volume inspect es-plugins
   ```

3. 进入对应目录，将下载好的 `analysis-ik` 文件夹拖入

4. 重启 Elasticsearch

   ```
   docker restart es
   ```
<img width="1391" height="562" alt="ik" src="https://github.com/user-attachments/assets/a46d2de7-bacf-4a47-9608-578785e6ab46" />


------

## 五、 测试分词器

### 1、 标准分词器

```
POST /_analyze
{
  "analyzer": "standard",
  "text": ["Hello，我是你们的爸爸！"]
}
```
<img width="1876" height="895" alt="标准分词" src="https://github.com/user-attachments/assets/eb9acdb6-3188-43ce-b39b-02ed4bfbbbfd" />


------

### 2、 IK 智能分词（ik_smart）

```
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": ["Hello，我是你们的爸爸！"]
}
```

📌 分词结果较粗，效率高。

------

### 3、 IK 最细粒度分词（ik_max_word）

```
POST /_analyze
{
  "analyzer": "ik_max_word",
  "text": ["Hello，我是程序员！"]
}
```

📌 分词更细，例如「程序员」→「程序」、「程序员」
<img width="1875" height="897" alt="ik分词" src="https://github.com/user-attachments/assets/cb5ac802-ab7a-4ec4-a2ed-dbcecdd1fc87" />


------

## 六、 扩展词典配置

当某些词汇（如行业术语）未被识别时，可添加自定义词典。

1. 找到配置文件：
    `elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml`
<img width="1472" height="831" alt="扩展词典" src="https://github.com/user-attachments/assets/dbdcad5a-45d2-4ce0-ad56-e94aa016a063" />

<img width="1618" height="799" alt="扩展词典2" src="https://github.com/user-attachments/assets/86da5175-5d9c-4db7-a1eb-88698d033335" />


1. 在文件中配置扩展词典，例如：

   ```
   <properties>
     <entry key="ext_dict">ext.dic</entry>
   </properties>
   ```

2. 在 `config` 目录下新建 `ext.dic`，添加自定义词汇：

   ```
   卧槽
   干甚
   ```

3. 上传覆盖后（只需要将修改的文件拖上去就行），**重启 ES**：

   ```
   docker restart es
   ```
<img width="1827" height="865" alt="扩展词典3" src="https://github.com/user-attachments/assets/cb40c094-2682-4369-adaf-dcf410aa15be" />
