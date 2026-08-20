# Spring Boot 基础注解与依赖注入笔记

> 适合 Spring Boot 零基础学习。本文从 Java 类、对象和 Bean 开始，依次讲解 Spring 容器、依赖注入、项目分层、Web 请求映射、JSON 响应和数据库事务。

## 一、先建立 Spring 的整体认识

Spring 最核心的工作之一，是帮助我们创建和组织对象。

在普通 Java 程序中，我们经常自己创建对象：

```java
UserService userService = new UserService();
```

在 Spring 项目中，可以把一些对象交给 Spring 创建和管理：

```java
@Service
public class UserService {
}
```

Spring 启动后会发现这个类，创建 `UserService` 对象，并把它放入 Spring 容器。其他对象需要使用它时，Spring 可以自动把它传进去。

可以先用下面的模型理解 Spring：

```text
发现需要管理的类
        ↓
创建对象或获得第三方框架创建的对象
        ↓
把对象注册到 Spring 容器，成为 Bean
        ↓
分析对象之间的依赖关系
        ↓
把需要的 Bean 注入其他 Bean
        ↓
在 Bean 周围增加事务、缓存等能力
```

Spring 容器可以暂时类比成一个“全局对象仓库”，但它并不是普通的全局变量。Spring 还会负责：

- 创建对象；
- 保存和查找对象；
- 注入对象需要的依赖；
- 管理对象的初始化和销毁；
- 管理 Bean 的作用域；
- 通过代理提供事务、缓存等额外功能。

## 二、类、对象和 Bean 的区别

### 1. 类是设计图

```java
public class UserService {

    public void sayHello() {
        System.out.println("Hello");
    }
}
```

`UserService` 是一个类。类描述对象应该具有哪些数据和行为，但类本身不是一个实际对象。

### 2. 对象是根据类创建出来的实例

```java
UserService userService = new UserService();
```

`new UserService()` 创建了一个实际对象。

### 3. Bean 是由 Spring 容器管理的对象

```java
@Component
public class UserService {
}
```

Spring 创建并放入容器的 `UserService` 对象叫作 Bean。

因此，Bean 不是“类”的特殊名称，而是“被 Spring 管理的对象”的名称。

```text
UserService                    → 类
new UserService()              → 普通 Java 对象
Spring 管理的 UserService 对象 → Bean
```

自己手动创建的对象通常不属于 Spring 容器：

```java
UserService userService = new UserService();
```

即使 Spring 容器中已经有一个 `UserService` Bean，上面手动创建的对象仍然是另一个对象，而且通常不能自动享受 Spring 的依赖注入、事务等能力。

### 4. 默认 Bean 名称

下面这个 Bean 的默认名称通常是 `userService`：

```java
@Component
public class UserService {
}
```

也可以指定名称：

```java
@Component("myUserService")
public class UserService {
}
```

一般不需要手动指定 Bean 名称。

## 三、`@Component`：最基础的组件注解

`@Component` 标记的是类，而不是普通函数：

```java
@Component
public class SmsSender {
}
```

它告诉 Spring：

> 请发现这个类，创建它的对象，并把对象放进 Spring 容器管理。

`@Component` 适合没有明确属于 Controller、Service、Repository 层的通用组件，例如：

```java
@Component
public class TokenParser {
}
```

```java
@Component
public class IdGenerator {
}
```

```java
@Component
public class SmsSender {
}
```

### `@Bean`：通过方法提供 Bean

如果需要通过方法创建对象，通常使用 `@Bean`，而不是 `@Component`：

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }
}
```

这里：

- `@Configuration` 表示这是一个 Spring 配置类；
- `@Bean` 告诉 Spring 执行这个方法；
- 方法返回的对象会被放进 Spring 容器；
- 默认 Bean 名称是方法名 `userService`。

## 四、组件扫描：Spring 怎么发现这些类

Spring Boot 启动类通常类似：

```java
@SpringBootApplication
public class HmDianPingApplication {

    public static void main(String[] args) {
        SpringApplication.run(HmDianPingApplication.class, args);
    }
}
```

`@SpringBootApplication` 包含组件扫描功能。Spring 默认扫描启动类所在包以及它的子包。

例如启动类位于：

```text
com.hmdp.HmDianPingApplication
```

那么这些位置通常可以被扫描：

```text
com.hmdp.controller
com.hmdp.service
com.hmdp.repository
com.hmdp.utils
```

而完全位于另一个根包中的类可能无法被默认扫描：

```text
com.example.service
```

因此，Spring Boot 启动类一般放在项目根包中。

## 五、依赖和依赖注入

### 1. 什么是依赖

如果 `OrderService` 完成工作时需要使用 `UserService`，就说 `OrderService` 依赖 `UserService`：

```java
public class OrderService {

