# 使用 Criteria API

Criteria API 用于通过创建查询定义对象来定义实体及其持久状态的查询。 条件查询使用Java编程语言API编写，具有类型安全的、可移植的特点。 此类查询的工作机制与基础数据存储无关。

本章节主要讨论以下主题：

* Criteria API 和 Metamodel API 概述
* 使用 Metamodel API 来对实体类建模
* 使用 Criteria API 和 Metamodel API 创建基本类型安全的查询


## Criteria API 和 Metamodel API 概述

与JPQL类似，Criteria API 基于持久化实体、它们的关系以及嵌入对象的抽象模式。 Criteria API 对此抽象模式进行操作，以允许开发人员通过调用 Java Persistence API 实体操作来查找、修改和删除持久性实体。 Metamodel API 与 Criteria API 协同工作，用于为 Criteria 查询建立持久化实体类的模型。

Criteria API 和 JPQL 是密切相关的，并且被设计为允许在它们的查询中进行类似的操作。 熟悉 JPQL 语法的开发人员将在 Criteria API 中找到等效的对象级操作。

以下简单的 Criteria 查询返回数据源中的 Pet 实体的所有实例：

```
EntityManager em = ...;
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.select(pet);
TypedQuery<Pet> q = em.createQuery(cq);
List<Pet> allPets = q.getResultList();
```

等价于 PQL 的查询是：

```
SELECT p
FROM Pet p
```


此查询演示了创建条件查询的基本步骤。

* 使用 EntityManager 实例创建 CriteriaBuilder 对象。
* 通过创建 CriteriaQuery 接口的实例来创建查询对象。 此查询对象的属性将使用查询的详细信息进行修改。
* 通过调用 CriteriaQuery 对象上的 from 方法设置查询根。
* 通过调用 CriteriaQuery 对象的 select 方法指定查询结果的类型。
* 通过创建 TypedQuery<T> 实例来准备执行查询，指定查询结果的类型。
* 通过调用 TypedQuery<T> 对象上的 getResultList 方法执行查询。 因为此查询返回实体的集合，所以结果存储在 List 中。

 
要创建 CriteriaBuilder 实例，请调用 EntityManager 实例上的 getCriteriaBuilder 方法：

```
CriteriaBuilder cb = em.getCriteriaBuilder();
```

使用 CriteriaBuilder 实例来创建查询对象：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
```

查询将返回 Pet 实体的实例。 要创建类型安全查询，请在创建 CriteriaQuery 对象时指定查询的类型。

调用查询对象的 from 方法来设置查询的 FROM 子句并指定查询的根：

```
Root<Pet> pet = cq.from(Pet.class);
```

调用查询对象的 select 方法，传入查询根，设置查询的 SELECT 子句：

```
cq.select(pet);
```

现在，使用查询对象创建一个可以针对数据源执行的  TypedQuery<T> 对象。 捕获对查询对象的修改以创建即可执行的查询：

```
TypedQuery<Pet> q = em.createQuery(cq);
```
通过调用其 getResultList 方法来执行此类型的查询对象，因为此查询将返回多个实体实例。 以下语句将结果存储在  List<Pet> 集合值对象中：


```
List<Pet> allPets = q.getResultList();
```
## 使用 Metamodel API 来对实体类建模


使用 Metamodel API 在特定持久化单元中创建受管实体的元模型。 对于特定包中的每个实体类，将创建一个具有尾随下划线和对应于实体类的持久化字段或属性的属性的元模型类。

以下实体 类com.example.Pet 具有四个持久化字段：id、name、color 和 owners ：

```
package com.example;
...
@Entity
public class Pet {
    @Id
    protected Long id;
    protected String name;
    protected String color;
    @ManyToOne
    protected Set<Person> owners;
    ...
}
```


对应的 Metamodel 类，如下：

```
package com.example;
...
@Static Metamodel(Pet.class)
public class Pet_ {

