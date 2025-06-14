---
{"dg-publish":true,"permalink":"/CS计算机科学/WordPress/WordPress VS Drupal：我走的弯路/","noteIcon":"","created":"2024-07-09T01:43:56.548+08:00","updated":"2024-07-09T02:20:25.000+08:00"}
---


作者：游鱼思

---

早在2009年，我为公司制作网站时，在调研使用哪个CMS，当时最流行是 Drupal 和 WordPress。

我对比了扩展数量、灵活度、上手难度，选了上手难但是灵活度高的 Drupal。

这些年就这么习惯了用Drupal，总觉得WordPress太简单了，是给小白用的，整不出花来。

近期才知道WordPress已经远远把Drupal甩在了身后，成了市占率62%的CMS，并且WooCommerce插件在电商网站中占比27%。而Drupal无论是生态、移动端支持、商城支付等，都已经落后。

这个话题很像 Windows桌面版 VS MacOS VS Linux桌面端

最终都是 技术思维 败给了 产品思维，免费生态 败给了 商业生态，灵活复杂 败给了 简单好用。

市场的选择有其深刻的原因，应该开放心态去深入调研、试用，不能为了小众而小众，只要经过了独立思考，即使选择了主流，也不是从众。

---

以下是ChatGPT的总结：

WordPress 的插件开发与 Drupal 的模块开发都提供了扩展和定制各自平台功能的能力，但它们在设计哲学、开发复杂性和入门门槛方面有所不同。下面是两者的一些比较：

### 1. 开发入门门槛
- **WordPress 插件开发** 相对简单，主要使用 PHP 语言，并且对新手较为友好。WordPress 提供了广泛的钩子（actions 和 filters）系统，允许开发者以较少的代码实现功能扩展。入门文档和教程也比较丰富，容易上手。
- **Drupal 模块开发** 通常被认为更为复杂，需要更深入的编程知识和对 Drupal 架构的理解。Drupal 使用了更多的现代 PHP 实践，如命名空间、服务和依赖注入等，这可能增加了学习的难度。但这也使得开发出的模块更为强大和灵活。

### 2. 系统架构
- **WordPress** 的架构相对简单，它强调插件和主题的开发，使其易于添加新功能或更改外观。插件开发者通常可以通过编写较少的代码并利用现有的钩子来实现功能。
- **Drupal** 的架构更为复杂，提供了更多的抽象层，这使得它在创建高度复杂和可扩展的应用方面更具优势。模块开发者需要适应这种复杂性，例如管理实体类型、表单API和渲染系统。

### 3. 社区和文档
- **WordPress** 拥有一个庞大的开发社区，大量的插件和主题可供参考，社区支持和在线资源也非常丰富。
- **Drupal** 的社区虽然较小，但专注于企业级和高级功能，提供的文档详尽严谨，适合需要构建复杂网站结构的开发者。

### 4. 开发用例
- **WordPress** 由于其简单性，更适合快速开发博客、简单企业网站和小型电商平台等。
- **Drupal** 的模块开发由于其强大的定制能力，更适合需要复杂数据结构和严格访问控制的大型项目和企业应用。

总的来说，如果你是初学者或者需要快速开发并部署简单到中等复杂度的网站，WordPress 插件开发可能更合适。而如果你是寻求开发高度定制化和拥有复杂交互的大型网站，Drupal 模块开发则可能更适合你的需求。每个平台的开发难度也与你的具体项目需求和个人技术背景有关。