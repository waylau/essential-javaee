# 基于字符串的 Criteria 查询

本章介绍如何创建弱类型的基于字符串的 Criteria API 查询。

主要讨论以下主题：

* 基于字符串的 Criteria API 查询的概述
* 创建基于字符串的查询
* 执行基于字符串的查询


## 概述

基于字符串的Criteria API查询（简称“基于字符串的查询”）是Java编程语言查询，它使用字符串而不是强类型元模型对象，以遍历数据层次结构时指定实体属性。 基于字符串的查询构造类似于元模型查询，可以是静态或动态的，并且可以表示与强类型化的元模型查询相同类型的查询和操作。

强类型的元模型查询是构造Criteria API 查询的首选方法。

基于字符串的查询对于元模型查询的主要优点是能够在开发时构造 Criteria 查询，而无需生成静态元模型类或以其他方式访问动态生成的元模型类。

基于字符串的查询的主要缺点是缺乏类型安全性。这个问题可能导致运行时错误，由于类型不匹配，如果你使用强类型的元模型查询，将在开发时捕获。


## 创建基于字符串的查询


要创建基于字符串的查询，请直接将实体类的属性名称指定为字符串，而不是指定元模型类的属性。例如，此查询查找名称属性的值为 Fido 的所有 Pet 实体：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(cb.equal(pet.get("name"), "Fido"));
```

属性的名称被指定为字符串。此查询等同于以下元模型查询：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Metamodel m = em.getMetamodel();
EntityType<Pet> Pet_ = m.entity(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(cb.equal(pet.get(Pet_.name), "Fido"));
```


注意：
基于字符串的查询中的类型不匹配错误将不会出现，直到代码在运行时执行，不像上面的元模型查询，其中类型不匹配将在编译时捕获。

连接操作以相同的方式指定：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
Join<Owner, Address> address = pet.join("owners").join("addresses");
```

在元模型查询中使用的所有条件表达式、方法表达式、路径导航方法和结果限制方法也可以在基于字符串的查询中使用。在每种情况下，使用字符串指定属性。例如，以下是使用 in 表达式的基于字符串的查询：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(pet.get("color").in("brown", "black"));
```

这是一个基于字符串的查询，按日期按降序排列结果：


```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.select(pet);
cq.orderBy(cb.desc(pet.get("birthday")));
```

## 执行基于字符串的查询



基于字符串的查询类似于强类型的 Criteria 查询执行。 首先，通过将条件查询对象传递到 EntityManager.createQuery 方法来创建 javax.persistence.TypedQuery 对象，然后在查询对象上调用 getSingleResult或getResultList 来执行查询：

```
CriteriaQuery<Pet> cq = cb.createQuery(Pet.class);
Root<Pet> pet = cq.from(Pet.class);
cq.where(cb.equal(pet.get("name"), "Fido"));
TypedQuery<Pet> q = em.createQuery(cq);
List<Pet> results = q.getResultList();
```
