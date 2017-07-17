# 控制并发访问

本章详细介绍了如何处理对实体数据的并发访问以及 Java Persistence API 应用程序开发人员可用的锁定策略。

主要讨论以下主题：

* 实体锁定和并发概述
* 锁定模式


## 实体锁定和并发概述

如果数据源中的数据由多个应用程序同时访问，则会涉及并发访问实体数据的问题。我们需要确保在并发访问基础数据的数据的完整性。

当在具有事务管理的数据库表中更新数据时，持久化提供程序假定数据库管理系统将保存短期读锁和长期写锁，以保持数据完整性。大多数持久性提供者将延迟数据库写入，直到事务结束，除非应用程序显式地调用flush，即，应用程序调用 EntityManager.flush 方法或执行查询时使用 flush 模式设置为 AUTO。

默认情况下，持久化提供程序使用乐观锁（optimistic locking），其中在提交对数据的更改之前，持久化提供程序检查自从读取数据以来没有其他事务修改或删除数据。这是通过数据库表中的版本列实现的，在实体类中有相应的版本属性。当行被修改时，版本值增加。原始事务检查版本属性，如果数据已被另一个事务修改，将抛出 javax.persistence.OptimisticLockException ，并且原始事务将回滚。当应用程序指定乐观锁模式时，即使实体数据未被修改，持久化提供程序也要验证特定实体在从数据库读取后未更改。

悲观锁（pessimistic locking）比乐观锁更进一步。使用悲观锁，持久化提供者创建一个事务，该事务获得对数据的长期锁定，直到事务完成，这防止其他事务在锁定结束之前修改或删除数据。当底层数据经常被许多事务访问和修改时，悲观锁是比乐观锁更好的策略。

>注意：
对不经常修改的实体使用悲观锁可能会导致应用程序性能下降。

### 使用乐观锁

使用 javax.persistence.Version 注解将持久化字段或属性标记为实体的版本属性。 version 属性使实体能够进行乐观并发控制。当在事务期间修改实体实例时，持久化提供程序读取并更新版本属性。应用程序可以读取版本属性，但不能修改值。

>注意：
虽然一些持久化提供程序可能支持没有版本属性的实体的乐观锁，但是在使用乐观锁时，便携式应用程序应始终使用具有版本属性的实体。如果应用程序尝试锁定没有版本属性的实体，并且持久化提供程序不支持非版本化实体的乐观锁定，那么将抛出 PersistenceException。

`@Version`注解具有以下要求。

* 每个实体只能定义一个`@Version`属性。
* 对于映射到多个表的实体，`@Version`属性必须在主表中。
* `@Version`属性的类型必须为以下之一：int、Integer、long、Long、short、Short 或 java.sql.Timestamp。

以下代码段显示如何在具有持久化字段的实体中定义版本属性：

```
@Version
protected int version;
```

以下代码段显示如何在具有持久化属性的实体中定义版本属性：

```
@Version
protected Short getVersion() { ... }
```

## 锁定模式

应用程序可以通过指定锁模式的使用来增加实体的锁定级别。 可以指定锁定模式以增加乐观锁定的级别或请求使用悲观锁定。

使用乐观锁模式导致持久性提供程序检查在事务期间读取（但未修改）的实体的版本属性以及更新的实体的版本属性。

使用悲观锁定模式指定持久性提供者将立即获取对应于实体状态的数据库数据的长期读取或写入锁定。

您可以通过指定在下表中列出的 javax.persistence.LockModeType 枚举类型中定义的锁定模式之一来为实体操作设置锁定模式。



锁定模式 | 说明
---- | ----
OPTIMISTIC | 为具有版本属性的所有实体获取乐观读锁定。
OPTIMISTIC_FORCE_INCREMENT | 对具有版本属性的所有实体获取乐观读锁定，并增加版本属性值。
PESSIMISTIC_READ | 立即获取对数据的长期读锁定，以防止数据被修改或删除。其他事务可以在保持锁定时读取数据，但不能修改或删除数据。持久性提供程序在请求读锁定时被允许获取数据库写锁定，但反之亦然。
PESSIMISTIC_WRITE | 立即获取对数据的长期写锁定，以防止读取，修改或删除数据。
PESSIMISTIC_FORCE_INCREMENT | 立即获取对数据的长期锁定，以防止数据被修改或删除，并增加版本化实体的版本属性。
READ | OPTIMISTIC 的同义词。使用 LockModeType.OPTIMISTIC 是新应用程序的首选。
WRITE | OPTIMISTIC_FORCE_INCREMENT 的同义词。使用 LockModeType.OPTIMISTIC_FORCE_INCREMENT 是新应用程序的首选。
NONE | 数据库中的数据不会发生额外的锁定。


