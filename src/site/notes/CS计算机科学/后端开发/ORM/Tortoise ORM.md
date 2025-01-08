---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/ORM/Tortoise ORM/","noteIcon":"","created":"2024-04-27T02:55:06.000+08:00","updated":"2024-04-25T15:45:35.000+08:00"}
---


作者：游鱼思

资料整理+踩坑笔记

---

## 参考资料

[Tortoise ORM：2.单表常用操作 (qq.com)](https://mp.weixin.qq.com/s/JB81ueFowRUOOym5la6v_g)

[Tortoise ORM：3.一对一关系 (qq.com)](https://mp.weixin.qq.com/s/QJsE29Qd3_ZCjRTjI6nBaA)

[Tortoise ORM：4.一对多关系 (qq.com)](https://mp.weixin.qq.com/s/NOTL2bec0NGAcFZGDMzTIA)

[Tortoise ORM：5.多对多关系 (qq.com)](https://mp.weixin.qq.com/s/mzQsGwUeaAmroGzdCXZWYA)

## 数据库连接和模式生成

使用`Tortoise.init()`和`await Tortoise.generate_schemas()`进行数据库连接和模式生成的方式，与使用`register_tortoise`函数为FastAPI应用注册Tortoise ORM有一些区别。主要区别在于控制级别、集成度、以及使用场景：

### 使用`Tortoise.init()`的方式

1. **直接控制**：通过`Tortoise.init()`直接初始化数据库连接和配置，给开发者提供了更精细的控制能力。你可以在任意位置调用`Tortoise.init()`来初始化数据库连接，这在一些特定场景（如脚本执行或特定任务中）很有用。
2. **手动生成模式**：使用`await Tortoise.generate_schemas()`手动控制何时生成数据库模式。这可以确保在数据库操作之前，所有的表结构都已正确生成。这种方式适用于需要精确控制数据库表生成时机的场景。
3. **灵活性**：这种方式允许在不同的环境或时机下灵活地初始化数据库。例如，在FastAPI应用之外的异步脚本中使用Tortoise ORM。
4. **缺少自动化**：相较于`register_tortoise`，这种方式缺少了一些自动化处理，如自动关闭数据库连接等。开发者需要更加注意资源管理和错误处理。

### 使用`register_tortoise`的方式

`register_tortoise_app(app)`函数是在FastAPI应用中注册Tortoise ORM的关键步骤，它起到几个重要的作用和机制：

1. **数据库连接和配置的绑定**：通过这个函数调用，Tortoise ORM 的配置信息（包括数据库连接信息和模型位置等）会被绑定到FastAPI应用上。这样，当应用启动和停止时，Tortoise ORM可以自动地处理数据库连接的开启和关闭。
2. **自动生成数据库模式**：如果配置了`generate_schemas=True`，Tortoise ORM会在应用启动时自动根据定义的模型生成数据库表结构。这意味着你不需要手动运行迁移脚本来创建或更新数据库表，Tortoise ORM会处理这些工作。
3. **异常处理器的添加**：通过设置`add_exception_handlers=True`，Tortoise ORM会向FastAPI应用添加一些异常处理器，这些处理器能够捕获数据库操作中可能出现的异常，并以HTTP异常的形式返回给客户端。这样可以提高应用的健壮性，并且减少了开发者处理常见数据库错误的工作量。
4. **中间件的集成**：虽然不是直接通过`register_tortoise`函数实现的，但Tortoise ORM的工作方式通常涉及到异步数据库操作。在FastAPI中，这需要正确地设置和管理异步上下文。通过注册Tortoise ORM，你可以确保数据库会话和事务在每个请求的生命周期内被正确地处理。
5. **便于项目的扩展和维护**：将Tortoise ORM注册到FastAPI应用中，可以让你的项目结构更加清晰，数据库操作与API逻辑分离，有利于项目的扩展和维护。

`register_tortoise_app(app)`是将Tortoise ORM整合到FastAPI应用的桥梁，确保了数据库操作的正确性、便捷性和高效性。这样的设计让开发者可以更加专注于业务逻辑的实现，而不用担心数据库层面的细节。

优点：

1. **集成度高**：`register_tortoise`函数专为FastAPI设计，它将Tortoise ORM的配置和初始化过程集成到FastAPI的生命周期中。这意味着数据库连接的开启和关闭会自动管理，与FastAPI应用的启动和停止同步。
2. **自动生成数据库模式**：通过配置参数`generate_schemas=True`，可以在应用启动时自动创建或更新数据库模式。这减少了手动维护数据库模式的需要。
3. **异常处理**：通过`add_exception_handlers=True`，自动为应用添加数据库相关的异常处理器。这提高了应用的健壮性和开发效率。
4. **适合Web应用**：这种方式更适合于Web应用的开发场景，尤其是当使用FastAPI作为后端框架时。它简化了配置和管理过程，让开发者可以专注于业务逻辑。


## Model定义

在Tortoise ORM的模型定义中，你可以使用以下四个参数来定义字段的行为：

1. `null=True`: 这个参数表示该字段可以存储NULL值。如果你不指定这个参数，那么默认情况下字段是不允许存储NULL值的。当你想要允许某个字段的值为空时，就可以使用这个参数。
2. `default=date.today`: 这个参数用于指定字段的默认值。在创建新对象时，如果没有为该字段提供值，那么将使用指定的默认值。在这个例子中，`date.today`表示默认值是当前日期。
3. `auto_now=True`: 这个参数用于在每次更新对象时自动更新字段的值为当前时间。换句话说，每次更新对象时，该字段的值都会被设置为当前时间。
4. `auto_now_add=True`: 这个参数用于在创建新对象时自动将字段的值设置为当前时间。它与`auto_now`参数的区别在于，`auto_now_add`只在对象创建时生效，而不会在更新对象时修改字段的值。

下面是一个示例，演示如何在Tortoise ORM模型定义中使用这些参数：

```python
from tortoise.models import Model
from tortoise import fields
from datetime import date

class MyModel(Model):
    my_field = fields.DatetimeField(null=True, default=date.today, auto_now=True, auto_now_add=True)
```

在这个示例中，`my_field`字段是一个DatetimeField类型的字段。它允许存储NULL值（null=True），默认值是当前日期（default=date.today），并且在每次更新对象时自动更新为当前时间（auto_now=True），在创建新对象时自动设置为当前时间（auto_now_add=True）。

在Tortoise ORM的模型定义中，`auto_now_add=True`参数通常用于DateTimeField类型的字段，以便在创建新对象时自动设置为当前时间。对于BigIntField类型的字段，它代表一个大整数，通常用于存储时间戳或类似的数值数据。

在Tortoise ORM中，`auto_now_add=True`参数不会直接应用于BigIntField类型的字段，因为它是用于处理时间戳或日期时间的特殊行为。

如果你想要在创建新对象时自动设置BigIntField字段的值为当前时间戳，你可以使用其他方法来实现。例如，在创建新对象之前，你可以手动获取当前时间戳，并将其赋给该字段。下面是一个示例：

```python
from tortoise.models import Model
from tortoise import fields
import time

class MyModel(Model):
    my_bigint_field = fields.BigIntField()

    async def save(self, *args, **kwargs):
        if not self.my_bigint_field:
            # 如果字段为空，则设置为当前时间戳
            self.my_bigint_field = int(time.time() * 1000)  # 假设这里使用毫秒级时间戳
        await super().save(*args, **kwargs)
```

在这个示例中，`save()`方法被重写，在保存对象时检查BigIntField字段是否为空。如果字段为空，则将其设置为当前时间戳。然后调用父类的`save()`方法来完成保存操作。这样可以在创建新对象时自动设置BigIntField字段的值为当前时间戳。

## 几个坑
### 外键

通过外键进行正向关联，使用字段名；进行反向关联，使用关系名。

要修改你的数据模型，使得 `Orders` 到 `Signals` 的关系是 N:1 （多个订单可以关联到一个信号），你需要确保 `ForeignKeyField` 的使用是正确的，并且 `related_name` 的设置反映了这种关系。在你提供的模型中，`related_name` 应该表示从 `Signals` 到 `Orders` 的逆向引用，通常应该是复数形式，表示多个 `Orders` 可以链接到单个 `Signals`。

根据你的描述，看起来只需要微调 `related_name` 和注释，其他部分已正确设置。以下是修改后的模型定义：

```python
from tortoise import fields, Model

class Orders(Model):
    """订单表，信号相关字段只有信号id，其他内容都是记录订单结果"""
    id = fields.IntField(pk=True, description="订单ID")
    uid = fields.CharField(null=True, max_length=10, index=True, description="下单的用户在Drupal的uid")  # 下单的用户id
    # 信号ID，注意 ForeignKeyField 指向 'models.Signals'，related_name 设置为 'orders'
    # 表示从 Signals 到 Orders 的关系是一对多，即一个 Signal 可以关联多个 Orders
    signal = fields.ForeignKeyField('models.Signals', related_name='orders', description="跟单的交易信号ID")
```


1. **ForeignKeyField**：此字段类型用于建立从多到一的关系（即多个 `Orders` 可以关联到一个 `Signals`）。在 `ForeignKeyField` 中：
   - 第一个参数是引用的模型，格式为 `'models.<ModelName>'`，这里是 `'models.Signals'`。
   - `related_name='orders'` 表示在 `Signals` 模型中可以通过 `orders` 属性访问关联的 `Orders` 实例列表。

2. **related_name 的使用**：`related_name` 属性在 `Signals` 模型中创建了一个 **虚拟的字段** ，允许通过一个信号获取所有关联的订单。例如，如果你有一个 `Signals` 实例 `signal`，你可以通过 `signal.orders` 访问所有关联的 `Orders` 实例。
3. **注释更新**：注释中提到的内容应该清楚地说明字段的用途和关系。

通过这些修改，你的数据模型现在应该正确地设置了 `Orders` 到 `Signals` 的 N:1 关联。这将允许每个信号关联多个订单，但每个订单只关联到一个信号。

[tortoisesvn - AttributeError: 'int' object has no attribute '_saved_in_db' Fastapi Tortoise(orm) - Stack Overflow](https://stackoverflow.com/questions/69122108/attributeerror-int-object-has-no-attribute-saved-in-db-fastapi-tortoiseor)

字段 user 是外键，Tortoise 在创建表时，会对外键字段再加个 _id 后缀

所以，create表的时候，外键字段要加后缀 _id，例如：

.create( user_id="" )

但是，在 .filter(user__关联到的表的字段名="") 是不需要加 _id 后缀的。
