# JPA 简介

Java Persistence API 为 Java 开发人员提供了一种用于在Java应用程序中管理关系数据的对象/关系映射工具。 Java 持久性包括四个方面：

* Java Persistence API
* 查询语言
* Java Persistence Criteria API
* 对象/关系映射元数据



## 实体（Entity）

**实体**是轻量级的持久化域对象。 通常，实体表示关系数据库中的表，并且每个实体实例对应于该表中的行。 实体的主要编程工件是实体类，尽管实体可以使用辅助类。

实体的持久状态通过持久化字段或持久化属性来表示。 这些字段或属性使用对象/关系映射注解将实体和实体关系映射到基础数据存储中的关系数据。

### 实体类的要求

一个实体类，需满足以下条件：

* 类必须用`javax.persistence.Entity`注解。
* 类必须有一个public或protected的无参数的构造函数。 类可以具有其他构造函数。
* 类不能声明为final。 没有方法或持久化实例变量必须声明为 final。
* 如果实体实例被当作值以分离对象方式进行传递（例如通过会话bean的远程业务接口），则该类必须实现`Serializable`接口。
* 实体可以扩展实体类或者是非实体类，并且非实体类可以扩展实体类。
* 持久化实例变量必须声明为 private、protected 或 package-private，并且只能通过实体类的方法直接访问。 客户端必须通过访问器或业务方法访问实体的状态。

### 实体类中的持久化字段和属性

可以通过实体的实例变量或属性访问实体的持久状态。 字段或属性必须是以下Java语言类型：

* Java 原语类型
* java.lang.String
* 其他可序列化类型，包括：
	* Java原始类型的包装器
	* java.math.BigInteger
	* java.math.BigDecimal
	* java.util.Date
	* java.util.Calendar
	* java.sql.Date
	* java.sql.Time
	* java.sql.TimeStamp
	* 用户定义的可序列化类型
	* byte[]
	* Byte[]
	* char[]
	* Character[]
* 枚举类型
* 其他实体和/或实体集合
* 可嵌入类

实体可以使用持久化字段、持久化属性或两者的组合。 如果映射注解应用于实体的实例变量，则实体使用持久化字段。 如果映射注解应用于实体的JavaBeans风格属性的getter方法，则实体使用持久化属性。

#### 持久化字段

如果实体类使用持久化字段，则 Persistence 运行时直接访问实体类实例变量。所有未注解的字段`javax.persistence.Transient`或未标记为Java `transient`将被持久化到数据存储。对象/关系映射注解必须应用于实例变量。

#### 持久化属性

如果实体使用持久性属性，实体必须遵循JavaBeans组件的方法约定。 JavaBeans风格的属性使用getter和setter方法，它们通常以实体类的实例变量名称命名。对于实体的Type类型的每个持久化属性，都有一个getter方法`getProperty`和setter方法`setProperty`。如果属性是布尔值，您可以使用`isProperty`而不是`getProperty。`例如，Customer实体使用持久化属性并具有称为`firstName`的专用实例变量，则该类定义用于检索和设置firstName实例变量的状态的`getFirstName`和`setFirstName`方法。

单值持久化属性的方法签名如下：

```java
Type getProperty()
void setProperty(Type type)
```

持久化属性的对象/关系映射注解必须应用于getter方法。映射注解不能应用于注解为`@Transient`或标记为`transient`的字段或属性。

#### 在实体字段和属性中使用集合

集合作为持久化字段和属性必须使用受Java集合支持的接口，而不管实体是否使用持久化字段或属性。可以使用以下集合接口：

* java.util.Collection
* java.util.Set
* java.util.List
* java.util.Map

如果实体类使用持久化字段，则上述方法签名中的类型必须是这些集合类型之一。也可以使用这些集合类型的泛型。例如，如果它具有包含一组电话号码的持久化属性，那么 Customer 实体将具有以下方法：

```java
Set<PhoneNumber> getPhoneNumbers() { ... }
void setPhoneNumbers(Set<PhoneNumber>) { ... }
```

如果实体的字段或属性由基本类型或可嵌入类的集合组成，请在字段或属性上使用`javax.persistence.ElementCollection`注解。