### 设置锁定模式

要指定锁定模式，请使用以下方法之一。

* 调用 EntityManager.lock 方法，传递其中一种锁定模式：

```
EntityManager em = ...;
Person person = ...;
em.lock(person, LockModeType.OPTIMISTIC);
```

* 调用以锁定模式作为参数的 EntityManager.find 方法之一：

```
EntityManager em = ...;
String personPK = ...;
Person person = em.find(Person.class, personPK, 
    LockModeType.PESSIMISTIC_WRITE);
```

* 调用以锁定模式作为参数的 EntityManager.refresh 方法之一：

```
EntityManager em = ...;
String personPK = ...;
Person person = em.find(Person.class, personPK);
...
em.refresh(person, LockModeType.OPTIMISTIC_FORCE_INCREMENT);
```

* 调用 Query.setLockMode 或 TypedQuery.setLockMode 方法，传递锁定模式作为参数：

```
Query q = em.createQuery(...);
q.setLockMode(LockModeType.PESSIMISTIC_FORCE_INCREMENT);
```

* 向`@NamedQuery`注解中添加一个 lockMode 元素：

```
@NamedQuery(name="lockPersonQuery",
  query="SELECT p FROM Person p WHERE p.name LIKE :name",
  lockMode=PESSIMISTIC_READ)
```



### 使用悲观锁

版本化实体以及没有版本属性的实体可以被悲观锁。

要以悲观的方式锁定实体，请将锁定模式设置为 PESSIMISTIC_READ、PESSIMISTIC_WRITE 或 PESSIMISTIC_FORCE_INCREMENT。

如果无法在数据库行上获取悲观锁，并且无法锁定数据导致事务回滚，则会抛出 PessimisticLockException。如果无法获取悲观锁，但锁定失败不会导致事务回滚，则会抛出 LockTimeoutException。

悲观地用 PESSIMISTIC_FORCE_INCREMENT 锁定版本化实体会导致版本属性增加，即使实体数据未修改。当悲观锁定版本化实体时，持久性提供程序将执行在乐观锁定期间发生的版本检查，如果版本检查失败，将抛出 OptimisticLockException。尝试使用 PESSIMISTIC_FORCE_INCREMENT 锁定非版本化实体不可移植，如果持久性提供程序不支持非版本化实体的乐观锁，则可能导致 PersistenceException。如果事务成功提交，则使用 PESSIMISTIC_WRITE 锁定版本化实体会导致版本属性递增。

#### 悲观锁超时

使用 javax.persistence.lock.timeout 属性指定持久性提供程序应等待的时间长度（以毫秒为单位），以获取数据库表上的锁定。如果获取锁定所需的时间超过此属性的值，将抛出 LockTimeoutException，但不会将当前事务标记为回滚。如果将此属性设置为0，持久性提供程序应抛出 LockTimeoutException，如果它不能立即获得锁定。

>注意：
可移植应用程序不应该依赖 javax.persistence.lock.timeout 的设置，因为锁定策略和底层数据库可能意味着超时值不能使用。 javax.persistence.lock.timeout 的值是一个提示，而不是约定。

可以通过将其传递给允许指定锁模式的 EntityManager 方法，Query.setLockMode 和 TypedQuery.setLockMode 方法，`@NamedQuery`注解和 Persistence.createEntityManagerFactory 方法来以编程方式设置此属性。它也可以设置为 persistence.xml 部署描述符中的一个属性。

如果 javax.persistence.lock.timeout 在多个位置设置，将按以下顺序确定该值：

* 一个 EntityManager 或 Query 方法的参数
* `@NamedQuery` 注解中的设置
* Persistence.createEntityManagerFactory 方法的参数
* persistence.xml 部署描述符中的值