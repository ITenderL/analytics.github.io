# Spring事务详解

# 什么是事务

**事务是逻辑上的一组操作，要么都执行，要么都不执行。**

系统中每个业务方法可能包括了多个原子性的数据库操作，比如下面的 `saveUser()` 方法中就有两个原子性的数据库操作。这些原子性的数据库操作是有依赖的，它们要么都执行，要不就都不执行。

``` java
  public void save(User user) {
    userMapper.save(user);
    userDetailMapper.save(user.getDetail());
  }
```

事务能否生效，跟数据库的存储引擎是否支持事务是有关的。MySQL数据库默认的存储引擎是Innodb，默认是支持事务的。

# 事务的四大特性

MySQL 事务具有以下四大特性：

**原子性（Atomicity）：**事务是最小的执行单元，事务中的所有操作要么全部执行成功，要么全部失败回滚，不能只执行其中一部分操作。

**一致性（Consistency）：**事务执行前后，数据库的完整性约束没有被破坏，数据总是从一个一致性状态转移到另一个一致性状态。例如，如果一个事务要求将某个账户的金额从 A 转移到 B，那么无论事务是否成功，最终账户 A 和账户 B 的总金额应该保持不变。

**隔离性（Isolation）：**并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；

**持久性（Durability）：**事务完成后，对数据库的修改将永久保存在数据库中，即使系统故障也不会丢失。

**只有保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。也就是说 A、I、D 是手段，C 是目的。**

事务四大特性是为了保证数据库的数据一致性和可靠性的，使得数据库在并发访问和故障恢复等复杂环境下，仍能保持数据的完整性。

这些特性对于许多应用场景，尤其是需要处理关键业务数据的应用，是非常重要的。例如在转账业务中，它分为两个关键性操作，首先是先扣除一个账户的钱，其次再给另一个账号增加钱。但是如果没有事务的保证，那么有可能第一次操作钱被扣了，但另一个账户钱没增加，那么这笔钱就凭空“消失”了。

# 事务的隔离级别

在 MySQL 中，事务的隔离级别指的是多个并发事务之间的隔离程度，它有四个级别：读未提交（Read Uncommitted）、读已提交（Read Committed）、可重复读（Repeatable Read）和串行化（Serializable）。

它们的具体区别如下：

**READ-UNCOMMITTED(读取未提交)** ：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

**READ-COMMITTED(读取已提交)** ：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。

**REPEATABLE-READ(可重复读)** ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

**SERIALIZABLE(可串行化)** ：最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

MySQL的默认隔离级别为**REPEATABLE-READ(可重复读)**。

# Spring事务

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring 是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过 undo log或者 redo log 实现的。

一般我们在程序里面使用的都是在方法上面加  @Transaction 注解，这种属于声明式事物。

声明式事务本质是通过 AOP 功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

# Spring支持事务的方式

Spring支持事务有两种方式，声明式事务和编程式事务。

### 编程式事务管理

通过 `TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。

使用`TransactionTemplate` 进行编程式事务管理的示例代码如下

```java
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
            try {
                // ....  业务代码
            } catch (Exception e){
                //回滚
                transactionStatus.setRollbackOnly();
            }
        }
    });
}
```

使用 `TransactionManager` 进行编程式事务管理的示例代码如下：

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void testTransaction() {

  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
          try {
               // ....  业务代码
              transactionManager.commit(status);
          } catch (Exception e) {
              transactionManager.rollback(status);
          }
}
```

### 声明式事务

推荐使用（代码侵入性最小），实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）。

使用 `@Transactional`注解进行事务管理的示例代码如下：

```java
@Transactional(rollbackFor = Exception.class)
public void saveUser(User user) throws IOException {
    userMapper.saveUser(user);
    throw new IOException();
}
```

# Spring事务传播行为

Spring 事务的传播行为说的是，当多个事务同时存在的时候， Spring 如何处理这些事务的行为。

**PROPAGATION_REQUIRED**：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设

**PROPAGATION_SUPPORTS**：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。

**PROPAGATION_MANDATORY：**支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

**PROPAGATION_REQUIRES_NEW：**创建新事务，无论当前存不存在事务，都创建新事务，如果当前存在事务，把当前事务挂起。

**PROPAGATION_NOT_SUPPORTED：**以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

**PROPAGATION_NEVER：**以非事务方式执行，如果当前存在事务，则抛出异常。

**PROPAGATION_NESTED：**如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按 REQUIRED 属性执行。

当传播行为设置了PROPAGATION_NOT_SUPPORTED，PROPAGATION_NEVER，PROPAGATION_SUPPORTS这三种时，就有可能存在事物不生效。

# Spring事务失效场景

## 1. 事务方法没有被Spring管理

如果事务方法所在的类没有注册到`Spring IOC`容器中，也就是说，事务方法所在类并没有被`Spring`管理，则`Spring`事务会失效。

