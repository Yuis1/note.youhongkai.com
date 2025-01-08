---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/数据交互/Drupal数据接入及采集/","tags":["Drupal"],"noteIcon":"","created":"2024-03-04T00:16:32.726+08:00","updated":"2024-04-23T23:01:30.413+08:00"}
---


作者: 游鱼思

---
## 模块对比

在Drupal中，如果你想对接外部API以引入外部数据源作为内容类型或实体（entity），有几个模块可以帮助你实现这一功能。以下是一些主要的模块和工具：

### **Feeds**

- [Feeds](https://www.drupal.org/project/feeds) 模块允许你导入或聚合内容到Drupal中。它可以从不同的来源获取数据，包括RSS/Atom feeds、json、CSV文件、XML文件等，并将这些数据导入为节点、用户、taxonomy terms或其他实体。Feeds模块通过可扩展的架构支持多种数据源和格式，是引入外部数据的强大工具。
- Feeds is the module for _importing_ or _aggregating_ data into nodes, users, taxonomy terms and other content entities using a web interface without coding a migration. Data can be imported from various formats, such as CSV, [JSON](https://www.drupal.org/project/feeds_ex "Requires the Feeds Extensible Parsers module"), [XML](https://www.drupal.org/project/feeds_ex "Requires the Feeds Extensible Parsers module") and RSS feeds.

### **Migrate API**

- Drupal的[Migrate API](https://www.drupal.org/docs/8/api/migrate-api/migrate-api-overview)（在Drupal 8及以上版本内建）为数据迁移提供了框架。你可以使用Migrate API来迁移外部数据源到Drupal中，创建内容、用户、分类术语等。Migrate API是高度可定制的，支持复杂的迁移逻辑和数据转换。

手册：[Migrate API | Drupal APIs | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/drupal-apis/migrate-api)

### **Migrate Plus**

- [Migrate Plus](https://www.drupal.org/project/migrate_plus) 模块提供了Migrate API的扩展功能，包括对JSON和XML数据源的支持、数据转换插件等。它可以帮助你更容易地从复杂的外部API导入数据。

### **Migrate Source JSON**

- 对于特定需要从JSON数据源导入数据的场景，[Migrate Source JSON](https://www.drupal.org/project/migrate_source_json)模块提供了一个简单的解决方案。这个模块扩展了Migrate API，允许你直接从JSON格式的数据源迁移数据。

### **Migrate Tools**

- [Migrate Tools](https://www.drupal.org/project/migrate_tools)模块为Migrate API提供了一些有用的工具和命令行界面，使数据迁移过程更加方便。这包括执行迁移、回滚迁移以及查看迁移状态的命令。
-  migrate:status - Lists migrations and their status.
- migrate:import - Performs import operations.
- migrate:rollback - Performs rollback operations.
- migrate:stop - Cleanly stops a running operation.
- migrate:reset-status - Sets a migration status to Idle if it's gotten stuck.
- migrate:messages - Lists any messages associated with a migration import.
- migrate:fields-source - List the fields available for mapping in a source
- migrate:tree - Shows a tree of migration dependencies

A **Web UI** provides a front-end equivalent to many of the drush commands.

### [Entity Importer](https://www.drupal.org/project/entity_import)

provides a user interface for site administrators to import various entities. At the moment the module only support CSV files as the source, but due to the plugin type architecture this can easily be expanded to other file formats. There are also plans in the road-map to support other common source types moving forward.

示例：[Importing Data from a REST API with Entity Import and Migrate in Drupal 8 | Aten Design Group](https://atendesigngroup.com/articles/importing-data-rest-api-entity-import-and-migrate-drupal-8)

### **HTTP Client Manager**

- [HTTP Client Manager](https://www.drupal.org/project/http_client_manager) 是一个高级的HTTP客户端，为开发者提供了一个简化的界面来消费外部HTTP服务，比如RESTful API。结合自定义模块开发，可以使用它来集成外部数据。

根据你的具体需求和数据源的类型（如RESTful API、CSV文件、XML或JSON数据等），你可以选择最适合你的模块。一般来说，Migrate系列的模块和Feeds模块非常适合批量导入外部数据，而HTTP Client Manager和RESTful Web Services模块则更适合与外部API进行实时交互。

#### 主要特点

- **封装 Guzzle**：HTTP Client Manager 模块封装了 Guzzle，提供了一个更符合 Drupal 开发习惯的接口。
- **配置管理**：允许通过 Drupal 的配置系统管理 HTTP 客户端的配置，如基础 URI、默认请求头、超时设置等。
- **日志和错误处理**：集成了 Drupal 的日志系统，便于跟踪 HTTP 请求和响应，以及处理错误。
- **可扩展性**：支持自定义中间件和事件订阅者，为请求/响应流程提供额外的处理逻辑。

#### 使用场景

HTTP Client Manager 模块适用于需要与外部服务进行交互的 Drupal 网站。例如，如果您的网站需要从第三方 API 拉取数据、提交表单数据到外部系统或集成外部服务，HTTP Client Manager 可以简化这些任务的开发工作。

说明文档：[HTTP Client Manager | Contributed module documentation | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/extending-drupal/contributed-modules/contributed-module-documentation/http-client-manager)

视频介绍：[Drupal 8 Http Client Manager - YouTube](https://www.youtube.com/watch?v=2zITl5Ka264)

### [Web service client](https://www.drupal.org/project/wsclient) D7

 provides an interface to consume external web services. It focuses on integrating web service operations as [Rules](https://www.drupal.org/project/rules) actions, but can also be used as data provider for other modules.

### [External Data Source](https://www.drupal.org/project/external_data_source) D10

Feeds 模块解决的列表的批量导入，这个模块解决的是已经有了数据条目后，字段值的外部导入。

Using external data source in Drupal as Microsevice architecture is a best practice. By using this module we will be able to create a new data source as a plugin that we can find it when we configure the field type "external data source field" .

比较遗憾的是，只支持字段，而不是整个实体。

在D10中有Bug，反正我是用不了，添加字段第二步就出错了，也查不到出错原因。

### [External Entities](https://www.drupal.org/project/external_entities)

将外部的数据集，转变为Drupal的 Entity

### [PHPExcel](https://www.drupal.org/project/phpexcel)

The PHPExcel module allows developers to export/import data to/from real Excel files.

[Accessing RDF from other sites | Drupal.org](https://www.drupal.org/node/1225782)

A SPARQL endpoint is a conformant SPARQL protocol service as defined in the

SPROT specification. A SPARQL endpoint enables users (human or other) to query

a knowledge base via the SPARQL language. 

[Resource Description Framework (RDF) | Drupal.org](https://www.drupal.org/project/rdf)

数据处理及展示

[SPARQL Entity Storage | Drupal.org](https://www.drupal.org/project/sparql_entity_storage)   10

[RDF entity | Drupal.org](https://www.drupal.org/project/rdf_entity)  10

[Views Datasource | Drupal.org](https://www.drupal.org/project/views_datasource)   7

Views Datasource is a set of plugins for [Drupal Views](http://drupal.org/project/views) for rendering content in a number of shareable, reusable formats based on XML, JSON and XHTML. These formats allow content in a

## Migrate

- **核心集成**：从Drupal 8开始，Migrate API已经成为Drupal核心的一部分，为数据迁移提供了强大的框架。
- **用途**：Migrate主要设计用于一次性的数据迁移任务，如从旧站点迁移到Drupal，或者在Drupal的不同版本间迁移内容。
- **灵活性和复杂性**：Migrate API提供了极高的灵活性，允许开发者编写复杂的迁移逻辑，包括数据转换和映射。这使得Migrate非常强大，但相对来说，学习曲线较陡峭。
- **扩展性**：通过额外的模块，如Migrate Plus、Migrate Tools等，Migrate API的功能可以得到扩展，支持更多类型的数据源和更复杂的迁移场景。
- **适用场景**：适用于大规模的数据迁移项目，需要处理复杂的数据映射和转换。

## Feeds

- **模块需求**：Feeds是一个贡献模块，需要单独安装。
- **用途**：Feeds模块主要设计用于定期导入内容，如从外部RSS源、CSV文件或其他类型的数据源定期更新或导入内容到Drupal中。
- **易用性**：Feeds模块通过提供图形用户界面（GUI）简化了导入流程，使得非开发者也可以配置和运行数据导入任务。
- **灵活性**：虽然Feeds提供较好的灵活性，但相对于Migrate，它在处理极其复杂的数据映射和转换方面能力有限。
- **适用场景**：适用于需要定期从外部数据源导入内容的场景，特别是当数据结构相对简单直接时。
Feeds 说明文档  [Feeds | Drupal.org](https://www.drupal.org/documentation/modules/feeds)   [Feeds | Contributed module documentation | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/contributed-modules/feeds)


### Feeds 生态

[Ecosystem modules for Feeds | Drupal.org](https://www.drupal.org/project/feeds/ecosystem)

1， Feeds模块， 这个是主模块，它是用来导入数据，开始是收割RSS数据，后来发展为导入各种数据，再后来，我们发现，基于这个模块，可以用来采集网页信息。  
2， job_scheduler D10，这个是Feeds模块依赖的插件，采集时的任务调度，很多时候需要依赖于这个模块。  
3， feeds_tamper D10，这个模块，用来在导入数据的时候，对数据进行预处理、也就是清理工作，非常好用的帮助模块。采集网页数据时，必备模块。  
4， feeds_xpathparser D10，这个模块使得我们可以使用Xpath规则，解析数据，网页采集过来的数据，是HTML格式的，需要使用Xpath规则解析。这个也是必备模块之一。

5， feeds_crawler 模块  D7，这是一个小爬虫，方便采集各种分页列表，非常好用，网页抓取必备模块。

6， feeds_smartparser 模块  D7 ，从HTML页面中，智能提取全文，网页采集必备模块之一。  
7， [Feeds Self Node Processor](https://www.drupal.org/project/feeds_selfnode_processor)  D7 ，采集过来的节点本身，还是一个Feed种子，通过HTTP请求，抓取更详细的信息，完善自身节点，网页采集必备模块之一。  You may need this is if your data source has remote data in both a list and in more detail in its main display.

8， Views/Ctools views data export，将网页采集过来的数据，导出成各种格式，支持XML，CSV，Excel。  
9， feeds_spider  D7 ，采集蜘蛛，类似于feeds_crawler，网页采集模块之一。只支持D7

[Feeds Fetcher POST](https://www.drupal.org/project/feeds_fetcher_post)   D10   This module adds a new Fetcher type plugin to use in the Feeds module. You may use this to fetch a Download URL (i.e. JSON/XML) and specify custom parameters before getting successful responses on POST requests.

[JSON Feed | Drupal.org](https://www.drupal.org/project/json_feed)

背景知识： [Json 查询语法](../../前端开发/Json%20查询语法.md)

### 总结

总的来说，D10 时代 Feeds 模块没落了。不仅安装困难，而且少了这些贡献模块后，Feeds对于爬取是半吊子支持，很难满足各种需求。

最大的痛点是，只能从列表API导入，并且还不能带分页等变量，也无法再次从详情API导入。

很少有被爬的网站会乖乖的把所有详情都塞在列表API里，还不分页吐给你，那简直是大善人。  
卸载了吧，没啥用。

- 如果你的项目涉及到一次性的大规模数据迁移，特别是涉及复杂数据映射和转换，推荐使用**Migrate**。
- 如果你的需求是定期从外部源导入内容到Drupal，并且希望通过用户界面来配置导入任务，那么**Feeds**可能是更合适的选择。


### Feeds安装踩坑记

PHP8.3, Drupal 10.2 ，Feeds **[8.x-3.0-beta4](https://www.drupal.org/project/feeds/releases/8.x-3.0-beta4 "View release notes")**

先手工安装了一遍feeds，发现无法创建 feed type，应该是缺少库。

卸载后，删除了模块文件夹，又通过composer安装了一遍。这下库是齐了，但是整站也无法打开了。

提示： The website encountered an unexpected error. Try again later.

侦查过程：

- settings.php 加入配置 $config['system.logging']['error_level'] = 'verbose';   // 显示所有消息，包括调试信息。
- 通过composer安装后，vendor/laminas  下面的库是齐全的，但feeds还是提示缺失库。
- 通过 drush watch:show 定位问题，发现是模块引用被指向了  web/modules 目录，而通过composer安装的模块却是在 web/modules/contrib 下。

查找资料：

 [The website encountered an unexpected error. [#3383226] | Drupal.org](https://www.drupal.org/project/feeds/issues/3383226) ，还需要使用 https://www.drupal.org/project/ludwig 来管理三方库。但是 ludwig 和 composer 又有冲突。

折腾过程：

1、尝试了手工安装：在web/modules/feeds，拷贝 vendor/laminas 到  web/libraries/laminas ，无效。  
2、在 web/modules/contrib 、 web/modules 两个目录下，都复制一份 feeds 文件夹，依次使用:

```bash
composer remove 'drupal/feeds'
drush cr
drush pmu feeds
```

提示卸载成功后，再手工去以上两个目录删除残留文件。

再通过composer安装一遍，且还要将 web/modules/contrib/feeds 移动到 web/modules/feeds 下

还是提示各种错误和缺失库。

卸载：通过 composer remove drupal/ludwig 、 drush pmu feeds、 drush cedit core.extension 并移除feeds  这些方案都试过，无法完全卸载，导致Drupal后台许多页面无法访问。

还得手工创建表，解决drupal后台无法访问的问题：

```sql
CREATE TABLE `feeds_feed` (
  `fid` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'The feed ID.', 
  `feed_type` varchar(128) NOT NULL COMMENT 'The feed type.',
  `bundle` varchar(128) NOT NULL COMMENT 'The feed bundle.', 
  `config` longblob COMMENT 'The feed configuration.',
  `created` int(11) NOT NULL COMMENT 'The Unix timestamp when the feed was created.',
  `updated` int(11) NOT NULL COMMENT 'The Unix timestamp when the feed was last updated.',
  PRIMARY KEY (`fid`),
  UNIQUE KEY `feed_bundle` (`bundle`),
  KEY `feed_type` (`feed_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Stores information about feeds.';

CREATE TABLE `feeds_subscription` (
  `fid` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'The subscription ID.', 
  `feed_id` int(10) unsigned NOT NULL COMMENT 'The feed ID.',
  `uid` int(10) unsigned NOT NULL DEFAULT '0' COMMENT 'The user ID subscribed to this feed.',
  `created` int(11) NOT NULL COMMENT 'The Unix timestamp when the subscription was created.',
  PRIMARY KEY (`sid`),
  UNIQUE KEY `feed_uid` (`feed_id`,`uid`),
  KEY `uid` (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Stores user subscriptions to feeds.';
```

然后在Drupal管理后台成功卸载Feeds。

3、手工安装feed到 web/modules/feeds，通过composer安装 ludwig，然后访问 /admin/reports/packages ，ludwig 自动下载了缺失的库。 ——也不行，还是提示 库 缺失。

#### 最终解决

按照以上步骤卸载 feeds 后 。

降级到PHP 8.1（可能非必须）

手工安装feed到 web/modules/feeds

运行composer获取依赖库：composer require laminas/laminas-feed

成功。

#### 错误排查

**添加Feeds时报错**

_Error_: Call to undefined function ini_restore() in _Laminas\Feed\Reader\Reader::detectType()_ (line _494_ of _/www/wwwroot/rc_juhe_drupal/vendor/laminas/laminas-feed/src/Reader/Reader.php_).  
解决办法：PHP环境的 ini_restore() 函数被禁用了，开启一下即可  

**添加Feeds时，提示 Invalid feed URL.**

解决办法：在Feed Type的设置中，关闭 "Auto detect feeds 自动检测feeds"

### Feeds 使用

需要吐槽一下，Feeds的使用比较抽象和反直觉。

第一步：先在 结构 - Feed Types (/admin/structure/feeds) 中，定义好需要爬取的数据模型和映射。这一步竟然不让添加爬取地址，这是最反直觉的地方。 会用到 JsonPath 等语法。

第二步：在 内容 - Feeds ( admin/content/feed ) 中，添加具体的爬取地址。开始爬取。

Feeds在Mapping 时，会出现一些让人困惑的术语，以下是解释：  

1. **Feed: 作者** - 这通常指的是Feed条目的原始作者。在RSS或Atom feed中，可能会有一个明确的作者字段。
    
2. **Feed: 发布于** - 指Feed条目首次公开发布的日期和时间。
    
3. **Feed: 已更改** - 指Feed条目最后一次被更新或修改的日期和时间。
    
4. **Feed: 种子ID** - 一个唯一标识符，用于区分Feed中的每个条目。
    
5. **Feed: Feed type** - Feeds模块允许你创建不同类型的Feed导入器。其实就是 admin/structure/feeds 下的条目。
    
6. **Feed: Importing status** - 指当前Feed导入的状态，例如，是否正在导入中，导入是否完成等。
    
7. **Feed: Items imported** - 指在上一次导入过程中成功导入的条目数量。
    
8. **Feed: Last import** - 指最近一次Feed导入操作的日期和时间。
    
9. **Feed: Next import** - 如果设置了自动定时导入，这将显示下一次计划导入的日期和时间。
    
10. **Feed: 已排入队列** - 指示有多少Feed条目在当前等待被导入。
    
11. **Feed: 源** - Feed的URL或源地址，即数据原始所在位置。
    
12. **Feed: 标题** - 指Feed条目的标题。
    
13. **Feed: UUID** - Universal Unique Identifier（全局唯一ID）

#### 视频教程

[Feeds to import content by CSV file Drupal 10, Drupal 9, Drupal 8 | D4Drupal - YouTube](https://www.youtube.com/watch?v=rINC4rhthsY)

[Using Feeds to integrate complex business data into Drupal: Pittsburgh 2023 - YouTube](https://www.youtube.com/watch?v=sq5Q52JZcl4)

[Getting Started with #Drupal Feeds: A Beginner's Data Import Guide | TECHX | SPECBEE - YouTube](https://www.youtube.com/watch?v=kY0jxypffLw)
