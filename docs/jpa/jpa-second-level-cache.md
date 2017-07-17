# 使用二级缓存

本章介绍如何修改二级缓存模式设置以提高使用 Java Persistence API 的应用程序的性能。

主要讨论以下主题：

* 二级缓存概述
* 指定缓存模式设置以提高性能


## 二级缓存概述

二级缓存（ second-level cache）是由持久化提供者管理的实体数据的本地存储，以提高应用程序性能。二级缓存通过避免昂贵的数据库调用，将实体数据保留在应用程序本地来帮助提高性能。二级缓存通常对应用程序是透明的，因为它由持久化提供程序管理，并且是应用程序的持久化上下文的基础。也就是说，应用程序通过正常的实体管理器操作读取并提交数据，而无需知道缓存的存在。

>注意：
持久化提供程序不需要支持二级缓存。便携式应用程序不应该依赖持久化提供程序对二级缓存的支持。

用于持久化单元的第二级缓存可以被配置为若干个二级缓存模式之一。以下缓存模式设置由 Java Persistence API 定义。

 

缓存模式 | 设置说明
---- | ----
ALL | 所有实体数据存储在该持久化单元的二级缓存中。
NONE | 在持久化单元中不缓存任何数据。持久化提供程序不能缓存任何数据。
ENABLE_SELECTIVE | 对已使用`@Cacheable`注解显式设置的实体启用缓存。
DISABLE_SELECTIVE | 为所有实体启用缓存，但使用`@Cacheable(false)`注解显式设置的实体除外。
UNSPECIFIED | 持久化单元的缓存行为是未定义的。将使用持久化提供程序的默认缓存行为。


在应用程序中使用二级缓存的一个后果是，数据库表中底层数据可能已更改，而缓存中的值没有更改，这种情况称为过期读取（stale read）。要避免过期读取，请使用以下任何策略：

* 将二级缓存更改为缓存模式设置之一
* 控制哪些实体可以缓存
* 更改缓存的检索或存储模式
* 哪些策略最好地避免过时读取取决于应用程序。

### 控制实体是否可以被缓存

javax.persistence.Cacheable 注解用于指定当使用 ENABLE_SELECTIVE 或 DISABLE_SELECTIVE 缓存模式时，实体类和任何子类可以进行缓存。子类可以通过添加`@Cacheable`注解并更改值来覆盖`@Cacheable`设置。

要指定可以缓存实体，请在类级别添加`@Cacheable`注解：

```
@Cacheable
@Entity
public class Person { ... }
```

默认情况下，`@Cacheable`注解为 true。以下示例是等效的：

```
@Cacheable(true)
@Entity
public class Person{ ... }
```

要指定不必缓存实体，请添加`@Cacheable` 注解并将其设置为 false：


```
@Cacheable(false)
@Entity
public class OrderStatus { ... }
```

当设置 ENABLE_SELECTIVE 缓存模式时，持久化提供程序将缓存具有`@Cacheable(true)`注解的实体以及该实体的任何未重写的子类。持久化提供程序不会缓存具有`@Cacheable(false)`或没有`@Cacheable`注解的实体。也就是说，ENABLE_SELECTIVE 模式将仅缓存已使用`@Cacheable`注解显式标记为缓存的实体。

当设置了 DISABLE_SELECTIVE 缓存模式时，持久化提供程序将缓存没有`@Cacheable(false)`注解的任何实体。没有`@Cacheable`注解的实体和具有`@Cacheable(true)`注解的实体将被缓存。也就是说，DISABLE_SELECTIVE 模式将缓存尚未明确阻止的所有实体被缓存。

如果缓存模式设置为 UNDEFINED 或未设置，则使用`@Cacheable`注解的实体的行为未定义。如果缓存模式设置为 ALL 或 NONE，则持久化提供程序将忽略`@Cacheable`注解的值。


## 指定缓存模式设置以提高性能


要调整持久化单元的缓存模式设置，请指定一种高速缓存模式作为 persistence.xml 部署描述符中的 shared-cache-mode 元素的值：

```
<persistence-unit name="examplePU" transaction-type="JTA">
    <provider>org.eclipse.persistence.jpa.PersistenceProvider</provider>
    <jta-data-source>java:comp/DefaultDataSource</jta-data-source>
    <shared-cache-mode>DISABLE_SELECTIVE</shared-cache-mode>
</persistence-unit>
```

>注意：
由于 Java Persistence API 规范不需要对二级缓存的支持，因此在使用不实现二级缓存的持久化提供程序时，在 persistence.xml 中设置二级缓存模式不会产生任何影响。

或者，您可以通过将 javax.persistence.sharedCache.mode 属性设置为共享缓存模式设置之一来指定共享缓存模式：

```
EntityManagerFactory emf = 
    Persistence.createEntityManagerFactory(
        "myExamplePU", new Properties().add(
            "javax.persistence.sharedCache.mode", "ENABLE_SELECTIVE"));
```

### 设置缓存检索和存储模式