    private UserService userService = new UserService();
}
```

依赖可以简单理解为：

> 一个对象完成工作时，需要使用另一个对象。

### 2. 什么是依赖注入

在 Spring 中，`OrderService` 不一定要自己创建 `UserService`。可以让 Spring 把已经管理的 `UserService` Bean 传进来：

```java
@Service
public class UserService {
}
```

```java
@Service
public class OrderService {

    private final UserService userService;

    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

Spring 大致会完成类似下面的工作：

```java
UserService userService = springContainer.getBean(UserService.class);
OrderService orderService = new OrderService(userService);
```

Spring 把 `UserService` 交给 `OrderService` 的过程，就是依赖注入，英文是 Dependency Injection，简称 DI。

## 六、三种常见注入方式

### 1. 构造方法注入：推荐

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

真正的注入发生在构造方法：

```java
public UserService(UserRepository userRepository)
```

Spring 创建 `UserService` 时会在容器中寻找 `UserRepository` Bean，然后把它作为参数传入。

如果一个类只有一个构造方法，Spring 通常允许省略 `@Autowired`：

```java
public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
}
```

写成下面这样也能工作，但 `@Autowired` 通常是多余的：

```java
@Autowired
public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
}
```

构造方法注入的优点：

- 依赖关系清楚；
- 字段可以使用 `final`；
- 创建对象时就能保证依赖存在；
- 更方便测试；
- 可以减少字段注入隐藏依赖的问题。

如果一个类有多个构造方法，Spring 可能不知道应该使用哪个构造方法，此时可以使用 `@Autowired` 明确指定。

### 2. `@Autowired` 字段注入

```java
@Service
public class OrderService {

    @Autowired
    private UserService userService;
}
```

`@Autowired` 是 Spring 提供的注解，主要按照类型寻找 Bean：

> 请从 Spring 容器中给我一个 `UserService` 类型的 Bean。

字段注入写起来简单，所以在一些教程和旧项目中很常见。不过实际开发通常更推荐构造方法注入。

### 3. `@Resource` 字段注入

```java
@Service
public class OrderService {

    @Resource
    private UserService userService;
}
```

`@Resource` 通常优先按照名称寻找 Bean，再尝试按照类型匹配。也可以明确指定名称：

```java
@Resource(name = "userService")
private UserService userService;
```

不同版本项目中的导包可能不同：

```java
// 较老的项目
import javax.annotation.Resource;

// 较新的项目
import jakarta.annotation.Resource;
```

### 4. 同一接口有多个实现类怎么办

```java
public interface PayService {
    void pay();
}
```

```java
@Service
public class AliPayService implements PayService {

    @Override
    public void pay() {
    }
}
```

```java
@Service
public class WechatPayService implements PayService {

    @Override
    public void pay() {
    }
}
```

下面的注入存在两个候选对象，Spring 可能无法判断应该选哪一个：

```java
@Autowired
private PayService payService;
```

可以使用 `@Qualifier` 明确指定：

```java
@Autowired
@Qualifier("aliPayService")
private PayService payService;
```

也可以使用 `@Resource` 指定 Bean 名称：

```java
@Resource(name = "aliPayService")
private PayService payService;
```

## 七、`@Override`：方法重写检查

正确拼写是 `@Override`，它是 Java 自带的注解，与 Spring 依赖注入无关。

接口：

```java
public interface PayService {
    void pay();
}
```

实现类：

```java
public class AliPayService implements PayService {

    @Override
    public void pay() {
        System.out.println("支付宝支付");
    }
}
```

`@Override` 表示当前方法正在实现接口中的方法，或者重写父类中的方法。

它还能让编译器帮助检查。如果错误地写成：

```java
@Override
public void pai() {
}
```

编译器会发现接口中没有 `pai()` 方法并报错。

## 八、Spring 项目的常见分层

一个常见的请求流程是：

```text
浏览器或 App
      ↓ HTTP 请求
Controller
      ↓ 调用
Service
      ↓ 调用
Repository / Mapper
      ↓
MySQL、Redis 等数据源
```

每一层有不同的职责：

| 层 | 主要职责 | 不适合承担的工作 |
| --- | --- | --- |
| Controller | 接收请求、解析参数、返回响应 | 复杂业务和直接操作数据库 |
| Service | 业务规则、流程编排、事务 | HTTP 请求和响应细节 |
| Repository / Mapper | 数据的增删改查 | 登录、下单等完整业务流程 |
| Component | 通用组件和辅助能力 | 应当保持职责单一 |

分层的主要目的不是让代码看起来复杂，而是让每一部分职责清楚、容易修改和测试。非常简单的功能也不必机械地创建大量空壳类。

## 九、`@Service`：业务逻辑层

```java
@Service
public class UserService {

    public void login(String phone, String code) {
        // 校验手机号
        // 校验验证码
        // 查询用户
        // 必要时创建用户
        // 生成登录令牌
    }
}
```

`@Service` 内部包含 `@Component`，所以它也能让 Spring 创建和管理对象。与直接使用 `@Component` 相比，`@Service` 更清楚地表达“这个类负责业务逻辑”。

并不是所有 Spring 注解都包含 `@Component`。只有 `@Service`、`@Controller`、`@Repository` 这类组件角色注解可以这样理解。`@Transactional`、`@PostMapping` 等注解有完全不同的职责。

## 十、`@Repository`：数据访问层

```java
@Repository
public class UserRepository {

    public User findById(Long id) {
        // 查询数据库
        return null;
    }

    public void save(User user) {
        // 保存到数据库
    }
}
```

Repository 负责数据访问，例如查询、新增、修改和删除。它不应该负责完整的登录、下单等业务流程。

`@Repository` 也包含 `@Component`。除此之外，Spring 还可以把某些底层持久化异常转换成统一的数据访问异常。

## 十一、`@Controller` 和 `@RestController`

### 1. `@Controller`

`@Controller` 用于接收 Web 请求，传统上更适合返回页面：

```java
@Controller
public class PageController {

    @GetMapping("/home")
    public String home() {
        return "home";
    }
}
```

这里的 `"home"` 可能被解释为视图名称，Spring 会寻找类似 `templates/home.html` 的页面。

### 2. `@RestController`

现代前后端分离项目通常返回 JSON，因此经常使用：

```java
@RestController
public class UserController {

    @GetMapping("/users/1")
    public User getUser() {
        return new User(1L, "小明");
    }
}
```

浏览器通常收到：

```json
{
  "id": 1,
  "name": "小明"
}
```

`@RestController` 可以理解为：

```java
@Controller
@ResponseBody
```

| 注解 | 返回值默认含义 | 常见场景 |
| --- | --- | --- |
| `@Controller` | 页面或视图名称 | 服务端渲染页面 |
| `@RestController` | HTTP 响应数据，常见为 JSON | 前后端分离、REST API |

## 十二、`@ResponseBody`：把返回值写进响应体

```java
@Controller
public class UserController {

    @ResponseBody
    @GetMapping("/users/1")
    public User getUser() {
        return new User(1L, "小明");
    }
}
```

执行过程大致是：

```text
Controller 返回 User 对象
        ↓
@ResponseBody 表示不要把它当作页面名称
        ↓
Spring 寻找合适的消息转换器
        ↓
Jackson 通常把 User 序列化成 JSON
        ↓
JSON 被写入 HTTP 响应体
```

Spring Boot Web 项目通常已经自动配置 Jackson，所以返回普通对象或集合时，一般不需要自己手动转换 JSON。

需要注意，`@ResponseBody` 的准确含义是“把返回结果写进 HTTP 响应体”，并不表示任何返回值都一定转换成 JSON。例如返回字符串时，浏览器可能收到普通文本：

```java
@ResponseBody
@GetMapping("/hello")
public String hello() {
    return "hello";
}
```

使用 `@RestController` 时，类中的方法已经具有 `@ResponseBody` 的效果，一般不需要重复添加。

## 十三、请求映射注解

### 1. `@RequestMapping`

`@RequestMapping` 用来建立请求和控制器之间的匹配关系。它可以写在类上，也可以写在方法上。

写在类上时，通常表示整个控制器的公共路径前缀：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @PostMapping
    public void createUser(@RequestBody User user) {
        userService.createUser(user);
    }
}
```

类上的路径和方法上的路径会拼接：

```text
类路径：   /users
方法路径： /{id}
最终路径： /users/{id}
```

因此，第一个方法接收：

```http
GET /users/10
```

第二个方法接收：

```http
POST /users
```

把公共路径放在类上可以减少重复，并表示这个 Controller 负责哪一类资源。

### 2. `@GetMapping`

`@GetMapping` 接收 HTTP GET 请求，通常用于查询：

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.getUser(id);
}
```

