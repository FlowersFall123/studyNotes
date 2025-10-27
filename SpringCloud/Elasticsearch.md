# ES基础

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
<img width="1827" height="865" alt="扩展词典3" src="https://github.com/user-attachments/assets/81889a68-b96b-40cc-9c93-b3fa127fd1f7" />


# Elasticsearch 索引库操作

------

## 一、Mapping 映射

**定义：**
 `Mapping` 是对索引库中字段类型和行为的约束（类似数据库中的表结构定义）。

### 常见属性

| 属性         | 说明                          | 示例                     |
| ------------ | ----------------------------- | ------------------------ |
| `type`       | 字段数据类型                  | `"type": "text"`         |
| `index`      | 是否创建索引（默认为 `true`） | `"index": false`         |
| `analyzer`   | 指定使用的分词器              | `"analyzer": "ik_smart"` |
| `properties` | 子字段定义（对象类型使用）    | `"properties": {...}`    |

### 常见字段类型

| 类型                                  | 说明           | 示例                 |
| ------------------------------------- | -------------- | -------------------- |
| `text`                                | 可分词文本     | 商品描述、文章内容   |
| `keyword`                             | 不分词精确匹配 | 品牌、国家、邮箱、IP |
| `byte` / `short` / `integer` / `long` | 整型           | 年龄、编号           |
| `float` / `double`                    | 浮点型         | 价格、权重           |
| `boolean`                             | 布尔值         | 是否上架             |
| `date`                                | 日期类型       | 发布时间             |
| `object`                              | 对象类型       | 嵌套子字段           |

------

## 二、索引库（Index）的 CRUD 操作

### 1. 创建索引库

```
PUT /fz
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart",
        "index": true
      },
      "age": {
        "type": "byte"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object",
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

📌 **说明：**

- `info`：文本字段，使用 `ik_smart` 分词器。
- `email`：不参与索引。
- `name`：对象类型，包含两个子字段。

<img width="1848" height="897" alt="创建索引库" src="https://github.com/user-attachments/assets/034dbc02-b3a0-4b97-aa58-133ed62e8d43" />

------

### 2. 查询索引库

```
GET /fz
```

------

###  3. 删除索引库

```
DELETE /fz
```

------

###  4. 给索引库添加字段（修改映射）

```
PUT /fz/_mapping
{
  "properties": {
    "weight": {
      "type": "byte"
    }
  }
}
```

📌 **注意：**
 只能新增字段，不能修改已有字段的类型（需要重建索引）。

------

## 三、文档（Document）的 CRUD 操作

### 1. 新增文档

```
POST /fz/_doc/1
{
  "info": "你们好呀！",
  "email": "123@qq.com",
  "name": {
    "firstName": "f",
    "lastName": "z"
  }
}
```
<img width="1848" height="946" alt="文档创建索引库" src="https://github.com/user-attachments/assets/fdb95398-5a7a-449f-83a3-f788eecaedce" />

------

### 2. 查询文档

```
GET /fz/_doc/1
```

------

###  3. 删除文档

```
DELETE /fz/_doc/1
```

------

### 4. 全量修改（先删除后新增）

```
PUT /fz/_doc/1
{
  "info": "你们好呀！",
  "email": "456@qq.com",
  "name": {
    "firstName": "f",
    "lastName": "z"
  }
}
```

📌 **说明：**
 相当于“先删后增”，原字段会被全部覆盖。

------

###  5. 局部修改（保留未修改字段）

```
POST /fz/_update/1
{
  "doc": {
    "name": {
      "firstName": "f",
      "lastName": "3"
    }
  }
}
```

📌 **说明：**

- 使用 `_update` + `doc` 关键字。
- 仅更新指定字段，其余字段保持不变。



# JavaRestClient-创建索引

## 一、依赖与初始化

### Maven 依赖

```
<!-- Elasticsearch Java 高级 REST 客户端 -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

> ⚠️ 注意：
>  `RestHighLevelClient` 在 Elasticsearch 8.x 中已经被弃用，改为 `ElasticsearchClient`。
>  本示例适用于 ES 7.x 系列。

