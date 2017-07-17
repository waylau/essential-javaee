# 查询语句

Java 持久化查询语言（Java Persistence query language，简称 JPQL）定义实体及其持久状态的查询。 查询语言允许您编写可用的查询语句，而不用管底层数据是如何实现存储的。

查询语言使用实体的抽象持久化模式（包括其关系）作为其数据模型，并基于此数据模型定义运算符和表达式。 查询的范围跨越包装在同一持久单元中的相关实体的抽象模式。 查询语言使用类似SQL的语法来基于实体抽象模式类型和它们之间的关系来选择对象或值。
 
本章节讨论以下主题：

* 查询语言术语
* 使用Java持久化查询语言创建查询
* 简化查询语法
* 示例
* 完整的查询语法
 
## 术语

本章节常用的术语有：

* 抽象模式（Abstract schema）：持久化模式抽象（持久化实体、它们的状态，以及它们的关系）在查询操作在之上。查询语言将通过该持久化模式抽象的查询翻译成在实体被映射到的数据库模式上执行的查询。
* 抽象模式类型（Abstract schema type）：实体的持久化属性在抽象模式中求值的类型。也就是说，实体中的每个持久化字段或属性在抽象模式中具有相同类型的对应状态字段。实体的抽象模式类型从实体类和Java语言注解提供的元数据信息中派生。
* Backus-Naur Form（BNF）：描述高级语言的语法的符号。本章中的语法图以BNF表示法表示。
* 导航（Navigation）：在查询语言表达式中遍历关系。导航操作是一个周期。
* 路径表达式（Path expression）：导航到实体状态或关系字段的表达式。
* 状态字段（State field）：实体的持久字段。
* 关系字段（Relationship field）：实体的持久字段，其类型是相关实体的抽象模式类型。


## 创建查询

EntityManager.createQuery 和 EntityManager.createNamedQuery 方法用于通过使用Java持久化查询语言查询来查询数据存储区。

createQuery 方法用于创建动态查询，这些查询是在应用程序的业务逻辑中直接定义的查询：

```
public List findWithName(String name) {
return em.createQuery(
    "SELECT c FROM Customer c WHERE c.name LIKE :custName")
    .setParameter("custName", name)
    .setMaxResults(10)
    .getResultList();
}
```


createNamedQuery 方法用于创建静态查询（ static query）或通过使用`javax.persistence.NamedQuery`注释在元数据中定义的查询。 `@NamedQuery`的 name 元素指定将与 createNamedQuery 方法一起使用的查询的名称。 `@NamedQuery`的query元素是查询语句：

```
@NamedQuery(
    name="findAllCustomersWithName",
    query="SELECT c FROM Customer c WHERE c.name LIKE :custName"
)
```

这里有一个 createNamedQuery 的例子，它使用`@NamedQuery`：

```
@PersistenceContext
public EntityManager em;
...
customers = em.createNamedQuery("findAllCustomersWithName")
    .setParameter("custName", "Smith")
    .getResultList();
```


### 查询中的命名参数

命名参数（Named Parameter）是以冒号（:）为前缀的查询参数。 查询中的命名参数通过以下方法绑定到参数：

```
javax.persistence.Query.setParameter(String name, Object value)
```

在以下示例中，findWithName 业务方法的 name 参数通过调用 Query.setParameter 绑定到查询中的`:custName`命名参数：

```
public List findWithName(String name) {
    return em.createQuery(
        "SELECT c FROM Customer c WHERE c.name LIKE :custName")
        .setParameter("custName", name)
        .getResultList();
}
```


命名参数区分大小写，可以由动态和静态查询使用。


### 查询中的位置参数

您可以在查询中使用位置参数（Positional Parameter）来代替命名参数。 位置参数前面带有问号（？），后跟查询中参数的数字位置。 方法 Query.setParameter(integer position, Object value) 用于设置参数值。

在以下示例中，将重写 findWithName 业务方法以使用输入参数：

```
public List findWithName(String name) {
    return em.createQuery(
        "SELECT c FROM Customer c WHERE c.name LIKE ?1")
        .setParameter(1, name)
        .getResultList();
}
```

 
输入参数从1开始编号。输入参数区分大小写，并且可以由动态和静态查询使用。

## 简化的查询语法

本节简要介绍查询语言的语法。 

### select 语句

select 查询有六个子句：SELECT、FROM、WHERE、GROUP BY、HAVING 和 ORDER BY。 SELECT 和 FROM子句是必需的，而 WHERE、GROUP BY、HAVING 和 ORDER BY子句是可选的。 下面是 select 查询的高级 BNF 语法：