它相当于：

```java
@RequestMapping(value = "/{id}", method = RequestMethod.GET)
```

### 3. `@PostMapping`

`@PostMapping` 接收 HTTP POST 请求，通常用于提交数据、创建数据或执行操作：

```java
@PostMapping("/login")
public Result login(@RequestBody LoginForm form) {
    return userService.login(form);
}
```

前端可以发送：

```http
POST /login
Content-Type: application/json

{
  "phone": "13800000000",
  "code": "123456"
}
```

它相当于：

```java
@RequestMapping(value = "/login", method = RequestMethod.POST)
```

类似的注解还有：

- `@PutMapping`：通常用于完整修改；
- `@PatchMapping`：通常用于部分修改；
- `@DeleteMapping`：通常用于删除。

常见实践是：类上用 `@RequestMapping` 设置公共路径，方法上用 `@GetMapping`、`@PostMapping` 等声明具体 HTTP 方法和子路径。

## 十四、`@Mapper`：MyBatis 如何把接口变成 Bean

```java
@Mapper
public interface UserMapper {

    User selectById(Long id);
}
```

`UserMapper` 是接口，不能直接执行：

```java
new UserMapper(); // 错误
```

MyBatis 会在运行时为 Mapper 接口生成代理对象，然后把代理对象注册到 Spring 容器：