    public static volatile SingularAttribute<Pet, Long> id;
    public static volatile SingularAttribute<Pet, String> name;
    public static volatile SingularAttribute<Pet, String> color;
    public static volatile SetAttribute<Pet, Person> owners;
}
```


Criteria 查询使用元模型类及其属性来引用被管实体类及其持久状态和关系。


### 使用元模型类

对应于实体类的元模型（metamodel）类具有以下类型：

```
javax.persistence.metamodel.EntityType<T>
```

注解处理器通常在开发时或在运行时生成元模型类。 使用 Criteria 查询的应用程序的开发人员可以执行以下任一操作：

* 通过使用持久化提供者的注解处理器来生成静态元模型类
* 通过执行以下操作之一获取元模型类：
	* 在查询根对象上调用 getModel 方法
	* 获取元模型接口的实例，然后将实体类型传递给实例的实体方法

以下代码片段显示了如何通过调用 Root<T>.getModel 来获取 Pet 实体的元模型类：

```
EntityManager em = ...;
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
EntityType<Pet> Pet_ = pet.getModel();
```


以下代码片段显示了如何获取 Pet 实体的元模型类，首先使用 EntityManager.getMetamodel 获取元模型实例，然后在元模型实例上调用实体：

```
EntityManager em = ...;
Metamodel m = em.getMetamodel();
EntityType<Pet> Pet_ = m.entity(Pet.class);
```

注意：
最常见的用例是在开发时生成类型安全的静态元模型类。 动态获取元模型类，通过调用Root<T>.getModel 或 EntityManager.getMetamodel然后实体方法，不允许类型安全，并且不允许应用程序在元模型类上调用持久字段或属性名称。

## 使用 Criteria API 和 Metamodel API 创建基本类型安全的查询


条件查询的基本语义由 SELECT 子句，FROM 子句和可选的 WHERE 子句组成，类似于 JPQL 查询。 条件查询使用Java 编程语言对象设置这些子句，因此可以以类型安全的方式创建查询。

### 创建条件查询

javax.persistence.criteria.CriteriaBuilder 接口用于构造

* Criteria 查询
* 选择
* 表达式
* 谓词
* 排序

要获取 CriteriaBuilder 接口的实例，请在 EntityManager 或 EntityManagerFactory 实例上调用 getCriteriaBuilder 方法。

以下代码显示了如何使用 EntityManager.getCriteriaBuilder 方法获取 CriteriaBuilder 实例：

```
EntityManager em = ...;
CriteriaBuilder cb = em.getCriteriaBuilder();
```


Criteria 查询通过获取以下接口的实例来构造：

```
javax.persistence.criteria.CriteriaQuery
```

CriteriaQuery 对象定义将导航到一个或多个实体的特定查询。 通过调用 CriteriaBuilder.createQuery 方法之一获 取CriteriaQuery 实例。 要创建类型安全查询，请按如下所示调用 CriteriaBuilder.createQuery 方法：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
```

CriteriaQuery 对象的类型应设置为查询的预期结果类型。 在上面的代码中，对象的类型设置为 CriteriaQuery<Pet>，用于查找 Pet 实体的实例。

以下代码段为返回 String 的查询创建 CriteriaQuery 对象：

```
CriteriaQuery<String> cq = cb.createQuery(String.class);
```

### 查询根

对于特定的 CriteriaQuery 对象，所有导航起源的查询的根实体称为查询根（query root）。它类似于 JPQL 查询中的 FROM 子句。

通过调用 CriteriaQuery 实例上的 from 方法创建查询根。 from 方法的参数是实体类或实体的 EntityType<T> 实例。

以下代码将查询根设置为 Pet 实体：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
```

以下代码使用 EntityType<T> 实例将查询根设置为 Pet 类：

```
EntityManager em = ...;
Metamodel m = em.getMetamodel();
EntityType<Pet> Pet_ = m.entity(Pet.class);
Root<Pet> pet = cq.from(Pet_);
```


条件查询可能有多个查询根。这通常发生在查询从几个实体导航时。

以下代码具有两个 Root 实例：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet1 = cq.from(Pet.class);
Root<Pet> pet2 = cq.from(Pet.class);
```

###　使用连接查询关系

对于导航到相关实体类的查询，查询必须通过调用查询根对象或另一个连接对象上的　From.join　方法之一来定义到相关实体的连接。连接方法类似于　JPQL　中的　JOIN　关键字。