```
QL_statement ::= select_clause from_clause 
  [where_clause][groupby_clause][having_clause][orderby_clause]
```


BNF 语法定义以下子句。

* SELECT 子句定义查询返回的对象或值的类型。
* FROM 子句通过声明一个或多个标识变量来定义查询的范围，可以在 SELECT 和 WHERE 子句中引用它们。标识变量表示以下元素之一：
	* 实体的抽象模式名称
	* 集合关系的元素
	* 一个单值关系的元素
	* 作为一对多关系的多边的集合的成员
* WHERE 子句是限制由查询检索的对象或值的条件表达式。虽然子句是可选的，但大多数查询都有一个 WHERE 子句。
* GROUP BY 子句根据一组属性对查询结果进行分组。
* HAVING 子句与 GROUP BY 子句一起使用，以根据条件表达式进一步限制查询结果。
* ORDER BY 子句将查询返回的对象或值排序到指定的顺序。


### update 和 delete 语句

update 和 delete 语句对实体集提供批量操作。 这些语句具有以下语法：

```bnf
update_statement :: = update_clause [where_clause] 
delete_statement :: = delete_clause [where_clause]
```

update 和 delete 子句确定要更新或删除的实体的类型。 WHERE 子句可用于限制更新或删除操作的范围。


## 示例