```text
UserMapper 接口
      ↓
MyBatis 生成代理对象
      ↓
代理对象被注册到 Spring 容器
      ↓
代理对象成为 Bean
      ↓
可以被 Service 注入
```

因此，虽然 `@Mapper` 通常不包含 `@Component`，对应的代理对象仍然可以成为 Spring Bean。这是 MyBatis 和 Spring 整合程序完成的。

如果 Mapper 很多，可以统一扫描：

```java
@SpringBootApplication
@MapperScan("com.hmdp.mapper")
public class HmDianPingApplication {
}
```

使用 `@MapperScan` 后，扫描范围内的 Mapper 接口通常不必逐个添加 `@Mapper`。

一个对象成为 Bean 不只有 `@Component` 这一种途径，还包括：

- 使用 `@Component` 及其派生注解；
- 使用 `@Bean` 方法；
- MyBatis 为 Mapper 创建并注册代理；
- Spring Boot 自动配置注册对象；
- 第三方框架通过 Spring 扩展机制注册对象。

## 十五、`@Transactional`：数据库事务

### 1. 事务解决什么问题

事务保证一组数据库操作要么全部成功，要么全部失败。

假设小明给小红转账 100 元：

```java
public void transfer() {
    accountMapper.subtractMoney(1L, 100);
    accountMapper.addMoney(2L, 100);
}
```

如果第一步成功、第二步失败，就会出现小明的钱被扣除而小红没有收到钱的问题。

加入事务：

```java
@Service
public class AccountService {

    @Transactional
    public void transfer() {
        accountMapper.subtractMoney(1L, 100);
        accountMapper.addMoney(2L, 100);
    }
}
```

执行结果：

```text
所有数据库操作成功 → 提交事务
中途出现符合条件的异常 → 回滚全部操作
```

### 2. Spring 大致如何实现事务

Spring 通常为目标 Bean 创建代理对象：

```text
Controller
   ↓
Spring 事务代理对象
   ↓
开启事务
   ↓
调用真正的 transfer()
   ↓
成功则提交，异常则回滚
```

可以用下面的伪代码帮助理解：

```java
openTransaction();

try {
    target.transfer();
    commitTransaction();
} catch (Exception e) {
    rollbackTransaction();
    throw e;
}
```

实际实现更加复杂，但核心思想就是由代理在业务方法执行前后管理事务。

### 3. 默认回滚规则

默认情况下：

- 抛出 `RuntimeException` 时回滚；
- 抛出 `Error` 时回滚；
- 普通受检异常默认不一定回滚。

如果希望遇到所有 `Exception` 都回滚，可以写：

