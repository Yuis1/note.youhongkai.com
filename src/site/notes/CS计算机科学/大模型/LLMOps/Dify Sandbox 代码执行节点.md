---
{"dg-publish":true,"permalink":"/CS计算机科学/大模型/LLMOps/Dify Sandbox 代码执行节点/","noteIcon":"","created":"2025-01-26T12:53:00.613+08:00","updated":"2025-01-26T21:09:56.439+08:00"}
---


作者：游鱼思

---
## 查看当前支持的Python包
### 通过节点代码查看

代码节点输入以下代码：

```
def main():
    import pkg_resources
    installed_packages = pkg_resources.working_set
    installed_packages_list = sorted(["%s==%s" % (i.key, i.version) for i in installed_packages])
    return {
        "result": installed_packages_list,
    }
```

输出变量：result，类型为 Array[String]

我的Dify是 v0.11.1 版本，输出如下：

```
{ "result": [ "anyio==4.6.2.post1", "certifi==2024.8.30", "charset-normalizer==3.4.0", "exceptiongroup==1.2.2", "h11==0.14.0", "httpcore==1.0.6", "httpx==0.27.2", "idna==3.10", "jinja2==3.0.3", "markupsafe==3.0.1", "pip==23.0.1", "pysocks==1.7.1", "requests==2.32.3", "setuptools==65.5.1", "sniffio==1.3.1", "socksio==1.0.0", "typing-extensions==4.12.2", "urllib3==2.2.3", "wheel==0.44.0" ] }
```


## 引入自定义Python包
### 通过Sandbox镜像容器的配置项

将需要的依赖放入`/docker/volumes/sandbox/dependencies/python-requirements.txt`，重启sandbox即可。

### 通过execute_code方法

来自： https://github.com/langgenius/dify/discussions/6127

要在 <代码执行> 模块中安装所需的依赖包，例如 `mysql-connector-python`，你需要在调用 `execute_code` 方法时包含该依赖包。以下是一个示例：

```
from core.helper.code_executor.entities import CodeDependency
from core.helper.code_executor.code_executor import CodeExecutor

# 定义依赖包
mysql_connector_dependency = CodeDependency(name='mysql-connector-python', version='latest')

# 执行包含依赖包的代码
try:
    result = CodeExecutor.execute_code(
        language='python3',
        preload='',
        code='import mysql.connector\nprint(mysql.connector.__version__)',
        dependencies=[mysql_connector_dependency]
    )
    print(result)
except CodeExecutionException as e:
    print(f'代码执行失败: {e}')
```

这个代码片段展示了如何指定 `mysql-connector-python` 作为依赖包，并执行一个简单的 Python 脚本来导入该模块并打印其版本号.

## 第三方Sandbox
### dify-sandbox-py

https://github.com/svcvit/dify-sandbox-py

**目的**
因为官方sandbox有很多关于权限的设置，那是一个更好的沙盒方案，但是个人实际使用过程中，Dify的代码节点完全是个人编辑，所以也不存在代码注入风险，希望有更大的权限，安装更多依赖包例如numpy>2.0，matplotlib，scikit-learn 减少一些看不懂的报错，因此参考官方sandbox的API调用示例，开发了本代码。

**特性**
- 去掉了网络访问的控制，默认就支持访问网络
- 使用UV作为依赖管理，安装依赖速度更快，重启可以毫秒级安装依赖。
- 第三方依赖安装与官方一致，将需要的依赖放入`/docker/volumes/sandbox/dependencies/python-requirements.txt`，重启sandbox即可。
- 镜像只有fastapi相关的依赖，任何你需要的依赖，需要自己加到python-requirements.txt中。