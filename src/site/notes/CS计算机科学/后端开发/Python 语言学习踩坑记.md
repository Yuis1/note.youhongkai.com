---
{"dg-publish":true,"permalink":"/CS计算机科学/后端开发/Python 语言学习踩坑记/","noteIcon":"","created":"2024-06-22T22:30:28.842+08:00","updated":"2024-05-28T11:45:52.000+08:00"}
---


作者：游鱼思

Python最烦人的，应该就是包依赖了。

对于我这种256G的丐版存储，抠门舍不得花空间建虚拟环境的人，非常不友好。

---
## Linux环境

停止使用CentOS，因为马上停止支持了。用Ubuntu吧。  
并且安装的各种系统包版本也较低，比如 openssl 、Python3

Tortoise-ORM requires Python >= 3.7

而 python3.11需要的openssl的版本为1.1.x及以上

低于 Python 3.7 的，或者 openssl 没有 1.1.x以上的，各种报错无法运行。  
因为这个浪费了N多时间。  

这篇文章提供了编译 openssl、python 的正确方法：

https://stackoverflow.com/questions/73407527/installing-ssl-package-with-pip-requires-ssl-package-to-be-already-installed

yum自带openssl才到1.0，需要更新到3  
yum install gcc libffi-devel zlib* openssl-devel

下载 openssl后，解压  
./config --prefix=/usr/local/openssl shared zlib

vi /etc/ld.so.conf

添加内容: /usr/local/openssl/lib  
ldconfig -v

设置openssl命令的软链接

`ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl`

验证是否安装成功

`openssl version -a`

## 下载安装 Python 3.11

如果您是从源代码编译安装 Python，那么需要在安装了 OpenSSL 库之后重新编译 Python。确保在编译 Python 之前安装 OpenSSL 库，这样 Python 编译时就可以包含 SSL 支持。

centos的yum最高目前只有3.6

```

wget https://www.python.org/ftp/python/3.11.7/Python-3.11.7.tgz

tar xvf Python-3.11.7.tgz

cd Python-3.11.7

./configure --with-openssl=/usr/local/openssl

make & make install
```

验证：

python3.11 -V

It is important that the --with-openssl option has the same value as the --prefix option when you configured OpenSSL above!!!

sudo make altinstall  则是可选安装，不替换系统默认版本。

最后，运行程序时还可能报出证书错误：

pip3 install certifi

python -c "import certifi; print(certifi.where())"

在 ~/.bashrc 添加

export SSL_CERT_FILE=/path/to/cacert.pem

## 宝塔面板的Pthon管理就是大坑

不仅 requirements.txt 无法在界面导入

安装的包在运行时还出各种错。

### 设置python的系统路径

在 Unix-like 系统（如 Linux 或 MacOS）中，你可以通过修改 PATH 环境变量来设置 Python 的运行路径。你可以在你的 shell 配置文件（如 ~/.bashrc 或 ~/.bash_profile）中添加以下行：

export PATH="/path/to/your/python:$PATH"

/path/to/your/python 应该被替换为你的 Python 解释器的路径。你可以通过运行 which python 或 which python3 来找到这个路径。

例如：

export PATH="/www/server/pyporject_evn/versions/3.11.4/bin:$PATH"

立即生效： source ~/.bashrc

## 烦人的环境配置

目前最佳实践还是使用 venv，以空间换简洁，并且和vscode集成，不用记乱七八糟的命令。

将虚拟环境下依赖的包输出到requirements.txt  
pip freeze > requirements.txt

2. **创建虚拟环境**：
   - 使用 Python 3 创建一个虚拟环境。这允许您在隔离的环境中安装和管理包，而不影响系统级别的 Python 安装。
   - 创建并激活虚拟环境的步骤如下：
	 ```
	 python3 -m venv .venv
	 source .venv/bin/activate  # 在 Unix 或 MacOS 上
	 .venv\Scripts\activate  # 在 Windows 上
	 ```
   - 在虚拟环境中，您可以使用 `pip` 而不是 `pip3`，因为 `pip` 会自动指向虚拟环境的 Python 版本。

3. **更新默认的 Python 版本**：
   - 如果您希望默认使用 Python 3，可以考虑更新系统的默认 Python 版本。但请注意，这可能影响依赖于 Python 2 的系统应用和脚本。
   - 在某些系统中，您可以通过修改 `~/.bashrc` 或 `~/.bash_profile` 文件中的别名来设置默认的 Python 版本：
 ```
alias python=python3
alias pip=pip3
 ```
>source ~/.bashrc

一旦激活，您将在终端提示符前看到虚拟环境的名称，表示您现在处于虚拟环境中。

4. 现在，您可以在虚拟环境中安装所需的Python包和库，例如：

```
   pip install package_name
```

   这将安装包并将其保存在虚拟环境中，而不会影响全局Python环境。