```java
@Transactional(rollbackFor = Exception.class)
public void transfer() throws Exception {
}
```

### 4. 可以放在方法或类上

放在方法上：

```java
@Transactional
public void createOrder() {
}
```

主要影响这个方法。

放在类上：

```java
@Service
@Transactional
public class OrderService {
}
```

通常会影响类中符合条件的公共方法。

### 5. 常见失效原因

#### 用在不合适的方法上

使用默认代理机制时，事务通常应该用在从 Bean 外部调用的 `public` 方法上。直接放在 `private` 方法上通常不能按预期工作。

#### 同一个类内部直接调用

```java
@Service
public class OrderService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
    }
}
```

`methodA()` 直接通过当前对象调用 `methodB()`，可能绕过 Spring 代理，导致 `methodB()` 上的事务不生效。

#### 把异常捕获后不再抛出

```java
@Transactional
public void createOrder() {
    try {
        saveOrder();
    } catch (Exception e) {
        // 异常被完全处理，没有继续抛出
    }
}
```

代理没有收到异常，可能认为方法执行成功并提交事务。

#### 数据库不支持事务

底层数据库和存储引擎本身需要支持事务。例如 MySQL 的 InnoDB 支持事务。

## 十六、一个完整示例

### Controller

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }
}
```

这里没有给构造方法写 `@Autowired`，因为类只有一个构造方法，Spring 会自动使用它完成注入。

### Service

```java
@Service
public class UserService {

    private final UserMapper userMapper;

    public UserService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    public User getUser(Long id) {
        if (id == null || id <= 0) {
            throw new IllegalArgumentException("用户 ID 不正确");
        }

        return userMapper.selectById(id);
    }

    @Transactional(rollbackFor = Exception.class)
    public User createUser(User user) {
        userMapper.insert(user);
        return user;
    }
}
```

### Mapper

```java
@Mapper
public interface UserMapper {

    User selectById(Long id);

    void insert(User user);
}
```

### 查询请求执行过程

```text
GET /users/10
      ↓
@GetMapping 匹配请求
      ↓
UserController.getUser(10)
      ↓
UserService.getUser(10)
      ↓
UserMapper 代理对象查询数据库
      ↓
返回 User 对象
      ↓
@RestController 使返回值进入响应体
      ↓
Jackson 将 User 转成 JSON
      ↓
JSON 返回给前端
```

## 十七、注解关系和分类

### 1. 用于把对象交给 Spring 管理

```text
@Component
├── @Service
├── @Repository
└── @Controller

@RestController ≈ @Controller + @ResponseBody
```

这些注解中，`@Service`、`@Repository` 和 `@Controller` 都具有 `@Component` 的组件注册能力，同时表达更加具体的职责。

### 2. 用于依赖注入

```text
构造方法注入（推荐）
@Autowired
@Resource
```

### 3. 用于处理 Web 请求

```text
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@ResponseBody
```

### 4. 用于数据库或框架能力

```text
@Mapper          → 标记 MyBatis Mapper 接口
@Transactional   → 声明事务边界
```

### 5. Java 自身的注解

```text
@Override → 表示实现接口方法或重写父类方法
```

## 十八、最后的记忆口诀

```text
类是设计图，对象是实例，Bean 是 Spring 管理的对象。

@Component：这是一个需要 Spring 管理的通用组件。
@Service：这是处理业务逻辑的 Bean。
@Repository：这是访问数据的 Bean。
@Controller：这是接收请求、通常返回页面的 Bean。
@RestController：这是接收请求、通常返回 JSON 的 Bean。

构造方法：Spring 可以通过它自动传入依赖。
@Autowired：主要按类型注入 Bean。
@Resource：通常优先按名称，再按类型注入 Bean。

@RequestMapping：声明公共路径或通用请求映射。
@GetMapping：接收 GET 请求。
@PostMapping：接收 POST 请求。
@ResponseBody：把方法返回值写进 HTTP 响应体。

@Mapper：让 MyBatis 为接口生成代理对象并注册到 Spring。
@Transactional：一组数据库操作全部成功或全部回滚。
@Override：检查方法是否正确重写父类或实现接口方法。
```

理解 Spring 的关键不是背注解，而是抓住这条主线：

> Spring 发现并管理对象，使用依赖注入把对象组装起来，再通过 Web、事务等模块为这些对象提供基础能力。