**补充：如果存在问题请参考文档修改对应的依赖：**[文档链接](https://b11et3un53m.feishu.cn/wiki/LDLew5xnDiDv7Qk2uPwcoeNpngf)

------

### 客户端初始化与销毁

```
private RestHighLevelClient restHighLevelClient;

@BeforeEach
void setUp(){
    restHighLevelClient = new RestHighLevelClient(
            RestClient.builder(HttpHost.create("http://192.168.195.131:9200"))
    );
}

@AfterEach
void tearDown() throws IOException {
    if (restHighLevelClient != null) restHighLevelClient.close();
}
```

> 💡 **说明：**
>
> - `RestHighLevelClient` 是操作 ES 的 Java 客户端。
> - 通过 `RestClient.builder(HttpHost.create(...))` 指定 ES 地址。
> - 每次测试后调用 `close()` 释放连接资源。

------

## 二、索引操作（Index APIs）

索引（Index）相当于数据库中的“表”。

### 创建索引并设置 Mapping

```
@Test
void testCreateIndex() throws IOException {
    // 1. 创建索引请求对象
    CreateIndexRequest request = new CreateIndexRequest("items");

    // 2. 设置索引的 Mapping 映射
    request.source(MAPPING_TEMPLATE, XContentType.JSON);

    // 3. 发送请求
    restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
}
```

**Mapping 模板**

```
static final String MAPPING_TEMPLATE = "{\n" +
        "  \"mappings\": {\n" +
        "    \"properties\": {\n" +
        "      \"id\": {\"type\": \"keyword\"},\n" +
        "      \"name\": {\"type\": \"text\", \"analyzer\": \"ik_max_word\"},\n" +
        "      \"price\": {\"type\": \"integer\"},\n" +
        "      \"stock\": {\"type\": \"integer\"},\n" +
        "      \"image\": {\"type\": \"keyword\", \"index\": false},\n" +
        "      \"category\": {\"type\": \"keyword\"},\n" +
        "      \"brand\": {\"type\": \"keyword\"},\n" +
        "      \"sold\": {\"type\": \"integer\"},\n" +
        "      \"commentCount\": {\"type\": \"integer\"},\n" +
        "      \"isAD\": {\"type\": \"boolean\"},\n" +
        "      \"updateTime\": {\"type\": \"date\"}\n" +
        "    }\n" +
        "  }\n" +
        "}";
```

> 💡 **说明：**
>
> - `text`：支持分词（用于全文搜索）。
> - `keyword`：不分词（用于精确匹配，如品牌、分类）。
> - `ik_max_word`：使用中文分词器 IK 最大词粒度模式。
> - `index: false`：表示该字段不参与索引，只能被获取不能被搜索。
> - `date`：ES 的日期类型。

------

### 判断索引是否存在

```
@Test
void testGetIndex() throws IOException {
    GetIndexRequest request = new GetIndexRequest("items");
    boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
    System.out.println("索引是否存在: " + exists);
}
```

------

### 删除索引

```
@Test
void testDeleteIndex() throws IOException {
    DeleteIndexRequest request = new DeleteIndexRequest("items");
    restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
}
```

------

## 三、文档操作（Document APIs）

文档（Document）相当于数据库中的“行数据”。

------

### 新增/更新文档

```
@Autowired
private IItemService itemService;

@Test
void testIndexDoc() throws IOException {
    // 1. 从数据库查询一条商品数据
    Item item = itemService.getById(577967);

    // 2. 将实体类转成 ES 对象（ItemDoc 是 ES 专用对象）
    ItemDoc itemDoc = BeanUtil.copyProperties(item, ItemDoc.class);

    // 3. 创建索引请求并设置文档 ID
    IndexRequest request = new IndexRequest("items").id(item.getId().toString());

    // 4. 设置请求体（JSON 格式）
    request.source(JSONUtil.toJsonStr(itemDoc), XContentType.JSON);

    // 5. 发送请求
    restHighLevelClient.index(request, RequestOptions.DEFAULT);
}
```

> 💡 **说明：**
>
> - 如果 ID 不存在 ⇒ 新增文档。
> - 如果 ID 已存在 ⇒ 覆盖（全量更新）。
<img width="1847" height="886" alt="获取的数据库数据" src="https://github.com/user-attachments/assets/86183aa7-4ee0-4d0e-930c-2497872a4390" />

------

### 查询文档

```
@Test
void testGetDoc() throws IOException {
    // 1. 创建请求对象
    GetRequest request = new GetRequest("items").id("577967");

    // 2. 发送请求
    GetResponse response = restHighLevelClient.get(request, RequestOptions.DEFAULT);

    // 3. 解析响应数据
    String json = response.getSourceAsString();
    ItemDoc itemDoc = JSONUtil.toBean(json, ItemDoc.class);

    System.out.println("itemDoc = " + itemDoc);
}
```

> ⚙️ **说明**：
>
> - `getSourceAsString()` 获取 `_source` 字段中的原始 JSON 数据。
> - 可用 `JSONUtil`（Hutool）反序列化为 Java 对象。

------

### 删除文档

```
@Test
void testDeleteDoc() throws IOException {
    DeleteRequest request = new DeleteRequest("items").id("577967");
    restHighLevelClient.delete(request, RequestOptions.DEFAULT);
}
```

------

### 局部更新文档

```
@Test
void testUpdateDoc() throws IOException {
    // 1. 创建更新请求
    UpdateRequest request = new UpdateRequest("items", "577967");

    // 2. 设置要更新的字段
    request.doc("price", 20000);

    // 3. 发送请求
    restHighLevelClient.update(request, RequestOptions.DEFAULT);
}
```

> 💡 **说明：**
>
> - `UpdateRequest.doc()` 用于部分更新（只修改指定字段）。
> - 如果字段不存在，会被新增。

------

## 四、批量操作（Bulk APIs）

用于一次性执行多条操作（新增/修改/删除），提升效率。

```
@Test
void testBulkDoc() throws IOException {
    // 1. 创建批量请求对象
    BulkRequest request = new BulkRequest();

    // 2. 批量添加操作（可混合增删改）
    request.add(new IndexRequest("items").id("1").source("json", XContentType.JSON));
    request.add(new IndexRequest("items").id("2").source("json", XContentType.JSON));
    request.add(new IndexRequest("items").id("3").source("json", XContentType.JSON));
    request.add(new IndexRequest("items").id("4").source("json", XContentType.JSON));

    // 3. 执行批量请求（只发一次 HTTP）
    restHighLevelClient.bulk(request, RequestOptions.DEFAULT);
}
```

> 💡 **说明：**
>
> - 批量请求可以显著减少网络开销，提高性能。
> - `BulkRequest` 可以同时添加 `IndexRequest`、`UpdateRequest`、`DeleteRequest`。





# Elasticsearch 查询

Elasticsearch 提供了功能强大的 **DSL（Domain Specific Language）查询语法**，用于实现复杂的数据检索。
 DSL 查询语句一般以 `_search` 结尾，通过 JSON 结构描述查询逻辑。

------

## 一、快速查询（match_all）

返回索引库中的所有文档。

```
GET /items/_search
{
  "query": {
    "match_all": {}
  }
}
```

> 🔹说明：
>
> - `match_all` 是最简单的查询，等价于“SELECT * FROM 表”。
> - 常用于测试、分页展示或配合 `sort`、`from`、`size` 使用。
<img width="1026" height="776" alt="快速查询" src="https://github.com/user-attachments/assets/91a83248-1f6e-4502-87b5-c2029870611d" />


------

## 二、叶子查询（Leaf Query）

叶子查询直接作用于某个字段。分为三大类：

------

### 1、全文检索查询（Full Text Queries）

> 用于对 **文本字段（text类型）** 执行搜索，会先**分词**再匹配。

#### （1）`match` 查询

对一个字段执行全文搜索。

```
GET /items/_search
{
  "query": {
    "match": {
      "name": "手机"
    }
  }
}
```

> 🔹说明：
>
> - `match` 会根据字段的分词器（analyzer）将“手机”分成词条后进行匹配。
> - 一般用于搜索描述、标题等文本。

------

#### （2）`multi_match` 查询

对多个字段同时进行全文匹配。

```
GET /items/_search
{
  "query": {
    "multi_match": {
      "query": "手机",
      "fields": ["name", "title"]
    }
  }
}
```

> 🔹说明：
>
> - 当搜索词可能出现在多个字段中时使用。
> - 例如在商品“名称”和“标题”字段中都查找“手机”。
<img width="1726" height="820" alt="全文检索" src="https://github.com/user-attachments/assets/2158249a-1383-44a1-a62f-1bc9e0811ecd" />


------

### 2、精确查询（Term-level Queries）

> 不进行分词匹配，用于对 `keyword`、数值、布尔、日期等字段进行精确过滤。

#### （1）`term` 精确匹配

```
GET /items/_search
{
  "query": {
    "term": {
      "brand": {
        "value": "vivo"
      }
    }
  }
}
```

> ⚠️注意：
>
> - `term` 不会对搜索条件分词。
> - 如果字段是 `text` 类型，会匹配失败（因为 text 已分词）。
> - **解决方法**：使用 `keyword` 字段，如 `brand.keyword`。

------

#### （2）`ids` 查询

根据文档 ID 精确查找。

```
GET /items/_search
{
  "query": {
    "ids": {
      "values": ["1", "2", "3"]
    }
  }
}
```

------

#### （3）`range` 范围查询

常用于数值、时间字段。

```
GET /items/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 1000,
        "lte": 3000
      }
    }
  }
}
```

> 🔹说明：
>
> - `gte`：大于等于
> - `lte`：小于等于
> - 也可以用 `gt`、`lt` 分别表示“>”、“<”。

------

### 2、地理坐标查询（Geo Queries）

> 用于处理地理位置数据，如经纬度搜索。

常见几种方式：

| 类型               | 描述                |
| ------------------ | ------------------- |
| `geo_bounding_box` | 按矩形范围搜索      |
| `geo_distance`     | 按中心点 + 半径搜索 |
| `geo_polygon`      | 按多边形范围搜索    |

示例（按半径范围搜索）：

```
GET /places/_search
{
  "query": {
    "geo_distance": {
      "distance": "5km",
      "location": { "lat": 31.2304, "lon": 121.4737 }
    }
  }
}
```

------

## 三、复合查询（Compound Query）

> 将多个查询条件组合起来。最常用的是 `bool` 查询。

### `bool` 查询语法

```
GET /items/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "手机" } }
      ],
      "should": [
        { "term": { "brand": { "value": "vivo" } } },
        { "term": { "brand": { "value": "小米" } } }
      ],
      "must_not": [
        { "range": { "price": { "gte": 2500 } } }
      ],
      "filter": [
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  }
}
```

| 子句       | 说明                     | 是否参与算分 |
| ---------- | ------------------------ | ------------ |
| `must`     | 必须匹配（相当于 AND）   | ✅            |
| `should`   | 可选匹配（相当于 OR）    | ✅            |
| `must_not` | 必须不匹配（相当于 NOT） | ❌            |
| `filter`   | 必须匹配但不参与评分     | ❌            |

> ✅ **算分（_score）** 表示匹配的相关性。
>  使用 `filter` 可以提高查询性能，因为不需要计算分数。

------

## 四、排序与分页（Sort & Pagination）

一旦启用排序，就不会计算 `_score`（因为相关性已无意义）。

### 1、排序

```
GET /items/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "price": { "order": "desc" } },
    { "create_time": { "order": "asc" } }
  ]
}
```

> 🔹支持多字段排序，从上到下依次优先。
>  🔹排序字段必须是 keyword、数值、日期等可比较类型。

------

### 2、普通分页

通过 `from` 和 `size` 控制。

```
GET /items/_search
{
  "query": { "match_all": {} },
  "from": 0,
  "size": 10,
  "sort": [
    { "price": { "order": "desc" } }
  ]
}
```

| 参数   | 含义                         | 默认值 |
| ------ | ---------------------------- | ------ |
| `from` | 起始位置（从第几个文档开始） | 0      |
| `size` | 返回的文档数                 | 10     |

> ⚠️ **性能注意**：
>
> - 深分页效率较低，例如 `from=10000`。
> - 解决办法：使用 `search_after` 或 `scroll` 查询。

### 3、深度分页（Deep Pagination）

####  背景

在常规分页中，我们使用：

```
GET /items/_search
{
  "from": 0,
  "size": 10
}
```

但是当 `from` 值很大（比如 `from = 10000`）时，性能会**急剧下降**。

------

#### 为什么会慢？

Elasticsearch 在分页时，会先查出 **from + size** 条数据，再丢弃前 `from` 条，只返回后面的结果。

> 例如：
>
> - 你要第 10001～10010 条数据（`from=10000, size=10`）；
> - ES 实际上会先取出 **10010 条文档**；
> - 然后丢掉前面 10000 条；
> - **浪费了大量内存和CPU**。

------

#### 解决方法：`search_after`

`search_after` 用于**深度分页**，是基于“上一页最后一条记录”的排序值来继续往后查的，第二次查询不需要从头开始查了。

它比传统分页快得多，因为不需要跳过前面的结果。

**使用 `search_after` 时，第二次查询不会再从头扫描所有数据，而是直接从“上次最后一条结果”往下继续查。**

------

#### 使用方式

首先要设置排序字段（必须是唯一、可排序的字段，例如 `_id` 或时间戳）：

#### 第一次请求（第一页）

```
GET /items/_search
{
  "size": 3,
  "sort": [
    { "price": "asc" },
    { "_id": "asc" }
  ]
}
```

> 返回结果时，最后一条记录可能长这样：
>
> ```
> "sort": [1000, "A3"]
> ```

------

#### 第二次请求（下一页）

使用上一页最后一条记录的 `sort` 值：

```
GET /items/_search
{
  "size": 3,
  "sort": [
    { "price": "asc" },
    { "_id": "asc" }
  ],
  "search_after": [1000, "A3"]
}
```

> 🔹说明：
>
> - `search_after` 数组的内容必须与 `sort` 字段一一对应；
> - 每次翻页都要记录上一次结果的最后一条的 sort 值；
> - 不能随意跳页，只能“向后翻页”（类似“滑动加载更多”）。

------

### 总结对比

| 方式         | 原理                        | 优点     | 缺点         |
| ------------ | --------------------------- | -------- | ------------ |
| from + size  | 先取 from+size 条，再丢前面 | 简单     | 深页性能差   |
| search_after | 基于上次结果继续查询        | 高效稳定 | 不能随机跳页 |

> 💡 一般建议：
>
> - 普通分页（前几页） → 用 `from + size`
> - 滚动加载、深度翻页 → 用 `search_after`
> - 批量导出 → 用 `scroll`

------

## 五、高亮显示（Highlight）

### 作用

在搜索结果中，将匹配的关键字用指定标签包裹（常用于搜索结果展示中“标红”或“加粗”）。

------

### 基本语法

```
GET /{索引库名}/_search
{
  "query": {
    "match": {
      "搜索字段": "搜索关键字"
    }
  },
  "highlight": {
    "fields": {
      "高亮字段名称": {
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```

> 🔹说明：
>
> - `pre_tags`：高亮前缀标签（通常是 HTML 标签）
> - `post_tags`：高亮后缀标签
> - 可以自定义样式，如 `<span style='color:red'>`。

------

### 示例

假设索引中有字段 `title`，我们想查找“手机”并高亮。

```
GET /items/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  },
  "highlight": {
    "fields": {
      "title": {
        "pre_tags": "<span style='color:red'>",
        "post_tags": "</span>"
      }
    }
  }
}
```

返回结果中会多出一个 `_highlight` 字段：

```
{
  "_source": {
    "title": "小米手机青春版",
    "price": 1999
  },
  "highlight": {
    "title": [
      "小米<span style='color:red'>手机</span>青春版"
    ]
  }
}
```
<img width="1718" height="817" alt="高亮" src="https://github.com/user-attachments/assets/01d28fdc-19d0-4162-917e-54890f184907" />


------

### 多字段高亮

可以对多个字段高亮：

```
"highlight": {
  "fields": {
    "title": {},
    "desc": {}
  }
}
```

------

### 小技巧

| 选项                           | 说明                                       |
| ------------------------------ | ------------------------------------------ |
| `"require_field_match": false` | 允许多个字段同时高亮（否则默认只高亮一个） |
| `"fragment_size"`              | 每个高亮片段的最大字符数                   |
| `"number_of_fragments"`        | 返回多少个高亮片段                         |

例如：

```
"highlight": {
  "require_field_match": false,
  "fields": {
    "title": {
      "fragment_size": 100,
      "number_of_fragments": 3
    }
  }
}
```

# Elasticsearch Java Rest Client 查询笔记

使用 `RestHighLevelClient` 可以在 Java 中操作 Elasticsearch，以下是常见查询场景：

------

## 一、快速查询（Match All 查询）

**示例：**

```
void TestMatchAll() throws IOException {
    // 1. 创建 SearchRequest 对象，指定索引
    SearchRequest request = new SearchRequest("items");

    // 2. 设置查询条件 —— 查询所有文档
    request.source().query(QueryBuilders.matchAllQuery());

    // 3. 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    // 4. 解析响应结果
    parseResponseResult(response);
}
```

**说明：**

- `QueryBuilders.matchAllQuery()`：匹配所有文档。
- 适合用于：测试查询、全量数据预览。

------

## 二、叶子查询（组合条件查询）

**示例：**

```
void testSearch() throws IOException {
    SearchRequest request = new SearchRequest("items");

    request.source().query(
        QueryBuilders.boolQuery()
            .must(QueryBuilders.matchQuery("name", "拉杆箱"))          // 必须匹配 name
            .filter(QueryBuilders.termQuery("brand", "莎米特"))        // 精确过滤 brand
            .filter(QueryBuilders.rangeQuery("price").lt(50000))       // 过滤 price < 50000
    );

    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    parseResponseResult(response);
}
```

**说明：**

- `boolQuery()`：组合多条件查询。
- `must()`：必须匹配（类似 SQL 的 AND）。
- `filter()`：过滤条件，不计算相关性，效率高。
- `rangeQuery()`：范围查询。

📌 **应用场景：**
 电商搜索，比如“品牌 = 莎米特 且 名称包含拉杆箱 且 价格小于50000”。

------

##  三、分页 + 排序查询

**示例：**

```
void testSortAndPage() throws IOException {
    int pageNo = 1, pageSize = 5; // 模拟前端分页参数

    SearchRequest request = new SearchRequest("items");
    request.source().query(QueryBuilders.matchAllQuery());

    // 分页
    request.source().from((pageNo - 1) * pageSize).size(pageSize);

    // 排序：先按销量降序，再按价格降序
    request.source().sort("sold", SortOrder.DESC)
                    .sort("price", SortOrder.DESC);

    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    parseResponseResult(response);
}
```

**说明：**

- `.from()`：起始位置（偏移量）。
- `.size()`：每页条数。
- `.sort(field, order)`：排序字段与顺序。
- 注意：深度分页性能差，可用 `search_after` 代替。

📌 **应用场景：**
 商品列表分页展示、排行榜。

------

## 四、高亮查询（Highlight）

**示例：**

```
void testHighlight() throws IOException {
    SearchRequest request = new SearchRequest("items");

    request.source().query(QueryBuilders.matchQuery("name", "拉杆箱"));

    // 设置高亮
    request.source().highlighter(
        SearchSourceBuilder.highlight()
            .field("name")          // 高亮字段
            .preTags("<em>")        // 高亮前缀
            .postTags("</em>")      // 高亮后缀
    );

    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    parseResponseResult(response);
}
```
<img width="1753" height="634" alt="Java高亮" src="https://github.com/user-attachments/assets/7cfc5a52-2b19-4a8c-b5ad-e9e65514e6c5" />
**说明：**

- 高亮用于在搜索结果中突出关键字。
- `preTags` / `postTags` 用于指定 HTML 标记（可自定义颜色等）。

📌 **应用场景：**
 搜索结果关键字标红显示。

------

## 五、通用结果解析方法
<img width="1344" height="665" alt="数据解析" src="https://github.com/user-attachments/assets/834b1300-7e26-40ef-b357-ac35f219058c" />
<img width="1411" height="720" alt="高亮显示" src="https://github.com/user-attachments/assets/ff7ad787-6154-429c-bf5d-0ae781d847b1" />
**示例：**

```
private static void parseResponseResult(SearchResponse response) {
    // 1. 获取命中结果集
    SearchHits searchHits = response.getHits();
    long total = searchHits.getTotalHits().value;
    System.out.println("total = " + total);

    // 2. 遍历命中文档
    for (SearchHit hit : searchHits.getHits()) {
        // 获取原始 JSON
        String json = hit.getSourceAsString();

        // JSON -> 对象
        ItemDoc doc = JSONUtil.toBean(json, ItemDoc.class);

        // 3. 处理高亮字段
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        if (highlightFields != null && !highlightFields.isEmpty()) {
            HighlightField hf = highlightFields.get("name");
            if (hf != null) {
                String hfName = hf.getFragments()[0].string();
                doc.setName(hfName); // 替换为高亮内容
            }
        }

        System.out.println("doc = " + doc);
    }
}
```

**说明：**

- `SearchHits`：包含所有匹配结果。
- `getSourceAsString()`：取 `_source` 内容（JSON格式）。
- `HighlightField`：高亮内容集合。
- `BeanUtil.toBean()`：把 JSON 转换成 Java 对象。