5. 当您完成工作时，可以使用以下命令来退出虚拟环境：

   ```
   deactivate
   ```

   这将使您返回到全局Python环境。

通过按照上述步骤，您可以轻松地创建和使用虚拟环境，以隔离不同项目的依赖关系，并确保它们不会相互干扰。

如果直接把 .venv 搬运到服务器目录，运行又会出现 证书错误

[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1002)

解决方法可能是： 找到并运行 Install Certificates.command

## 混乱的包名

提示：requests.exceptions.InvalidSchema: Missing dependencies for SOCKS support.

解决：pip3 install 'requests[socks]'

这里的坑在于  'requests[socks]' 需要引号。

### websocket有几个包

1、功能不全的 websocket

导入的时候：

import websocket

2、websocket-client  不带服务端，不支持异步

pip3 install websocket-client

导入的时候也是：

import websocket

3、websockets 带服务端，支持异步

pip3 install websockets

导入的时候：

import websockets

## 代理

如果是采用socket5代理访问api，可能报错：

Using SOCKS proxy, but the 'socksio' package is not installed. Make sure to install httpx using `pip install httpx[socks]`.

pip3 install "httpx[socks]"

pip3 install PySocks

还是报错："Could not install packages due to an OSError: Missing dependencies for SOCKS support"

解决：需要先关闭代理，或者在新的shell中安装，否则这个错误会阻塞安装。

## 数据类型

Python 中的 `Decimal` 和 `float` 都是用来表示浮点数的,但是它们在精度、存储方式和运算规则上存在一些区别。

1. **精度**

`float` 是用二进制来近似表示浮点数,因此在某些情况下会有精度丢失的问题。例如:

```python
>>> 0.1 + 0.2
0.30000000000000004
```

而 `Decimal` 则是用字符串的形式精确地表示一个浮点数,可以避免这种精度丢失的问题。

```python
>>> from decimal import Decimal
>>> Decimal('0.1') + Decimal('0.2')
Decimal('0.3')
```

2. **存储方式**

`float` 在内存中是用二进制的方式存储,而 `Decimal` 则是用字符串的形式存储。`Decimal` 更加精确,但是存储和运算的效率相对会低一些。

3. **运算规则**

`float` 遵循 IEEE 754 标准,在运算过程中可能会出现一些我们预期之外的结果。而 `Decimal` 可以通过设置不同的上下文(Context)来控制运算的规则,比如精度、舍入模式等。

4. **使用场景**

如果是科学计算或大部分的普通运算,使用 `float` 已经足够。但如果涉及到金融计算等对精度要求极高的场景,使用 `Decimal` 会更加合适。

总的来说,`Decimal` 用于需要精确计算的场合,如财务、货币等领域,而 `float` 则用于科学计算或普通的简单运算。在精度要求不是很高的情况下,使用 `float` 可以获得更好的性能。

## Python 语法

在 Python 中，`__init__.py` 文件有几个重要的作用，主要用于组织模块和包。以下是它的主要功能：

1. **标记目录为 Python 包**：
   - 在 Python 中，任何包含 `__init__.py` 文件的目录都被视为 Python 包。这允许你将一系列相关的模块组织到一个包中。例如，如果有一个目录 `my_package`，包含文件 `__init__.py`，那么你可以通过 `import my_package` 来导入这个包。
2. **初始化代码**：
   - `__init__.py` 文件也可以包含用于包初始化的代码。当包被导入时，`__init__.py` 中的代码将被执行。这可以用来设置包所需的任何初始状态或变量。
3. **控制包的命名空间**：
   - 通过在 `__init__.py` 文件中定义 `__all__` 列表，你可以明确地指定当使用 `from package import *` 时应该导出哪些模块。这有助于避免命名空间污染。
4. **简化导入**：
   - 你可以在 `__init__.py` 文件中导入包内的模块，这样外部导入这个包时，可以直接访问这些模块。例如，在 `__init__.py` 中写入 `from .module import MyClass`，那么可以直接通过 `import my_package` 后使用 `my_package.MyClass`。
5. **向后兼容性**：
   - 在 Python 3.3 之前，`__init__.py` 文件是定义包的必要条件。在 Python 3.3 及以后，引入了隐式的命名空间包，不再强制要求有 `__init__.py` 文件。但出于向后兼容性和明确性的考虑，很多项目仍然使用 `__init__.py` 文件。

总的来说，`__init__.py` 在定义和组织 Python 包时扮演着重要角色，尽管在最新版本的 Python 中，它不再是严格必需的。

Q：通常在Prompt末尾，添加注释 # noqa: E501 ，这有什么用途？  
A：在Python中，# noqa: E501 是一种特殊的行内注释，用于告诉某些代码检查工具忽略特定的规则。具体来说，# noqa: E501 是用来指示代码检查工具忽略“行太长”的警告。

noqa：这是一个常见的约定，用于指示“不对此行应用质量保证（Quality Assurance）检查”。noqa 代表 "No Quality Assurance"。