如果通过设置共享缓存模式为持久化单元启用了二级高速缓存，那么可以通过设置 javax.persistence.cache.retrieveMode 和 javax.persistence.cache.storeMode 来进一步修改二级缓存的行为属性。您可以通过将属性名称和值传递到 EntityManager.setProperty 方法，在持久化上下文级别设置这些属性，或者您可以在每个 EntityManager 操作（EntityManager.find或EntityManager.refresh）或每个查询级别。

#### 缓存检索模式

缓存检索模式由 javax.persistence.retrieveMode 属性设置，控制如何从缓存中读取数据，以调用 EntityManager.find 方法和查询。

您可以将 retrieveMode 属性设置为由 javax.persistence.CacheRetrieveMode 枚举类型 USE（缺省值）或 BYPASS 定义的常量之一。

当属性设置为 USE 时，将从二级缓存检索数据（如果可用）。如果数据不在缓存中，持久化提供程序将从数据库读取数据。

当属性设置为 BYPASS 时，将绕过二级缓存，并调用数据库以检索数据。

#### 缓存存储模式

缓存存储模式由 javax.persistence.storeMode 属性设置，控制数据如何存储在高速缓存中。

storeMode 属性可以设置为 javax.persistence.CacheStoreMode 枚举类型定义的常量之一：USE（缺省值），BYPASS 或 REFRESH。

当属性设置为 USE 时，当从数据库读取或提交数据时，将创建或更新缓存数据。如果数据已经在缓存中，则将数据从数据库读取时，将存储模式设置为 USE 不会强制刷新。

当属性设置为  BYPASS 时，不会在缓存中插入或更新从数据库读取或提交到数据库的数据。也就是说，缓存不变。

当属性设置为 REFRESH 时，在从数据库读取或提交数据时创建或更新缓存数据，并且在数据库读取时强制对缓存中的数据进行刷新。

#### 设置缓存检索或存储模式

要为持久化上下文设置缓存检索或存储模式，请调用具有属性名称和值对的 EntityManager.setProperty 方法：

```
EntityManager em = ...;
em.setProperty("javax.persistence.cache.storeMode", "BYPASS");
```

要在调用 EntityManager.find 或 EntityManager.refresh 方法时设置缓存检索或存储模式，请首先创建  Map<String, Object> 实例并添加名称/值对，如下所示：

```
EntityManager em = ...;
Map<String, Object> props = new HashMap<String, Object>();
props.put("javax.persistence.cache.retrieveMode", "BYPASS");
String personPK = ...;
Person person = em.find(Person.class, personPK, props);
```

>注意：
当调用 EntityManager.refresh 方法时，将忽略缓存检索模式，因为刷新调用总是导致从数据库读取数据，而不是缓存。

要在使用查询时设置检索或存储模式，请根据查询类型调用 Query.setHint 或 TypedQuery.setHint 方法：

```
EntityManager em = ...;
CriteriaQuery<Person> cq = ...;
TypedQuery<Person> q = em.createQuery(cq);
q.setHint("javax.persistence.cache.storeMode", "REFRESH");
...
```


在查询中或在调用 EntityManager.find 或 EntityManager.refresh 方法时设置存储或检索模式将覆盖实体管理器的设置。

### 以编程方式控制二级缓存

javax.persistence.Cache 接口定义了用于以编程方式与第二级缓存交互的方法。 Cache 接口定义了执行以下操作的方法：

* 检查特定实体是否具有缓存数据
* 从缓存中删除特定实体
* 删除所有实例（和子类的实例）
* 清除所有实体数据的缓存

>注意：
如果第二级缓存被禁用，对 Cache 接口的方法的调用没有效果，除了 contains，它将总是返回 false。


#### 检查实体的数据是否已缓存

调用 Cache.contains 方法来确定给定实体是否当前在二级缓存中。如果实体的数据被缓存，contains 方法返回 true，如果数据不在缓存中，则返回 false：

```
EntityManager em = ...;
Cache cache = em.getEntityManagerFactory().getCache();
String personPK = ...;
if (cache.contains(Person.class, personPK)) {
  // the data is cached
} else {
  // the data is NOT cached
}
```

#### 从缓存中删除实体

调用 Cache.evict 方法之一来从二级缓存中删除特定实体或给定类型的所有实体。要从缓存中删除特定实体，请调用 evict 方法并传入实体类和实体的主键：

```
EntityManager em = ...;
Cache cache = em.getEntityManagerFactory().getCache();
String personPK = ...;
cache.evict(Person.class, personPK);
```

要删除特定实体类（包括子类）的所有实例，请调用 evict 方法并指定实体类：


```
EntityManager em = ...;
Cache cache = em.getEntityManagerFactory().getCache();
cache.evict(Person.class);
```

Person 实体类的所有实例将从缓存中删除。如果 Person 实体有一个子类 Student，对上述方法的调用也将从缓存中删除所有 Student 实例。

#### 从缓存中删除所有数据

调用 Cache.evictAll 方法以完全清除二级缓存：

```
EntityManager em = ...;
Cache cache = em.getEntityManagerFactory().getCache();
cache.evictAll();
```