下面查询语句来自 [roster](http://docs.oracle.com/javaee/7/tutorial/persistence-basicexamples002.htm#GIQSQ) 应用的 Player 实体。

### 简单查询


#### 简单的 Select 查询

```
SELECT p
FROM Player p
```

* 检索的数据：所有球员。
* 描述：FROM 子句声明一个名为 p 的标识变量，省略可选的关键字 AS。 如果包括 AS 关键字，则该子句将写如下：

```
FROM Player AS p
```

Player 元素是 Player 实体的抽象模式名称。



#### 消除重复值

```
SELECT DISTINCT p
FROM Player p
WHERE p.position = ?1
```

* 检索的数据：具有由查询的参数指定的 position 的球员。
* 说明：DISTINCT 关键字用来消除重复值。

WHERE 子句通过检查球员的 position（Player 实体的持久化字段） 来限制球员检索。`?1`元素表示查询的输入参数。

#### 使用命名参数

```
SELECT DISTINCT p
FROM Player p
WHERE p.position = :position AND p.name = :name
```

* 检索的数据：具有指定 position 和  name 的球员。
* 说明：position 和  name 元素是 Player 实体的持久化字段。 WHERE 子句将这些字段的值与使用 Query.setNamedParameter 方法设置的查询的命名参数进行比较。 查询语言表示使用冒号（:）后跟标识符的命名输入参数。 第一个输入参数是`:position`，第二个是`:name`。


### 导航到相关实体的查询

在查询语言中，表达式可以遍历或导航到相关实体。 这些表达式是 JPQL 和 SQL 之间的主要区别。 JPQL 通过导航到相关实体来进行查询，而 SQL 是通过连接表来查询。



#### 带关系的简单查询

```
SELECT DISTINCT p
FROM Player p, IN (p.teams) t
```

* 检索的数据：属于某个团队的所有球员。
* 描述：FROM 子句声明两个标识变量：p 和 t。 p 变量表示球员实体，t 变量表示相关的团队实体。 t 的声明引用先前声明的 p 变量。 IN 关键字表示团队是相关实体的集合。 p.teams 表达式从一个 Player 导航到它的相关 Team。 p.teams 表达式中的周期是导航运算符。

您还可以使用 JOIN 语句编写相同的查询：

```
SELECT DISTINCT p
FROM Player p JOIN p.teams t
```

这个查询同时也可以被重写为:

```
SELECT DISTINCT p
FROM Player p
WHERE p.team IS NOT EMPTY
```

#### 导航到单值关系字段

使用 JOIN 子句语句导航到单值关系字段：

```
SELECT t
FROM Team t JOIN t.league l
WHERE l.sport = 'soccer' OR l.sport ='football'
```

在此示例中，查询将返回 soccer 或 football 联赛中的所有球队。


#### 遍历与输入参数的关系

```
SELECT DISTINCT p
FROM Player p, IN (p.teams) AS t
WHERE t.city = :city
```

* 检索的数据：球队属于指定 city 的球员。
* 说明：此查询与上一个示例类似，但添加了输入参数。 FROM 子句中的 AS 关键字是可选的。 在 WHERE 子句中，持久变量 city 之前的周期是一个定界符，而不是导航运算符。 严格地说，表达式可以导航到关系字段（相关实体），但不导航到持久字段。 要访问持久字段，表达式使用句点作为分隔符。

表达式无法超越（或进一步限定）作为集合的关系字段。 在表达式的语法中，集合值字段是终端符号。 因为 teams 字段是一个集合，所以 WHERE 子句不能指定p.teams.city（一个非法表达式）。


#### 遍历多个关系

```
SELECT DISTINCT p
FROM Player p, IN (p.teams) t
WHERE t.league = :league
```

* 检索的数据：属于指定联赛的球员。
* 说明：此查询中的表达式导航两个关系。 p.teams 表达式导航 Player-Team 关系，t.league 表达式导航 Team-League 关系。

在其他示例中，输入参数是 String 对象; 在此示例中，参数是类型为 League 的对象。 此类型匹配 WHERE 子句的比较表达式中的联赛关系字段。



#### 根据相关领域进行导航

```
SELECT DISTINCT p
FROM Player p, IN (p.teams) t
WHERE t.league.sport = :sport
```


* 数据检索：参与指定 sport 的球员。
* 说明：sport 持久字段属于 League 实体。 要获得 sport 字段，查询必须首先从 Player 实体导航到Team（p.teams），然后从 Team 导航到 League 实体（t.league）。 因为它不是集合，所以　league 关系字段可以跟随　sport　持久字段。


### 其他条件表达式的查询

每个 WHERE 子句必须指定一个条件表达式，其中有几种。 在前面的例子中，条件表达式是测试相等的比较表达式。 以下示例演示了一些其他种类的条件表达式。

#### LIKE表达式

```
SELECT p
FROM Player p
WHERE p.name LIKE 'Mich%'
```

* 检索的数据：名称以“Mich”开头的所有球员。
* 说明：LIKE 表达式使用通配符来搜索与通配符模式匹配的字符串。 在这种情况下，查询使用 LIKE 表达式和 ％ 通配符找到名称以字符串“Mich”开头的所有球员。 例如，“Michael”和“Michelle”都匹配通配符模式。
 

#### IS NULL 表达式


```
SELECT t
FROM Team t
WHERE t.league IS NULL
```

* 检索的数据：所有不与 league 相关联的团队。
* 说明：IS NULL 表达式可用于检查是否已在两个实体之间设置关系。 在这种情况下，查询检查团队是否与任何 league 相关联，并返回没有 league 的团队。
 
#### IS EMPTY 表达式

```
SELECT p
FROM Player p
WHERE p.teams IS EMPTY
```

* 检索的数据：所有不属于某个团队的球员。
* 说明：Player 实体的团队关系字段是一个集合。如果球员不属于某个团队，则团队集合为空，条件表达式为 TRUE。
 

#### BETWEEN 表达式

```
SELECT DISTINCT p
FROM Player p
WHERE p.salary BETWEEN :lowerSalary AND :higherSalary
```

* 检索的数据：工资在指定工资范围内的球员。
* 说明：此 BETWEEN 表达式具有三个算术表达式：持久字段（`p.salary`）和两个输入参数（`:lowerSalary`和`:higherSalary`）。以下表达式等价于 BETWEEN 表达式：
 
```
p.salary >= :lowerSalary AND p.salary <= :higherSalary
```

####  比较运算符

```
SELECT DISTINCT p1
FROM Player p1, Player p2
WHERE p1.salary > p2.salary AND p2.name = :name
```

* 检索的数据：薪水高于具有指定名称的球员的薪水的所有球员。
* 说明：FROM 子句声明同一类型（Player）的两个标识变量（p1和p2）。需要两个标识变量，因为WHERE子句比较一个球员的工资和其他球员的工资。

### 批量更新和删除

以下示例显示如何在查询中使用 UPDATE 和 DELETE 表达式。 UPDATE 和 DELETE 根据 WHERE 子句中设置的条件或条件对多个实体进行操作。 UPDATE 和 DELETE 查询中的 WHERE 子句遵循与 SELECT 查询相同的规则。

#### 更新查询

```
UPDATE Player p
SET p.status = 'inactive'
WHERE p.lastPlayed < :inactiveThresholdDate
```

* 说明：如果球员的最后一个游戏比在 inactiveThresholdDate 中指定的日期早，此查询会将一组球员的状态设置为无效。


#### 删除查询

```
DELETE
FROM Player p
WHERE p.status = 'inactive'
AND p.teams IS EMPTY
```

* 说明：此查询将删除所有不在队伍中的非活动球员。


## 完整的查询语法


完整的查询语言语法，可以直接参考规范

* Java Persistence API 2.0：<http://jcp.org/en/jsr/detail?id=317>
* Java Persistence API 2.1：<http://jcp.org/en/jsr/detail?id=338>