# Spring 框架中的 `@Transactional` 注解入门指南

## 1. 什么是事务？

在数据库操作中，**事务（Transaction）** 是指一组操作的集合，这些操作要么全部成功执行，要么全部失败回滚。事务具有以下四个特性，通常称为 **ACID** 特性：

- **原子性（Atomicity）**：事务中的所有操作要么全部完成，要么全部不完成。
- **一致性（Consistency）**：事务执行前后，数据的状态保持一致。
- **隔离性（Isolation）**：多个事务并发执行时，彼此之间互不干扰。
- **持久性（Durability）**：事务一旦提交，数据的更改将永久保存。

在 Java 应用程序中，事务通常涉及对数据库的操作（如插入、更新、删除等）。Spring 框架提供了声明式事务管理（Declarative Transaction Management），通过 `@Transactional` 注解简化事务的使用。

---

## 2. 什么是 `@Transactional`？

`@Transactional` 是 Spring 框架中用于管理事务的核心注解。它可以帮助开发者以声明式的方式实现事务管理，而无需手动编写事务的开启、提交或回滚代码。

### 主要功能：
- **自动开启事务**：当方法被调用时，Spring 自动开启一个事务。
- **自动提交事务**：如果方法正常执行完成，Spring 会自动提交事务。
- **自动回滚事务**：如果方法抛出未捕获的异常（默认是运行时异常 `RuntimeException` 或错误 `Error`），Spring 会自动回滚事务。
- **支持传播行为和隔离级别**：可以通过注解属性配置事务的行为。

---

## 3. 如何使用 `@Transactional`？

`@Transactional` 可以应用于类或方法上。以下是其基本用法：

### （1）**在方法上使用**

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
        // 如果这里抛出异常，事务会回滚
    }
}
```

### （2）**在类上使用**

如果在类上使用 `@Transactional`，则该类中的所有公共方法都会启用事务管理。

```java
@Service
@Transactional
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public void createUser(User user) {
        userRepository.save(user);
    }

    public void updateUser(User user) {
        userRepository.update(user);
    }
}
```

---

## 4. 自定义事务行为

`@Transactional` 提供了一些属性，可以用来定制事务的行为：

- **propagation**：事务传播行为（如是否需要新事务）。
- **isolation**：事务隔离级别（如读已提交、可重复读等）。
- **timeout**：事务超时时间（单位为秒）。
- **readOnly**：是否为只读事务。
- **rollbackFor** 和 **noRollbackFor**：指定哪些异常触发回滚或不触发回滚。

### 示例：

```java
@Transactional(
    propagation = Propagation.REQUIRED, // 默认传播行为
    isolation = Isolation.READ_COMMITTED, // 隔离级别
    timeout = 30, // 超时时间
    readOnly = false, // 是否只读
    rollbackFor = {SQLException.class}, // 指定回滚的异常
    noRollbackFor = {IllegalArgumentException.class} // 不回滚的异常
)
public void complexOperation() {
    // 复杂的事务逻辑
}
```

---

## 5. 事务传播行为（Propagation）

事务传播行为定义了当前事务方法如何与外部事务交互。常见的传播行为包括：

- **REQUIRED（默认）**：如果当前存在事务，则加入该事务；否则创建一个新事务。
- **REQUIRES_NEW**：总是创建一个新事务，挂起当前事务（如果有）。
- **SUPPORTS**：如果当前存在事务，则加入该事务；否则以非事务方式运行。
- **NOT_SUPPORTED**：以非事务方式运行，挂起当前事务（如果有）。
- **MANDATORY**：必须在一个已有的事务中运行，否则抛出异常。
- **NEVER**：必须以非事务方式运行，如果当前存在事务，则抛出异常。
- **NESTED**：如果当前存在事务，则在嵌套事务中运行；否则创建一个新事务。

### 示例：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void createOrder(Order order) {
    // 总是创建一个新事务
}
```

---

## 6. 事务隔离级别（Isolation）

事务隔离级别定义了事务之间的可见性和并发控制。常见的隔离级别包括：

- **DEFAULT**：使用底层数据库的默认隔离级别。
- **READ_UNCOMMITTED**：允许读取未提交的数据（可能导致脏读、不可重复读、幻读）。
- **READ_COMMITTED**：只能读取已提交的数据（防止脏读，但可能出现不可重复读和幻读）。
- **REPEATABLE_READ**：保证同一事务中多次读取的结果一致（防止脏读和不可重复读，但可能出现幻读）。
- **SERIALIZABLE**：最高隔离级别，完全串行化执行事务（防止脏读、不可重复读和幻读，但性能较差）。

### 示例：

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void readData() {
    // 读取已提交的数据
}
```

---

## 7. 事务回滚规则

默认情况下，Spring 只会在方法抛出 `RuntimeException` 或 `Error` 时回滚事务。如果需要针对特定的检查异常（Checked Exception）进行回滚，可以使用 `rollbackFor` 属性。

### 示例：

```java
@Transactional(rollbackFor = SQLException.class)
public void insertData() throws SQLException {
    // 如果抛出 SQLException，事务会回滚
}
```

---

## 8. 注意事项

- **代理机制**：`@Transactional` 的实现依赖于 Spring AOP，因此它只能作用于被 Spring 管理的 Bean 的公共方法（`public` 方法）。私有方法或非 Spring 管理的类无法生效。
- **自调用问题**：如果一个类中的方法直接调用另一个带有 `@Transactional` 注解的方法，事务可能不会生效，因为 Spring 的代理机制无法拦截内部调用。
- **只读事务**：对于查询操作，建议设置 `readOnly = true`，这样可以优化性能（如避免不必要的锁）。

---

## 9. 总结

`@Transactional` 是 Spring 中非常重要的注解，用于简化事务管理。通过合理配置事务的传播行为、隔离级别和回滚规则，可以确保应用程序的数据库操作满足事务的 ACID 特性。使用时需要注意代理机制和自调用问题，以避免潜在的坑。

希望这份文档能帮助你更好地理解和使用 `@Transactional`！如果你有任何疑问，欢迎随时提问！
