---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/数据交互/Migrate 模块使用/","noteIcon":"","created":"2024-06-22T22:30:28.569+08:00","updated":"2024-04-23T23:05:11.000+08:00"}
---


作者: 游鱼思

---
## 简介

Migrate 模块需要编写自定义代码，用到命令行。UI功能很有限。

手册：[Migrate API | Drupal APIs | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/drupal-apis/migrate-api)

[Migrating data from XML, JSON or SOAP source | Migrate source plugins | Drupal Wiki guide on Drupal.org](https://www.drupal.org/docs/8/api/migrate-api/migrate-source-plugins/migrating-data-from-xml-json-or-soap-source)

[Pull Content From a Remote Drupal 8 Site Using Migrate and JSON API | Lullabot](https://www.lullabot.com/articles/pull-content-from-a-remote-drupal-8-site-using-migrate-and-json-api)

视频教程：

[Migrate from JSON file with Migration API in Drupal 10 | JSON Migration import | JSONMigration - YouTube](https://www.youtube.com/watch?v=pi9w9MPd1zw)

## 对Node字段进行更新
### 使用场景

通过migrate 模块，将Node中的字段从外部json进行更新。其中，node中有一个uid，需要传递为外部json的url参数，从而拿到该node的详情。

要通过Drupal Migrate模块从外部JSON更新Node中的字段，并且需要将Node中的`uid`字段作为URL参数传递以获取特定Node的详细信息，您需要执行以下步骤来实现这个需求。这个过程大致包括创建自定义迁移模块、编写迁移配置文件、以及可能需要编写一些自定义代码来处理特定的数据获取和转换逻辑。

### 步骤 1: 创建自定义迁移模块

1. **创建一个新的Drupal模块**：如果你还没有一个自定义模块来放置迁移代码，你需要创建一个。在你的Drupal安装的`modules/custom`目录下创建一个新目录，比如叫`my_migrate_module`。
2. **模块基本文件**：在`my_migrate_module`目录中，创建`my_migrate_module.info.yml`和`my_migrate_module.module`文件。`.info.yml`文件中定义模块的基本信息。

### 步骤 2: 编写迁移配置文件

1. **创建迁移YAML文件**：在你的模块目录下，创建一个`config/install`目录。然后，创建迁移的YAML文件，例如`migrate_plus.migration.my_custom_migration.yml`。这个文件将包含迁移的具体配置。
2. **定义源**：在迁移配置文件中，你需要定义数据源。使用`urls`参数获取外部JSON数据，并使用Node中的`uid`作为参数。这可能需要自定义插件来动态构建URL。

```yaml
source:
  plugin: url
  data_fetcher_plugin: http
  data_parser_plugin: json
  urls: 'https://example.com/api/details?uid=[uid]'
  # 可能需要自定义source插件来处理动态URL。
  fields:
    - name: uid
      label: 'Unique Identifier'
    - name: title
      label: 'Title'
    - ...
  ids:
    uid:
      type: integer
```

这里的`[uid]`是个占位符，表示你可能需要一种方式来动态替换这个值。由于标准的迁移插件不支持这种动态URLs，你可能需要创建一个自定义的source插件来实现。

### 步骤 3: 定义目的地和处理过程

1. **目的地**：配置迁移的目的地类型，这里是更新现有的Node。

```yaml
destination:
  plugin: 'entity:node'
```

2. **处理过程**：在迁移过程中指定如何映射源字段到目的地字段。可能还需要使用处理插件来转换数据格式或值。

```yaml
process:
  title: title
  field_custom: source_field_name
  # 可能需要进一步的处理逻辑
```

### 步骤 4: 执行迁移

1. **安装模块**：确保你的自定义模块已经启用。
2. **执行迁移**：使用Drush命令执行迁移。

```shell
drush migrate:import my_custom_migration
```


## 定期获取

要实现定期自动化获取数据的功能，如每隔6小时更新Drupal节点，你可以利用Drupal的Cron系统或外部的任务调度器。以下是两种常见的方法：

### 方法 1: 使用Drupal Cron

Drupal内建的Cron系统可以用来定期执行任务。你可以将迁移任务配置为Cron任务，但这通常需要编写一些自定义代码来触发迁移过程。

1. **实现`hook_cron`**：在你的自定义模块中实现`hook_cron()`。在这个hook中，你可以使用Migrate Tools提供的服务来触发迁移。

```php
/**
 * Implements hook_cron().
 */
function my_migrate_module_cron() {
  // 检查上次运行时间，防止重复执行。
  $last_run = \Drupal::state()->get('my_migrate_module_last_cron_run', 0);
  $interval = 6 * 60 * 60; // 6小时
  $current_time = \Drupal::time()->getRequestTime();
  if (($current_time - $last_run) >= $interval) {
    // 执行迁移
    $migration_ids = ['my_custom_migration'];
    $migration_plugin_manager = \Drupal::service('plugin.manager.migration');
    foreach ($migration_ids as $migration_id) {
      if ($migration = $migration_plugin_manager->createInstance($migration_id)) {
        $executable = new \Drupal\migrate_tools\MigrateExecutable($migration, new \Drupal\migrate\MigrateMessage());
        $executable->import();
      }
    }
    // 更新上次运行时间
    \Drupal::state()->set('my_migrate_module_last_cron_run', $current_time);
  }
}
```

2. **配置Cron**：确保Drupal的Cron设置为每小时运行。在管理界面中，访问`/admin/config/system/cron`来配置。请注意，这取决于外部的Cron触发器（如服务器Cron作业或在线Cron服务）或Drupal的内部触发器是否频繁调用。

### 方法 2: 使用外部Cron服务

如果你需要更精确的控制执行时间，或者Drupal的内部Cron不适合你的用例，你可以使用外部的Cron作业来触发迁移。

1. **设置外部Cron**：在你的服务器上设置一个Cron作业，或者使用在线Cron服务。这个Cron作业将定期执行一个命令或访问一个URL来触发迁移过程。
2. **使用Drush**：如果你的环境中安装了Drush，外部Cron作业可以直接执行Drush命令来启动迁移。例如，Cron命令可以是：

```bash
0 */6 * * * /path/to/drush -r /path/to/drupal/root migrate:import my_custom_migration
```

这个Cron作业每6小时执行一次迁移。

3. **使用HTTP请求**：如果你使用在线Cron服务或无法直接运行Drush，可以创建一个自定义路由和控制器在Drupal中，当访问这个路由时触发迁移。然后，让Cron服务请求这个URL。

```php
// 在my_migrate_module.routing.yml中定义路由
my_migrate_module.migrate_trigger:
  path: '/migrate/trigger'
  defaults:
    _controller: '\Drupal\my_migrate_module\Controller\MigrateController::triggerMigration'
    _title: 'Trigger Migration'
  requirements:
    _permission: 'access content'
```

请注意，暴露一个触发迁移的URL可能带来安全风险。确保适当保护这个路由，例如通过IP限制、密钥验证等。