---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/Python 常用数据库库/","noteIcon":"","created":"2024-03-14T13:12:24.823+08:00","updated":"2024-04-24T01:01:34.112+08:00"}
---

## SQLAlchemy VS Tortoise ORM

都是流行的 Python ORM (对象关系映射) 库，它们提供了将数据库表映射到 Python 类的功能，以便使用面向对象的方式操作数据库。尽管它们的目标相同，但它们在设计理念、性能、易用性等方面各有特点。

### SQLAlchemy

1. **成熟度和社区支持**：SQLAlchemy 存在已久，社区庞大，提供了大量的文档和第三方扩展。它被广泛应用于各种规模和类型的项目中。
2. **灵活性和功能性**：SQLAlchemy 提供了两种主要使用方式：核心（使用 SQL 表达式语言）和 ORM（使用类映射到数据库表）。这种设计提供了极高的灵活性，允许开发者根据需要选择最合适的抽象层级。
3. **同步和异步**：SQLAlchemy 从 1.4 版本开始引入了对异步数据库操作的支持，但它的同步操作仍然是其核心功能之一。
4. **性能**：由于其提供了丰富的功能和灵活的使用方式，可能在某些情况下性能不如一些更轻量级的ORM库。但对于需要复杂查询和大量数据库操作的应用来说，其性能已经足够优秀。

### [Tortoise ORM](ORM/Tortoise%20ORM.md)

1. **异步设计**：Tortoise ORM 是为现代异步Python应用设计的，如使用 Asyncio。它在设计上就考虑到了异步操作，这使得它特别适合构建异步应用。
2. **简洁性和易用性**：Tortoise ORM 提供了更简洁的API和较少的配置，使得快速开发变得更加容易。它的目标是提供一个易于使用的ORM，同时保持足够的功能。
3. **社区和成熟度**：虽然 Tortoise ORM 的社区相对较小，但它正在快速增长，并且已经被多个项目采用。不过，相比于 SQLAlchemy，它的成熟度和文档可能略显不足。
4. **性能**：Tortoise ORM 由于其异步性质，可以在处理大量并发数据库操作时提供很好的性能。但在同步环境下，其性能可能不如专门的同步ORM库。

### 综合比较

- **选择SQLAlchemy的理由**：需要高度的灵活性和控制、有复杂的数据库操作需求、项目中同时需要同步和异步操作或者依赖一个大型社区和丰富的文档。
- **选择Tortoise ORM的理由**：构建全异步的应用、追求简洁和易用性、需要良好的并发性能。

在选择 ORM 时，考虑项目的具体需求是很重要的。如果你的项目是基于 FastAPI 这样的现代异步框架，可能会更倾向于选择 Tortoise ORM，以保持整个应用的异步性。然而，如果你需要更复杂的数据库操作或者更广泛的社区支持，SQLAlchemy 仍然是一个非常好的选择。

## Tortoise ORM
### 外键
```Python
from tortoise import fields
from tortoise.models import Model

class Tournament(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(max_length=100)

class Team(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(max_length=100)
    tournament = fields.ForeignKeyField('models.Tournament', related_name='teams')
```

上面的示例中，`Tournament` 模型代表一个锦标赛，而 `Team` 模型代表参加锦标赛的队伍。`Team` 模型中的 `tournament` 字段是一个 `ForeignKeyField`，它指向 `Tournament` 模型，表示每个队伍参加一个特定的锦标赛。`related_name='teams'` 参数指定了从 `Tournament` 实例访问其相关 `Team` 实例的属性名称。如果没有明确指定关联表的引用字段，默认情况下它会引用关联表的主键。

使用 related_name 的示例：如果想获取某个特定锦标赛的所有队伍，可以这样做：

```Python
tournament = await Tournament.get(id=tournament_id) 
teams = await tournament.teams.all()
```

`tournament.teams` 利用了 `related_name` 属性来访问所有与该锦标赛关联的队伍实例。`.all()` 是异步查询方法，用于获取所有相关的队伍记录。

## index 索引

在 Tortoise ORM 中，索引（Index）是用来提高数据库查询效率的一种数据库对象。正确地使用索引可以显著加快查询速度，尤其是在处理大量数据时。以下是一些关于如何在 Tortoise ORM 中设置索引字段的技巧：

### 1. 单字段索引

在定义模型字段时，可以通过设置 `index=True` 来为该字段创建索引。这适用于那些经常作为查询条件的字段。

```python
from tortoise import fields, models

class User(models.Model):
    username = fields.CharField(max_length=50, index=True)
    email = fields.CharField(max_length=100, unique=True)  # unique 也会创建索引
```

在这个例子中，`username` 字段被设置为索引，这有助于加速基于 `username` 的查询操作。

### 2. 复合索引

如果你的查询条件经常涉及多个字段，那么创建一个涵盖这些字段的复合索引可能会更有效。在 Tortoise ORM 中，你可以在模型的 Meta 类中使用 `indexes` 列表来定义复合索引。

```python
class Event(models.Model):
    name = fields.CharField(max_length=50)
    date = fields.DateField()
    city = fields.CharField(max_length=50)

    class Meta:
        indexes = [("date", "city")]  # 复合索引
```

这个复合索引有助于加速同时基于 `date` 和 `city` 的查询。

### 3. 使用场景

- **选择性高的字段**：对于具有高选择性（即字段值具有高度唯一性）的字段，索引特别有用。
- **查询操作多于写入操作**：索引可以加快查询速度，但每次数据变更（插入、更新、删除）时，索引也需要更新，这可能会减慢写入操作。因此，在频繁查询的字段上使用索引，在频繁更新的字段上谨慎使用索引。

### 4. 避免过度索引

虽然索引能提高查询效率，但每个索引都需要占用额外的存储空间，并且在数据变更时需要维护。过多的索引可能会导致数据库性能下降，特别是在插入和更新操作上。因此，应该只为那些经常用作查询条件的字段创建索引。

### 5. 使用数据库迁移工具管理索引

Tortoise ORM 支持数据库迁移，这意味着你可以使用迁移工具来添加、修改或删除索引，而不需要直接操作数据库。这有助于保持数据库结构的版本控制和历史记录。

通过合理使用索引，你可以优化 Tortoise ORM 应用的数据库查询性能，提高应用的响应速度和用户体验。在设计数据库模型和选择索引字段时，考虑实际的应用场景和查询模式至关重要。

分区：Tortoise ORM 本身不直接支持创建分区。