`@ElementCollection`的两个属性是`targetClass`和`fetch`。 `targetClass`属性指定基本类或可嵌入类的类名，如果字段或属性是使用Java编程语言泛型定义的，则它是可选的。可选的`fetch`属性用于分别使用`LAZY`或`EAGER`的`javax.persistence.FetchType`常量来指定是否应“懒加载”或“急加载”地检索集合。默认情况下，系统会用`LAZY`。

以下实体Person具有持久化字段，它是抓取 String 类的集合的时候使用了急加载。 targetClass 元素不是必需的，因为它使用泛型定义字段：

```java
@Entity
public class Person {
    ...
    @ElementCollection(fetch=EAGER)
    protected Set<String> nickname = new HashSet();
    ...
}
```

实体元素和关系的集合可以由`java.util.Map`表示。 Map由键和值组成。

使用 Map 元素或关系时，以下规则适用:

* Map 键或值可以是基本Java编程语言类型、可嵌入类或实体。
* 当 Map 值是可嵌入类或基本类型时，使用`@ElementCollection`注解。
* 当 Map 值是实体时，使用`@OneToMany`或`@ManyToMany`注解。
* 仅在双向关系的一侧使用 Map 类型。


如果 Map 的键类型是 Java 编程语言基本类型，请使用注解`javax.persistence.MapKeyColumn`来设置键的列映射。默认情况下，`@MapKeyColumn`的 name 属性的格式为`RELATIONSHIP-FIELD/PROPERTY-NAME_KEY`。例如，如果引用关系字段名称为 image，则默认名称属性为`IMAGE_KEY`。

如果 Map 的键类型是实体，请使用`javax.persistence.MapKeyJoinColumn`注解。如果需要多个列来设置映射，请使用注解`javax.persistence.MapKeyJoinColumns`来包含多个`@MapKeyJoinColumn`注解。如果不存在`@MapKeyJoinColumn`，则映射列名称默认设置为`RELATIONSHIP-FIELD/PROPERTY-NAME_KEY`。例如，如果关系字段名称为employee，那么缺省名称属性为`EMPLOYEE_KEY`。

如果 Java 编程语言通用类型未在关系字段或属性中使用，则必须使用`javax.persistence.MapKeyClass`注解显式设置键类。

如果 Map 键是主键，或者实体的持久字段或属性是 Map 值，请使用`javax.persistence.MapKey`注解。 `@MapKeyClass`和`@MapKey`注解不能在同一个字段或属性上使用。

如果 Map 值是Java编程语言基本类型或可嵌入类，它将被映射为底层数据库中的集合表。如果不使用通用类型，则`@ElementCollection`注解的`targetClass`属性必须设置为 Map 值的类型。

如果 Map 值是一个实体，并且是多对多或一对多单向关系的一部分，则它将被映射为底层数据库中的连接表。使用映射的单向一对多关系也可以使用`@JoinColumn`注解映射。

如果实体是一对多/多对一双向关系的一部分，则它将被映射到表示 Map 的值的实体的表中。如果不使用通用类型，则`@OneToMany`和`@ManyToMany`注解的`targetEntity`属性必须设置为 Map 值的类型。

#### 验证持久化字段和属性

 Java API for JavaBeans Validation (Bean Validation)提供了一种验证应用程序数据的机制。Bean Validation 集成到了 Java EE 容器中，允许在企业应用程序的任何层中使用相同的验证逻辑。

Bean Validation 约束可以应用于持久化实体类、可嵌入类和映射的超类。默认情况下，Persistence  提供程序将在PrePersist、PreUpdate 和 PreRemove 生命周期事件之后立即自动对具有持久字段或以Bean Validation 约束注解的属性的实体执行验证。

Bean Validation 约束是应用于Java 编程语言类的字段或属性的注解。Bean Validation 提供了一组约束以及用于定义自定义约束的API。自定义约束可以是默认约束的特定组合，或不使用默认约束的新约束。每个约束与至少一个验证器类相关联，验证器类验证约束字段或属性的值。自定义约束开发人员还必须为约束提供一个验证器类。

Bean Validation 约束应用于持久化类的持久化字段或属性。当添加Bean Validation 约束时，使用与持久类相同的访问策略。也就是说，如果持久化类使用字段访问，则在类的字段上应用Bean Validation 约束注解。如果类使用属性访问，应用对getter方法的约束。