``` java
// @Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Transactional(rollbackFor = Exception.class)
    public void saveUser(User user) {
        userMapper.saveUser(user);
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateUser(User user) {
        userMapper.updateUser(user);
    }
}

```

**解决方案**

类注册到Spring容器，保证每个事务注解的每个Bean被Spring管理。。

## 2. 非public修饰的方法

Java的访问权限主要有四种：private、default、protected、public，它们的权限从左到右，依次变大。

但如果我们在开发过程中，把有某些事务方法，定义了错误的访问权限，就会导致事务功能出问题。

**访问权限被定义成了private，这样会导致事务失效，spring要求被代理方法必须是public的。**

```java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;
    
    @Transactional(rollbackFor = Exception.class)
    private void saveUser(User user) {
        userMapper.saveUser(user);
    }
}
```

**解决方案**

使用public修饰事务的方法。

## 3. 方法使用final或static类型修饰

如果`Spring`使用了`Cglib`代理实现（比如你的代理类没有实现接口），而你的业务方法恰好使用了`final`或者`static`关键字，那么事务也会失败。更具体地说，它应该抛出异常，因为`Cglib`使用字节码增强技术生成被代理类的子类并重写被代理类的方法来实现代理。如果被代理的方法的方法使用`final`或`static`关键字，则子类不能重写被代理的方法。

如果`Spring`使用`JDK`动态代理实现，`JDK`动态代理是基于接口实现的，那么`final`和`static`修饰的方法也就无法被代理。

总而言之，方法连代理都没有，那么肯定无法实现事务回滚了。

``` java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;
    
    @Transactional(rollbackFor = Exception.class)
    public final void saveUser(User user) {
        userMapper.saveUser(user);
    }
}
```

**解决方案**

想办法去掉final或者static关键字。

## 4. 数据库或表本身不支持事务

Spring事务时依赖于数据库的，如果数据库不支持事务，Spring事务也不生效。

要确保你用的数据库和表是支持事务的。

## 5. 异常被捕获，没有抛出

如果`@Transactional` 没有特别指定，Spring 只会在遇到运行时异常RuntimeException或者error时进行回滚。

```java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Transactional
    public void saveUser(User user) {
        try {
            userMapper.saveUser(user);
            int i = 1 / 0;
        } catch (Exception e) {
			e.printStackTrace();
        }
    }
}

```

**解决方案**

不要捕获异常，如果没有设置@Transactional(rollbackFor = Exception.class)，那就直接抛出RuntimeException

```java
@Transactional(rollbackFor = Exception.class)
public void saveUser(User user) {
    try {
        userMapper.saveUser(user);
        int i = 1 / 0;
    } catch (Exception e) {
        throw new RuntimeException("事务回滚");
    }
}
```

## 6. 抛出可检查异常，事务注解没有指定回滚异常类型

如果`@Transactional` 没有特别指定，Spring 只会在遇到运行时异常RuntimeException或者error时进行回滚，而`IOException`等检查异常不会影响回滚。

```java
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Transactional
    public void saveUser(User user) throws IOException {
        userMapper.saveUser(user);
        throw new IOException();
    }
}

```

**解决方案：**

知道原因后，解决方法也很简单。配置`rollbackFor`属性，例如`@Transactional(rollbackFor = Exception.class)`。

## 7. 传播机制使用的不合理，不支持事务或不使用同一个事务

`Spring`事务的传播机制是指在多个事务方法相互调用时，确定事务应该如何传播的策略。`Spring`提供了七种事务传播机制：`REQUIRED`、`SUPPORTS`、`MANDATORY`、`REQUIRES_NEW`、`NOT_SUPPORTED`、`NEVER`、`NESTED`。如果不知道这些传播策略的原理，很可能会导致交易失败。

```java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Resource
    private TestTransactionService2 testTransactionService2;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void saveUser(User user) {
        userMapper.saveUser(user);
        testTransactionService2.updateUser(new User());
        throw new RuntimeException("测试事务回滚");
    }
}

```



```java
@Service
public class TestTransactionService2 {
    @Resource
    private UserMapper userMapper;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void saveUser() {
        User user = new User();
        user.setName("张三");
        user.setUsername("zhangsan");
        user.setPassword("123456");
        userMapper.saveUser(user);
    }

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void updateUser(User user) {
        user.setId(1);
        user.setName("胖胖");
        user.setUsername("pangpang");
        userMapper.updateUser(user);
    }
}
```

测试结果：

TestTransactionService没有插入成功。

TestTransactionService2更新成功

上面的代码中，`saveUser()`插入失败，`updateUser()`方法也不会回滚，因为这里使用的传播是`REQUIRES_NEW`，传播机制`REQUIRES_NEW`的原理是如果当前方法中没有事务，就会创建一个新的事务。如果一个事务已经存在，则当前事务将被挂起，并创建一个新事务。在当前事务完成之前，不会提交父事务。如果父事务发生异常，则不影响子事务的提交。