连接的目标使用类型　EntityType<T>　的元模型类来指定连接实体的持久化字段或属性。

连接方法返回 Join<X，Y> 类型的对象，其中X是源实体，Y是连接的目标。在以下代码片段中，Pet 是源实体，Owner 是目标，而 Pet_ 是静态生成的元模型类：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);

Root<Pet> pet = cq.from(Pet.class);
Join<Pet, Owner> owner = pet.join(Pet_.owners);
```


您可以将连接链接在一起以导航到目标实体的相关实体，而无需为每个连接创建 Join<X，Y> 实例：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);

Root<Pet> pet = cq.from(Pet.class);
Join<Owner, Address> address = pet.join(Pet_.owners).join(Owner_.addresses);
```

### 条件查询中的路径导航

在 Criteria 查询的 SELECT 和 WHERE 子句中使用的路径对象可以是查询根实体，连接实体或其他 Path 对象。 使用 Path.get 方法导航到查询实体的属性。

get 方法的参数是实体的 Metamodel 类的相应属性。 属性可以是单值属性，由 Metamodel 类中的`@SingularAttribute`指定，也可以是由`@CollectionAttribute`、`@SetAttribute`、`@ListAttribute`或`@MapAttribute`之一指定的集合值属性。

以下查询返回数据存储中所有宠物的名称。 在查询根 pet 上调用 get 方法，使用 Pet 实体的元模型类 Pet_ 的 name 属性作为参数：


```
CriteriaQuery<String> cq = cb.createQuery(String.class);

Root<Pet> pet = cq.from(Pet.class);
cq.select(pet.get(Pet_.name));
```

### 限制条件查询结果

通过调用 CriteriaQuery.where 方法设置的条件可以限制对 CriteriaQuery 对象的查询结果。 调用 where 方法类似于在 JPQL 查询中设置 WHERE 子句。

where 方法计算 Expression 接口的实例，以根据表达式的条件限制结果。 要创建 Expression 实例，请使用 Expression 和 CriteriaBuilder 接口中定义的方法。

#### 表达式接口方法

Expression 对象用于查询的 SELECT、WHERE 或 HAVING 子句。 下表显示了可以与 Expression 对象一起使用的条件方法。



方法	| 描述
---- | ----
isNull | 测试表达式是否是  null
isNotNull | 测试表达式是否是非 null
in | 测试表达式是否在列表里面



以下查询使用 Expression.isNull 方法查找 color 属性为 null 的所有宠物：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(pet.get(Pet_.color).isNull());
```

以下查询使用 Expression.in 方法查找所有颜色是 brown 和 black 宠物：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(pet.get(Pet_.color).in("brown", "black"));
```

in 方法还可以检查属性是否是集合的成员。


#### CriteriaBuilder 接口中的表达式方法

CriteriaBuilder 接口定义了用于创建表达式的其他方法。 这些方法对应于 JPQL 的算术、字符串、日期、时间和案例运算符和函数。下表显示了可以与 CriteriaBuilder 对象一起使用的条件方法。


条件方法 | 描述
---- |----
equal | 测试两个表达式是否相等
notEqual | 测试两个表达式是否不相等
gt | 测试第一个数字表达式是否大于第二个数字表达式
ge | 测试第一个数字表达式是否大于第二个数字表达式
lt | 测试第一个数字表达式是否小于第二个数字表达式
le | 测试第一个数字表达式是否小于或等于第二个数字表达式
between | 测试第一个表达式是否在第二个和第三个表达式之间
like | 测试表达式是否匹配给定模式


 
以下代码使用 CriteriaBuilder.equal 方法：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(cb.equal(pet.get(Pet_.name), "Fido"));
```

以下代码使用 CriteriaBuilder.gt 方法：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
Date someDate = new Date(...);
cq.where(cb.gt(pet.get(Pet_.birthday), date));
```

以下代码使用 CriteriaBuilder.between 方法：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
Date firstDate = new Date(...);
Date secondDate = new Date(...);
cq.where(cb.between(pet.get(Pet_.birthday), firstDate, secondDate));
```

以下代码使用 CriteriaBuilder.like 方法：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(cb.like(pet.get(Pet_.name), "*do"));
```