[表21-1](https://docs.oracle.com/javaee/7/tutorial/bean-validation001.htm#GKAGK)列出了Bean Validation的内置约束，在`javax.validation.constraints`包中定义。

表21-1 中列出的所有内置约束都有相应的注解 ConstraintName.List，用于在同一个字段或属性上分组相同类型的多个约束。例如，以下持久性字段有两个`@Pattern`约束：

```java
@Pattern.List({
    @Pattern(regexp="..."),
    @Pattern(regexp="...")
})
```

以下实体类 Contact 具有应用于其持久化字段的  Bean Validation 约束：

```
@Entity
public class Contact implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @NotNull
    protected String firstName;
    @NotNull
    protected String lastName;
    @Pattern(regexp = "[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\\."
            + "[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@"
            + "(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\\.)+[a-z0-9]"
            + "(?:[a-z0-9-]*[a-z0-9])?",
            message = "{invalid.email}")
    protected String email;
    @Pattern(regexp = "^\\(?(\\d{3})\\)?[- ]?(\\d{3})[- ]?(\\d{4})$",
            message = "{invalid.phonenumber}")
    protected String mobilePhone;
    @Pattern(regexp = "^\\(?(\\d{3})\\)?[- ]?(\\d{3})[- ]?(\\d{4})$",
            message = "{invalid.phonenumber}")
    protected String homePhone;
    @Temporal(javax.persistence.TemporalType.DATE)
    @Past
    protected Date birthday;
    ...
}
```


### 实体中的主键

每个实体都有唯一的对象标识符。例如，客户实体可以通过客户号码来标识。唯一标识符或主键（primary key）使客户端能够定位特定实体实例。每个实体都必须有一个主键。实体可以具有简单主键或复合主键。

简单主键使用`javax.persistence.Id`注注解来表示主键属性或字段。

当主键由多个属性组成时，使用复合主键，该属性对应于一组单个持久化属性或字段。复合主键必须在主键类中定义。复合主键使用`javax.persistence.EmbeddedId`和`javax.persistence.IdClass`注解来表示。

主键或复合主键的属性或字段必须是以下Java语言类型之一：

* Java 原始类型
* Java 原始包装类型
* java.lang.String
* java.util.Date（时间类型应为DATE）
* java.sql.Date
* java.math.BigDecimal
* java.math.BigInteger

不应在主键中使用浮点类型。如果使用生成的主键，则只有整型类型是可移植的。

主键类必须满足这些要求。

* 类的访问控制修饰符必须是public。
* 如果使用基于属性的访问，主键类的属性必须为public或protected。
* 该类必须有一个公共默认构造函数。
* 类必须实现 hashCode() 和  equals(Object other) 方法。
* 类必须是可序列化的。
* 复合主键必须被表示并映射到实体类的多个字段或属性，或者必须被表示并映射为可嵌入类。
* 如果类映射到实体类的多个字段或属性，则主键类中的主键字段或属性的名称和类型必须与实体类的名称和类型匹配。

以下主键类是组合键，customerOrder 和 itemId 字段一起唯一标识一个实体：


```java
public final class LineItemKey implements Serializable {
    private Integer customerOrder;
    private int itemId;

    public LineItemKey() {}

    public LineItemKey(Integer order, int itemId) {
        this.setCustomerOrder(order);
        this.setItemId(itemId);
    }

    @Override
    public int hashCode() {
        return ((this.getCustomerOrder() == null
                ? 0 : this.getCustomerOrder().hashCode())
                ^ ((int) this.getItemId()));
    }

    @Override
    public boolean equals(Object otherOb) {
        if (this == otherOb) {
            return true;
        }
        if (!(otherOb instanceof LineItemKey)) {
            return false;
        }
        LineItemKey other = (LineItemKey) otherOb;
        return ((this.getCustomerOrder() == null
                ? other.getCustomerOrder() == null : this.getCustomerOrder()
                .equals(other.getCustomerOrder()))
                && (this.getItemId() == other.getItemId()));
    }

    @Override
    public String toString() {
        return "" + getCustomerOrder() + "-" + getItemId();
    }
    /* Getters and setters */
}
```


### 实体关系中的多重性

多重性是以下类型。

* 一对一（One-to-one）：每个实体实例与另一个实体的单个实例相关。例如，要建模其中每个存储仓包含单个窗口小部件的物理仓库，StorageBin 和  Widget 将具有一对一的关系。一对一关系使用对应的持久性属性或字段上的`javax.persistence.OneToOne`注解。
* 一对多（One-to-many）：实体实例可以与其他实体的多个实例相关。例如，销售订单可以有多个订单项。在订单应用程序中，CustomerOrder 将与 LineItem 具有一对多关系。一对多关系使用对应的持久性属性或字段上的`javax.persistence.OneToMany`注解。
* 多对一（Many-to-one）：实体的多个实例可以与另一个实体的单个实例相关。这种多重性与一对多关系相反。在刚刚提到的示例中，从 LineItem 的角度来看，与 CustomerOrder 的关系是多对一的。多对一关系在对应的持久性属性或字段上使用`javax.persistence.ManyToOne`注解。
* 多对多（Many-to-many）：实体实例可以与彼此的多个实例相关。例如，每个大学课程有很多学生，每个学生可以采取几个课程。因此，在注册申请中，课程和学生将具有多对多关系。多对多关系使用对应的持久化属性或字段上的`javax.persistence.ManyToMany`注解。

### 实体关系中的方向

关系的方向可以是双向的或单向的。双向关系具有拥有侧和反侧。单向关系只有一个拥有方。关系的拥有方决定 Persistence 运行时如何更新数据库中的关系。

#### 双向关系

在双向关系中，每个实体都有一个引用另一个实体的关系字段或属性。通过关系字段或属性，实体类的代码可以访问其相关对象。如果实体具有相关字段，则该实体被称为“知道”其相关对象。例如，如果 CustomerOrder 知道它具有什么 LineItem 实例，并且如果 LineItem 知道它属于哪个 CustomerOrder，则它们具有双向关系。

双向关系必须遵循这些规则。

* 双向关系的反面必须通过使用`@OneToOne`、`@OneToMany`或`@ManyToMany`注解的`mappedBy`元素引用其拥有方。`mappedBy`元素指定实体中作为关系所有者的属性或字段。
* 多对一双向关系的许多方面不能定义`mappedBy`元素。许多方面总是关系的拥有方。
* 对于一对一双向关系，拥有侧对应于包含相应外键的一侧。
* 对于多对多双向关系，任一侧可以是拥有侧。


#### 单向关系

在单向关系中，只有一个实体具有引用另一个实体的关系字段或属性。例如，LineItem 将具有标识产品的关系字段，但产品不具有 LineItem 的关系字段或属性。换句话说，LineItem 知道产品，但产品不知道哪些 LineItem 实例引用它。

#### 查询和关系方向

Java Persistence  查询语言和 Criteria API 查询通常在关系之间导航。关系的方向确定查询是否可以从一个实体导航到另一个实体。例如，查询可以从 LineItem 导航到Product，但不能以相反的方向导航。对于 CustomerOrder 和 LineItem，查询可以在两个方向上导航，因为这两个实体具有双向关系。

#### 级联操作和关系

使用关系的实体通常依赖于关系中另一个实体的存在。例如，订单项是订单的一部分；如果订单被删除，订单项也应该被删除。这被称为级联删除关系。

`javax.persistence.CascadeType`枚举类型定义在关系注释的级联元素中应用的级联操作。下表列出了实体的级联操作。



级联操作 |	描述
----|----
ALL | ALL 级联操作将应用于父实体的相关实体。 所有等同于指定 cascade={DETACH, MERGE, PERSIST, REFRESH, REMOVE}
DETACH | 如果父实体与持久化上下文分离，则相关实体也将被分离。
MERGE | 如果父实体被合并到持久化上下文中，则相关实体也将被合并。
PERSIST | 如果父实体被持久化到持久化上下文中，则相关实体也将被持久化。
REFRESH | 如果父实体在当前持久化上下文中被刷新，相关实体也将被刷新。
REMOVE | 如果父实体从当前持久化上下文中删除，相关实体也将被删除。


## 管理实体

## 查询实体

## 数据库模式创建

 