E501：这是一个特定的错误码，通常由PEP 8风格指南中的代码检查工具（如 flake8）使用，用于表示一行代码的长度超过了建议的最大长度（默认是79个字符）。

当你在一行代码的末尾添加 # noqa: E501 注释时，它会告诉支持这一约定的代码检查工具忽略这一行的长度限制。这在某些情况下很有用，例如当一行代码包含长字符串或复杂表达式，而你不想将其拆分成多行时。

在Python中，前缀为双下划线（__）的变量或参数名具有特殊的含义。这种命名习惯通常用于实现特定的面向对象编程概念，如下所述：

名称改编（Name Mangling）：

当成员变量或函数名以双下划线 (__) 开头时，Python解释器会对其进行名称改编。这意味着这些名称在内部会被转换成包含其所属类名的形式。例如，在类 MyClass 中，一个名为 __example 的变量实际上会被转换为 _MyClass__example。

这种改编是为了避免在继承时子类意外重写基类的成员。

限制外部访问：

双下划线前缀通常用于表示一个变量或方法是“私有”的，意味着它们不应该从类的外部访问或修改。虽然Python没有强制的访问控制，但这是一种约定，用于指示程序员应该如何使用这些成员。

在Python中，**kwargs 是一种特殊的参数，用于在函数定义中收集任意数量的关键字参数。这里的 ** 是一个解包操作符，它允许将多个关键字参数（这些参数以键值对的形式出现）聚集成一个字典。kwargs 是 “keyword arguments” 的缩写，但你可以使用任何名称，关键是 ** 前缀。

当一个函数定义包含 **kwargs 时，它可以接受任何数量的关键字参数，而不是只接受固定数量或固定名称的参数。这些参数在函数内部作为字典处理，其中参数名是字典的键，参数值是字典的值。

这种机制使得函数非常灵活，因为你可以传递不同数量和不同名称的参数，而不需要修改函数的定义。它在需要处理不确定数量的参数时非常有用，尤其是在编写通用函数或库时。

## 工厂函数

工厂函数（Factory Function）是一种常见的编程模式，它用于创建并返回对象实例。Python中的工厂函数通常是一个函数，接受一些参数，并根据这些参数来创建并返回新的对象。下面我将详细解释Python工厂函数的用法。

**1. 创建工厂函数：**

首先，您需要定义一个工厂函数，它将负责创建对象。这个函数可以具有任何名称，通常以`create_`或`make_`等前缀开头，以清楚地表示其目的。下面是一个简单的示例，创建一个工厂函数来生成人员对象：

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

def create_person(name, age):
    return Person(name, age)
```

在上面的示例中，`create_person`是工厂函数，它接受名称和年龄作为参数，并创建一个`Person`对象。

**2. 使用工厂函数创建对象：**

要创建对象，只需调用工厂函数并传递必要的参数。例如：

```python
person1 = create_person("Alice", 30)
person2 = create_person("Bob", 25)
```

通过这种方式，您可以使用工厂函数轻松地创建多个对象。

**3. 工厂函数的好处：**

使用工厂函数有以下好处：

- **封装创建过程**：工厂函数封装了对象的创建过程，使代码更易维护和扩展。
- **参数验证**：工厂函数可以在创建对象之前验证传递的参数，以确保它们是有效的。
- **对象复用**：工厂函数可以实现对象的复用，避免重复创建相同类型的对象。
- **创建灵活性**：工厂函数可以根据不同的参数组合创建不同的对象类型。

**4. 示例扩展：**

以下是一个更复杂的示例，使用工厂函数创建不同类型的汽车对象：

```python
class Car:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year

def create_car(make, model, year):
    return Car(make, model, year)

def create_electric_car(make, model, year, battery_capacity):
    car = Car(make, model, year)
    car.battery_capacity = battery_capacity
    return car

def create_hybrid_car(make, model, year, battery_capacity, engine_type):
    car = Car(make, model, year)
    car.battery_capacity = battery_capacity
    car.engine_type = engine_type
    return car
```

在这个示例中，我们定义了三个工厂函数，用于创建不同类型的汽车对象。根据参数的不同，可以创建普通汽车、电动汽车或混合动力汽车。

希望这个详细的解释有助于理解Python工厂函数的用法。工厂函数是一种强大的设计模式，可以在代码中提高灵活性和可维护性。

## 类

在Python中，类的实例化过程涉及到两个方法：__new__()和__init__()。当你创建一个类的实例时，首先会调用__new__()方法来创建一个新的实例，然后调用__init__()方法来初始化这个实例。这两个方法都可以接受参数。

Python并没有像PHP那样的__invoke()魔术方法，可以让对象像函数一样被调用。但是，Python有一个类似的特性，那就是__call__()方法。如果一个类定义了__call__()方法，那么它的实例就可以像函数一样被调用。