---
{"dg-publish":true,"permalink":"/CS计算机科学/WordPress/WordPress Taxonomy/","noteIcon":"","created":"2025-07-31T11:03:03.997+08:00","updated":"2024-07-23T14:44:26.000+08:00"}
---


作者：游鱼思

---

在 WordPress 中，分类法（Taxonomy）是一个用于组织内容的通用术语。

WordPress系统中已经自带了两个Taxonomy：Tag、Category

Tag：也即是不带层级的Taxonomy

Category：带层级的Taxonomy

Taxonomy 都是可多选的，是否仅单选由UI控件来控制。

### 分类法（Taxonomy）
- **默认分类法**：
  - **分类（Categories）**：一种层次结构的分类法，可以有父分类和子分类。常用于博客文章的组织。
  - **标签（Tags）**：一种非层次结构的分类法，没有父子关系。通常用于给文章添加关键词，以便更灵活地组织内容。

### 自定义分类法（Custom Taxonomies）
- **自定义分类法**：是用户或开发者根据特定需求创建的分类法。它们可以是层次结构的（类似于分类）或非层次结构的（类似于标签）。
  - **层次结构的自定义分类法**：类似于默认的分类（Categories），可以有父子关系。例如，“烹饪方法（Cooking Methods）”可以有“炒菜（Stir-Fried）”和“蒸煮（Steamed/Boiled）”作为子分类。
  - **非层次结构的自定义分类法**：类似于默认的标签（Tags），没有父子关系。例如，“营养功能（Nutritional Functions）”中的“高蛋白（High-Protein）”和“高纤维（High-Fiber）”。

### 分类法和自定义分类法的区别
- **预定义 vs. 自定义**：
  - 默认分类法是 WordPress 核心提供的，已经预定义好，可以直接使用。
  - 自定义分类法是用户或开发者根据需要自己创建的，可以完全根据需求进行定制。
  
- **用途**：
  - 默认分类法一般用于常见的内容组织，例如博客文章的分类和标签。
  - 自定义分类法通常用于特定类型内容的分类，例如菜谱的烹饪方法、饮食方式等。
- **创建和管理**：
  - 默认分类法可以直接在 WordPress 后台中创建和管理。
  - 自定义分类法需要通过代码或使用插件（如 Custom Post Type UI）来创建和管理。

### 什么时候使用自定义分类法
- **特定需求**：当默认的分类和标签不能满足您的需求时，可以创建自定义分类法。例如，如果需要对菜谱进行特定维度的分类（如烹饪方法、营养功能等），可以创建相应的自定义分类法。
- **更好的组织**：自定义分类法允许您以更精细、更符合逻辑的方式组织内容，使得前端展示和用户体验更加友好。

### 实施示例
```php
function create_recipe_taxonomies() {
    // Cooking Methods
    register_taxonomy('cooking_method', 'recipe', array(
        'label' => 'Cooking Methods',
        'rewrite' => array('slug' => 'cooking-method'),
        'hierarchical' => true,
    ));
    
    // Meal Times
    register_taxonomy('meal_time', 'recipe', array(
        'label' => 'Meal Times',
        'rewrite' => array('slug' => 'meal-time'),
        'hierarchical' => true,
    ));
    
    // Diet Types
    register_taxonomy('diet_type', 'recipe', array(
        'label' => 'Diet Types',
        'rewrite' => array('slug' => 'diet-type'),
        'hierarchical' => true,
    ));
    
    // Nutritional Functions
    register_taxonomy('nutritional_function', 'recipe', array(
        'label' => 'Nutritional Functions',
        'rewrite' => array('slug' => 'nutritional-function'),
        'hierarchical' => false,
    ));
    
    // Condition-Specific Diets
    register_taxonomy('condition_specific', 'recipe', array(
        'label' => 'Condition-Specific Diets',
        'rewrite' => array('slug' => 'condition-specific'),
        'hierarchical' => true,
    ));
}
add_action('init', 'create_recipe_taxonomies', 0);
```

Advanced Custom Fields (ACF) 可以新增、修改自定义分类法。此外，也可以使用以下扩展：

### 1. Custom Post Type UI (CPT UI)

这是一个非常流行且易于使用的插件，用于创建和管理自定义分类法和自定义文章类型。

- **优点**：
  - 用户友好的界面。
  - 不需要编写代码。
  - 可以与 ACF 集成使用。

### 2. TaxoPress

TaxoPress 插件允许您在 WordPress 中创建、管理和优化分类法。

- **优点**：
  - 支持标签和分类的合并、拆分和批量管理。
  - 可以创建自定义分类法。
  - 提供自动标签、标签云和其他功能。

### 3. Pods – Custom Content Types and Fields

Pods 是一个功能强大的插件，可以创建和管理自定义内容类型和分类法。

- **优点**：
  - 可以创建自定义文章类型、自定义分类法和自定义字段。
  - 提供一个强大的界面来管理内容关系。
  - 与 ACF 集成良好。

### 4. Toolset Types

Toolset Types 插件允许您创建自定义文章类型、分类法和字段。

- **优点**：
  - 提供一个综合的解决方案来管理自定义内容类型。
  - 可以轻松创建和管理自定义分类法。
  - 与其他 Toolset 插件集成良好。

### 使用 ACF 与自定义分类法的集成
**使用 ACF 为自定义分类法添加字段**
   - 安装并激活 ACF 插件。
   - 导航到 "Custom Fields" > "Add New" 创建一个新的字段组。
   - 添加需要的字段。
   - 在 "Location" 规则中，选择 "Taxonomy Term" 并选择您创建的自定义分类法。

### 示例

如果已经使用 CPT UI 创建了一个名为 "cooking_method" 的自定义分类法，可以使用以下步骤在 ACF 中为它添加字段：

1. 在 ACF 中创建一个新的字段组，例如 "Cooking Method Details"。
2. 添加一个字段，例如 "Cooking Duration"（烹饪时间）。
3. 在 "Location" 规则中，选择 "Taxonomy Term" 并选择 "cooking_method"。

编辑某个烹饪方法的分类项时，就可以看到并填写这些自定义字段。

通过结合使用这些插件，您可以轻松创建和管理自定义分类法，并为这些分类法添加自定义字段，以满足您网站的需求。

## FAQ

1、通过ACF创建 taxonomy（通过 advance - Meta Box 选项可控制展示样式），关联到文章类型后，该文章类型在新增内容时，编辑器右侧可以创建该taxonomy的item。

但是问题是：如何创建taxonomy，却又不让编辑人员新增，并且展示已创建的item为radio button？

解决方案1：

- 关闭该Taxonomy在UI中的展示：advance - visibility - display in UI
- 使用ACF再创建字段组，新建类型为 taxonomy 的字段，其中有选项可以不让编辑时新增。

解决方案2：不用ACF，用 Meta box插件。

参考：

[ACF: How do I create terms for a custom taxonomy, and prevent an editor from creating new ones : r/Wordpress (reddit.com)](https://www.reddit.com/r/Wordpress/comments/17alpxt/acf_how_do_i_create_terms_for_a_custom_taxonomy/)

2、ACF有个Bug：当ACF创建的Taxonomy的 分类法键 与 关联到的文章类型的 文章类型键 相同时，在 创建/编辑该文章时，将不会展示已经添加的分类项。

![](/img/user/Z-attach/Pasted image 20240719143044.png)