当传播行为设置了PROPAGATION_NOT_SUPPORTED，PROPAGATION_NEVER，PROPAGATION_SUPPORTS这三种时，也有可能存在事物不生效。

**解决方案**:

将事务传播策略更改为默认值`REQUIRED`。`REQUIRED`原理是如果当前有一个事务被添加到一个事务中，如果没有，则创建一个新的事务，父事务和被调用的事务在同一个事务中。即使被调用的异常被捕获，整个事务仍然会被回滚。

## 8. 多线程调用事务方法(事务不在同一个线程中执行)

在TestTransactionService事务的方法中，开启多线程去调用TestTransactionService2事务方法，TestTransactionService2在更新的方法中抛出运行时异常。TestTransactionService插入成功，TestTransactionService2更新失败回滚。

```java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Resource
    private TestTransactionService2 testTransactionService2;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void saveUser(User user) {
        userMapper.saveUser(user);
        new Thread(() -> testTransactionService2.updateUser(new User())).start();
    }
}

```

```java
@Service
public class TestTransactionService2 {
    @Resource
    private UserMapper userMapper;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void updateUser(User user) {
        user.setId(1);
        user.setName("胖胖");
        user.setUsername("pangpang");
        userMapper.updateUser(user);
        throw new RuntimeException("更新失败");
    }
}

```



在TestTransactionService事务的方法中，开启多线程去调用TestTransactionService2事务方法，TestTransactionService在新增的方法中抛出运行时异常。TestTransactionService插入失败，TestTransactionService2更新成功。

```java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Resource
    private TestTransactionService2 testTransactionService2;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void saveUser(User user) {
        userMapper.saveUser(user);
        new Thread(() -> testTransactionService2.updateUser(new User())).start();
        throw new RuntimeException("插入失败");
    }
}
```

```java
@Service
public class TestTransactionService2 {
    @Resource
    private UserMapper userMapper;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void updateUser(User user) {
        user.setId(1);
        user.setName("胖胖");
        user.setUsername("pangpang");
        userMapper.updateUser(user);
    }
}
```

经过上面的测试，两个方法不在同一个线程执行，导致多个线程获取到的数据库连接不是同一个，执行的事务也不是同一个。导致最终的数据不一致的问题。

**解决方案**

尽量保证事务方法在同一个线程中执行。

## 9. 同一个类中的方法相互调用

在我们日常的开发工作中，经常遇到同一个类中的方法互相调用的场景。

### 9.1 同一类中非事务的方法调用事务的方法，事务不生效

```java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    // @Transactional(rollbackFor = Exception.class)
    public void saveUser(User user) {
        userMapper.saveUser(user);
        updateUser(user);
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateUser(User user) {
        user.setId(1);
        user.setName("胖胖");
        user.setUsername("pangpang");
        userMapper.updateUser(user);
        int i = 1 / 0;
    }
}
```

**解决方案**

1. 新建一个service，把同类中调用的方法，放到新的service中，通过注入新的service进行调用。
2. 使用**AopContext.currentProxy()** 为当前service生成一个代理对象，使用代理对象进行调用。

### 9.2 同一类中事务方法调用事务方法，事务生效

``` java
@Service
public class TestTransactionService {
    @Resource
    private UserMapper userMapper;

    @Transactional(rollbackFor = Exception.class)
    public void saveUser(User user) {
        userMapper.saveUser(user);
        updateUser(user);
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateUser(User user) {
        user.setId(1);
        user.setName("胖胖");
        user.setUsername("pangpang");
        userMapper.updateUser(user);
        int i = 1 / 0;
    }
}
```

# Spring事务的原理

对于Spring事务的原理大家应该也不会陌生。大家多多少少都会有所了解。但是还是要说一句，Spring的事务是依赖与数据库的事务完成的，如果数据库没有提供事务的支持，Spring也无法提供事务支持。

Spring事务既然是依赖于数据库来实现的，那么就需要执行以下步骤：

1. 获取数据库连接
2. 开启一个事务
3. 执行SQL，业务逻辑
4. 提交/回滚事务
5. 关闭连接

> @Transactional 是基于 AOP 实现的，AOP ⼜是使⽤动态代理实现的。如果⽬标对象实现了接⼝，默 认情况下会采⽤ JDK 的动态代理，如果⽬标对象没有实现了接⼝,会使⽤ CGLIB 动态代理。 
>
> @Transactional 在开始执⾏业务之前，通过代理先开启事务，在执⾏成功之后再提交事务。如果中途 遇到的异常，则回滚事务。

# 总结

本文主要讲了Spring事务，包括事务概念，事务的隔离级别，Spring事务的传播行为，Spring事务失效的场景和Spring事务实现的原理。

希望对大家有帮助，大家一起学习，一起进步。