要指定多个条件谓词，请使用 CriteriaBuilder 接口的复合谓词方法，如下表所示。


方法 | 描述
---- | ----
and | 两个布尔表达式的逻辑与
or | 两个布尔表达式的逻辑或
not | 给定布尔表达式的逻辑非


以下代码显示了查询中复合谓词的使用：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(cb.equal(pet.get(Pet_.name), "Fido")
        .and(cb.equal(pet.get(Pet_.color), "brown")));
```


### 管理条件查询结果

对于返回多个结果的查询，组织这些结果通常很有帮助。 CriteriaQuery 接口定义了以下排序和分组方法：

* orderBy 方法根据实体的属性来排序查询结果
* groupBy 方法根据实体的属性将查询的结果组合在一起，并且 having 方法根据条件限制这些组

#### 排序结果

要对查询的结果排序，请调用 CriteriaQuery.orderBy 方法，传入 Order 对象。 要创建 Order 对象，请调用 CriteriaBuilder.asc 或 CriteriaBuilder.desc 方法。 asc 方法用于通过递增的表达式参数的值来对结果排序。 desc 方法用于通过递减表达式参数的值来对结果排序。 以下查询显示了 desc 方法的使用：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.select(pet);
cq.orderBy(cb.desc(pet.get(Pet_.birthday)));
```


在此查询中，结果将按宠物的生日从高到低排序。 也就是说，12月出生的宠物将出现在5月出生的宠物之前。

以下查询显示了 asc 方法的用法：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
Join<Owner, Address> address = pet.join(Pet_.owners).join(Owner_.address);
cq.select(pet);
cq.orderBy(cb.asc(address.get(Address_.postalCode)));
```

在此查询中，结果将按宠物所有者的邮政编码从低到高排序。 也就是说，拥有者居住在10001邮政编码的宠物将出现在其所有者居住在91000邮政编码的宠物之前。

如果多个 Order 对象被传递给 orderBy，则优先级由它们在 orderBy 的参数列表中出现的顺序确定。 第一个 Order 对象具有优先级。

以下代码按多个条件排列结果：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
Join<Pet, Owner> owner = pet.join(Pet_.owners);
cq.select(pet);
cq.orderBy(cb.asc(owner.get(Owner_.lastName)), owner.get(Owner_.firstName)));
```


此查询的结果将按宠物所有者的姓氏，然后是名字按字母顺序排序。

#### 分组结果

CriteriaQuery.groupBy 方法将查询结果分为组。要设置这些组，请将表达式传递给 groupBy：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.groupBy(pet.get(Pet_.color));
```


此查询返回所有宠物实体，并按宠物的颜色对结果进行分组。

使用 CriteriaQuery.having 方法与 groupBy 结合过滤组。以条件表达式作为参数的 having 方法根据条件表达式来限制查询结果：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.groupBy(pet.get(Pet_.color));
cq.having(cb.in(pet.get(Pet_.color)).value("brown").value("blonde"));
```

在此示例中，查询按颜色对返回的 Pet 实体进行分组，如前面的示例所示。然而，唯一返回的组将是其中 color 属性被设置为 brown 或 blonde 的宠物实体。也就是说，在此查询中不会返回 gray-colored 颜色的宠物。

### 执行查询

要准备执行查询，请使用查询结果的类型创建一个 TypedQuery<T> 对象，将 CriteriaQuery 对象传递给 EntityManager.createQuery。

要执行查询，请对 TypedQuery<T> 对象调用 getSingleResult或getResultList。

#### 单值查询结果

使用TypedQuery<T>.getSingleResult  方法执行返回单个结果的查询：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
...
TypedQuery<Pet> q = em.createQuery(cq);
Pet result = q.getSingleResult();
```

#### 集合值查询结果

使用TypedQuery<T>.getResultList 方法执行返回一组对象的查询：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
...
TypedQuery<Pet> q = em.createQuery(cq);
List<Pet> results = q.getResultList();
```
