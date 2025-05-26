---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/Drupal 10 模块开发/","noteIcon":"","created":"2024-06-22T22:30:28.461+08:00","updated":"2024-05-06T12:12:53.000+08:00"}
---


作者: 游鱼思

---
## 开发环境准备

官方推荐使用DDEV，基于Docker

## Symfony

是 Drupal 的底层框架。

## Service



## Hook

在 .module 文件里调用Hook

调用格式：自定义模块名_hook方法名

## 路由系统

在 Drupal 中，路由是一个系统，它负责处理和响应 HTTP 请求。路由系统将 URL 路径映射到特定的控制器方法（也称为回调函数），这个方法负责生成请求的响应。

每个路由都有一个唯一的路由名称，以及与之关联的路径（URL）、控制器类和方法、访问权限等信息。这些信息通常在模块的 *.routing.yml 文件中定义。

例如，你可能有一个路由，它的路径是 /my-module/my-page，当用户访问这个 URL 时，Drupal 的路由系统会找到与这个路径关联的控制器方法，并执行它来生成页面的内容。

路由系统是 Drupal 的核心组件，它使得模块可以定义自己的页面、表单、API 端点等，而无需关心底层的 HTTP 机制。

### Controller组件

在 Drupal 中，Controller 是处理 HTTP 请求并返回一个响应的组件。每个 Controller 类通常包含一个或多个方法，这些方法被称为 "controller methods" 或 "action methods"。每个方法对应一个特定的路由（URL），当该路由被请求时，对应的方法就会被执行。

Controller 的主要职责是：

接收请求：Controller 从请求对象中获取输入和参数。

处理请求：Controller 使用获取的输入和参数执行必要的业务逻辑。

返回响应：Controller 生成并返回一个响应对象。这个响应可以是一个 HTML 页面，也可以是 JSON 数据，或者其他任何格式的内容。

在 Drupal 中，Controller 类通常放在模块的 src/Controller 目录下，并且需要继承 Drupal\Core\Controller\ControllerBase 类。ControllerBase 类提供了一些常用的便利方法，例如访问数据库、翻译系统、当前用户等。

## 表单

在 Drupal 中，创建自定义表单需要创建一个新的类，该类继承自 Drupal\Core\Form\FormBase 或 Drupal\Core\Form\ConfigFormBase，并实现 FormInterface 接口。这个类需要定义一些特定的方法，包括 getFormId(), buildForm(), validateForm(), 和 submitForm()。

ContainerInterface 是 Symfony 组件的一部分，Symfony 是 Drupal 的底层框架。ContainerInterface 是服务容器的接口，服务容器是用于管理服务（如数据库连接、配置系统等）的对象。

create() 方法是一个特殊的方法，它允许 Drupal 的依赖注入容器自动为你的表单类提供服务。这个方法接收一个 ContainerInterface 参数，你可以从这个容器中获取你需要的服务。

以下是一个创建自定义表单的基本示例：

'''php

namespace Drupal\momo_papers\Form;

use Drupal\Core\Form\FormBase;

use Drupal\Core\Form\FormStateInterface;

use Symfony\Component\DependencyInjection\ContainerInterface;

class MomoPapers extends FormBase {

  public static function create(ContainerInterface $container) {

    // 在这里，你可以从 $container 中获取你需要的服务，并将它们作为你的类的构造函数的参数。

    // 例如，如果你需要数据库连接，你可以这样做：

    // $database = $container->get('database');

    // return new static($database);

  }

  public function getFormId() {

    // 返回一个唯一的字符串，用于标识你的表单。

    return 'momo_papers';

  }

  public function buildForm(array $form, FormStateInterface $form_state) {

    // 在这里，你可以构建你的表单数组。

    // 例如，你可以添加一个文本字段：

    // $form['example'] = [

    //   '#type' => 'textfield',

    //   '#title' => $this->t('Example'),

    // ];

    // return $form;

  }

  public function validateForm(array &$form, FormStateInterface $form_state) {

    // 在这里，你可以添加你的表单验证逻辑。

  }

  public function submitForm(array &$form, FormStateInterface $form_state) {

    // 在这里，你可以处理表单提交的数据。

  }

}

'''

需要在你的模块的 *.routing.yml 文件中定义一个路由，以便用户可以访问你的表单。

## 注释

采用 PHPDoc 来注释标签。

{@inheritdoc} 是一个特殊的注释标签，通常用在 DocBlocks（PHPDoc 注释）中。它的主要作用是表示当前方法的注释应该从父类或接口继承。当你在子类的方法上使用 {@inheritdoc} 注释时，这意味着你想要使用父类或实现的接口上相同方法的注释。这在你覆盖或实现父类或接口的方法时非常有用，因为你可以避免重复编写相同的注释。

@return array：这个标签表示该方法将返回一个数组。这对于理解方法的预期输出和编写调用代码非常有用。

@ingroup ：这个标签将当前元素（通常是一个类、方法或函数）分配到一个特定的“组”或“模块”。这在生成文档时非常有用，因为它允许你根据组来组织和分类文档。
