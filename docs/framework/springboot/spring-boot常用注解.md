# SpringBoot常用注解

# 前言

最近的面试中，不止一次的被问到了SpringBoot的常用注解。正好顺便来梳理一下，做个总结。

# 常用注解

## 1. @SpringBootApplication

@SpringBootApplication是SpringBoot的一个核心注解，作用在SpringBoot的启动类上。

当我们在创建一个SpringBoot项目时，会创建一个启动类。

```java
@SpringBootApplication
public class BaseApplication {

    public static void main(String[] args) {
        SpringApplication.run(BaseApplication.class, args);
    }
}
```

我们点进@SpringBootApplication注解，会发现他包含了三个注解@SpringBootConfiguration, @EnableAutoConfiguration, @ComponentScan。

``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

@SpringBootConfiguration：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类

@EnableAutoConfiguration：启动SpringBoot自动配置

@ComponentScan：Bean扫描，扫描那些被Spring管理的Bean。如：@Controller, @Component等

## 2. Spring Bean相关的注解

### @Component，@Controller，@Service，@Repository

`@Component`：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。

`@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。

`@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。

`@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

### @Configuration

我们一般用来声明这个类是一个配置类。

### @Bean

一般作用在方法上，用来注入一个Spring Bean。

``` java
@Bean
@ConditionalOnClass(ThreadPoolExecutor.class)
public ThreadPoolExecutor myThreadPool() {
    return new ThreadPoolExecutor(10, 10, 10, TimeUnit.SECONDS,new ArrayBlockingQueue<>(100));
}
```

### @RestController

@RestController可以看作是@Controller和@ResponseBody两个注解,表示这是个控制器 bean,并且是将函数的返回值直接填入 HTTP 响应体中,是 REST 风格的控制器。

### @Scope

声明 Spring Bean 的作用域。

``` java
@Bean
@Scope("singleton")
public User getUser() {
    return new User();
}
```

singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。

prototype : 每次请求都会创建一个新的 bean 实例。

request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。

session : 每一个 HTTP Session 会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

### @Autowried

自动导入对象到类中，被注入进的类同样要被 Spring 容器管理比如：Service 类注入到 Controller 类中。

```java
@RestController
@RequestMapping("excel")
public class ExcelUserController {

    @Autowired
    private  UserService userService;

}
```

## 3. Http请求相关的注解

## @RequestMapping，@PostMapping，@GetMapping，@PutMapping，@DeleteMapping

用来标识接口的请求路径。

@RequestMapping：作用在类上。

@PostMapping：post请求的

@GetMapping：get请求的

@PutMapping：update请求

@DeleteMapping：delete请求

### @RequestBody

作用于post请求的参数体。

```java
@PostMapping("/add")
public String add(@RequestBody User user) {
    return "ok";
}
```

### @RequestParam,@PathVariable

`@PathVariable`：用于获取路径参数，rest请求。

`@RequestParam`：用于获取查询参数，url?parm=。

```java
@GetMapping("/user/{userId}/list")
public List<User> userList(
         @PathVariable("userId") Long userId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```

## 4.全局异常处理

1. `@ControllerAdvice` :注解定义全局异常处理类
2. `@ExceptionHandler` :注解声明异常处理方法

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * 请求参数异常处理
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       ......
    }
}
```

## 5. 事务相关@Transactional

在要开启事务的方法上使用`@Transactional`注解即可!

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
  ......
}
```

## 6. 读取配置

### @Value

使用 `@Value("${property}")` 读取比较简单的配置信息：

```java
@Value("${auth.url}")
private String authUrl;
```

#### @ConfigurationProperties

通过`@ConfigurationProperties`读取配置信息并与 bean 绑定.

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  省略getter/setter
  ......
}
```

### `@PropertySource`

`@PropertySource`读取指定 properties 文件

```java
@Component
@PropertySource("classpath:website.properties")

class WebSite {
    @Value("${url}")
    private String url;

  省略getter/setter
  ......
}
```

# 总结

这里主要列举了一些比较常用的注解，可能不是很全面，希望对大家有帮助。