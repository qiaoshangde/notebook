





```
JSONUtil.toBean(json, User.class);
JSONUtil.toJsonStr(shop);
```

Redis 缓存中经常这样使用：
```

// Java 对象转 JSON，然后存入 Redis
String json = JSONUtil.toJsonStr(shop);

stringRedisTemplate.opsForValue()
        .set("cache:shop:" + shop.getId(), json);


```

读取时：
```

// 从 Redis 取出 JSON
String json = stringRedisTemplate.opsForValue()
        .get("cache:shop:" + id);

// JSON 转回 Java 对象
Shop shop = JSONUtil.toBean(json, Shop.class);
```




ctrl + shift +u
Boolean.TRUE.equals(success)#排除自动拆箱的风险

uuid  一版使用hutool工具类（好用）。获取时间好像也是用这个

事务   事务保证原子性（两个动作要么都成功，要么都失败），一致性
事务（是java的？还是mysq的）

lua脚本。redis提供的一个脚本，在这个脚本中额可以编写多个redis命令，确保多条命令执行时的原子性。lua是一种编程语言

在redis中使用lua语法：EVAL "lua拼接字符串" key类型的参数数量
lua中使用redis return redis.cal("","","")


ClasssPatgResource()   就是reaource中的东西

静态常量和静态代码块


用lua脚本来进行写。因为lua是原子性的，所以不会有线程安全问题。

# 🚀 IDEA 常用快捷键速查手册（Spring Boot 开发篇）

## 一、代码导航与跳转

| 快捷键 (Windows/Linux)              | 快捷键 (Mac)                              | 功能说明                                                   |
| :------------------------------- | :------------------------------------- | :----------------------------------------------------- |
| **`Ctrl + B`** 或 **`Ctrl + 左键`** | **`Command + B`** 或 **`Command + 左键`** | **转到定义**（Go to Declaration）。查看类、方法、变量的原始声明处。           |
| **`Ctrl + Alt + B`**             | **`Command + Option + B`**             | **转到实现**（Go to Implementation）。从接口或抽象方法，直接跳转到具体的实现类代码。 |
| **`Ctrl + H`**                   | **`Control + H`**                      | **查看类型层级**（Type Hierarchy）。以树形结构展示当前类/接口的所有父类、子类和实现类。  |
| **`Ctrl + N`**                   | **`Command + O`**                      | **全局搜索类**（Go to Class）。快速查找并跳转到项目中的某个类。                |
| **`Ctrl + Shift + N`**           | **`Command + Shift + O`**              | **全局搜索文件**（Go to File）。快速查找并跳转到项目中的某个文件。               |
| **`Ctrl + Alt + 左/右箭头`**         | **`Command + Option + 左/右箭头`**         | **前进/后退**导航。在最近浏览过的代码位置之间来回切换。                         |

## 二、代码编辑与生成

| 快捷键 (Windows/Linux)   | 快捷键 (Mac)                  | 功能说明                                                                                       |
| :-------------------- | :------------------------- | :----------------------------------------------------------------------------------------- |
| **`Alt + Enter`**     | **`Option + Enter`**       | **万能意图操作**（Show Intentions）。这是最核心的快捷键！<br>当代码报红时：**导包**、修复错误、生成方法等。<br>更多时候：提供各种智能建议和代码操作。 |
| **`后缀补全`** (如 `.var`) | **`后缀补全`** (如 `.var`)      | **智能模板补全**（Postfix Completion）。<br>先写表达式，紧跟 `.var` 并回车，自动生成变量声明语句。                         |
| **`Ctrl + Alt + V`**  | **`Command + Option + V`** | **提取变量**（Extract Variable）。选中一个表达式，用此快捷键自动生成变量声明。效果类似 `.var`。                              |
| **`Alt + Insert`**    | **`Command + N`**          | **生成代码**（Generate）。在编辑器中，快速生成构造器、Getter/Setter、`toString()` 等方法。                           |

## 三、其他实用命令

| 快捷键 (Windows/Linux) | 快捷键 (Mac) | 功能说明 |
| :--- | :--- | :--- |
| **`Ctrl + Shift + O`** | **`Shift + Command + O`** | **自动整理/导入**当前文件中所有未导入的类。 |
| **`Ctrl + Alt + L`** | **`Command + Option + L`** | **格式化代码**（Reformat Code）。一键美化当前文件代码格式（缩进、空格等）。 |
| **`Shift + Shift`** | **`Shift + Shift`** | **全局搜索**（Search Everywhere）。最强大的搜索框，可以搜类、文件、菜单、操作等一切内容。 |

---

**📌 一句话总结：**

- **报错不会修？** → `Alt + Enter` (万能键)
- **想看具体实现？** → `Ctrl + Alt + B` (接口→实现)
- **想偷懒声明变量？** → `.var` 或 `Ctrl + Alt + V`
- **满屏代码乱糟糟？** → `Ctrl + Alt + L` (一键美化)

