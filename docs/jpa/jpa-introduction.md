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

`javax.persistence.CascadeType`枚举类型定义在关系注解的级联元素中应用的级联操作。下表列出了实体的级联操作。



级联操作 |	描述
----|----
ALL | ALL 级联操作将应用于父实体的相关实体。 所有等同于指定 cascade={DETACH, MERGE, PERSIST, REFRESH, REMOVE}
DETACH | 如果父实体与持久化上下文分离，则相关实体也将被分离。
MERGE | 如果父实体被合并到持久化上下文中，则相关实体也将被合并。
PERSIST | 如果父实体被持久化到持久化上下文中，则相关实体也将被持久化。
REFRESH | 如果父实体在当前持久化上下文中被刷新，相关实体也将被刷新。
REMOVE | 如果父实体从当前持久化上下文中删除，相关实体也将被删除。


级联删除关系使用`@OneToOne`和`@OneToMany`关系的`cascade=REMOVE`元素指定。 例如：

```
@OneToMany(cascade=REMOVE, mappedBy="customer")
public Set<CustomerOrder> getOrders() { return orders; }
```

#### 关系中的孤儿删除

当从关系中去除一对一或一对多关系中的目标实体时，通常希望将删除操作级联到目标实体。 这样的目标实体被认为是“孤儿”，并且`orphanRemoval`属性可以用于指定应该移除孤立的实体。 例如，如果订单有许多订单项，其中一个订单项从订单中删除，则删除的订单项将被视为孤儿。 如果`orphanRemoval`设置为true，则在从订单中删除订单项时，订单项实体将被删除。

`@OneToMany`和`@oneToOne`中的`orphanRemoval`属性采用布尔值，默认值为 false。

以下示例将在删除客户实体时将删除操作级联到孤儿订单实体：

```
@OneToMany(mappedBy="customer", orphanRemoval="true")
public List<CustomerOrder> getOrders() { ... }
```

### 实体中的可嵌入类

可嵌入类用于表示实体的状态，但不具有它们自己的持久性标识，这点与实体类不同。 可嵌入类的实例共享拥有它的实体的身份。 可嵌入类仅作为另一个实体的状态存在。 实体可以具有单值或集合值可嵌入类属性。

可嵌入类与实体类具有相同的规则，但是使用`javax.persistence.Embeddable`注解而不是`@Entity`进行注解。

以下可嵌入类ZipCode具有字段zip和plusFour：

```
@Embeddable
public class ZipCode {
    String zip;
    String plusFour;
    ...
}
```

以下是可嵌入类在 Address 实体中的使用：

```
@Entity
public class Address {
    @Id
    protected long id
    String street1;
    String street2;
    String city;
    String province;
    @Embedded
    ZipCode zipCode;
    String country;
    ...
}
```

拥有可嵌入类作为其持久状态的一部分的实体可以使用`javax.persistence.Embedded`来注解字段或属性，但不是必须这样做的。

可嵌入类本身可以使用其他可嵌入类来表示它们的状态。 它们还可以包含基本Java编程语言类型或其他可嵌入类的集合。 可嵌入类也可以包含与其他实体或实体集合的关系。 如果可嵌入类有这样的关系，则关系是从目标实体或实体集合到拥有可嵌入类的实体。

## 实体继承

实体支持类继承、多态关联和多态查询。 实体类可以扩展非实体类，非实体类可以扩展实体类。 实体类可以是抽象的和具体的。

花名册示例应用程序演示实体继承，如名单应用程序中的实体继承中所述。

### 抽象实体

抽象类可以通过用`@Entity`装饰类来声明一个实体。 抽象实体就像具体实体，但不能实例化。

抽象实体可以像具体实体一样被查询。 如果抽象实体是查询的目标，则查询对抽象实体的所有具体子类进行操作：

```
@Entity
public abstract class Employee {
    @Id
    protected Integer employeeId;
    ...
}
@Entity
public class FullTimeEmployee extends Employee {
    protected Integer salary;
    ...
}
@Entity
public class PartTimeEmployee extends Employee {
    protected Float hourlyWage;
}
```


### 映射超类

实体可以从包含持久状态和映射信息但不是实体的超类继承。 也就是说，超类不使用`@Entity`注解进行修饰，并且不由Java Persistence提供程序映射为实体。 当您具有多个实体类共用的状态和映射信息时，最常使用这些超类。

映射的超类通过用注解`javax.persistence.MappedSuperclass`装饰类来指定：

```
@MappedSuperclass
public class Employee {
    @Id
    protected Integer employeeId;
    ...
}
@Entity
public class FullTimeEmployee extends Employee {
    protected Integer salary;
    ...
}
@Entity
public class PartTimeEmployee extends Employee {
    protected Float hourlyWage;
    ...
}
```

映射的超类不能被查询，不能在EntityManager或Query操作中使用。 您必须在EntityManager或Query操作中使用映射超类的实体子类。 映射的超类不能是实体关系的目标。 映射的超类可以是抽象的或具体的。

映射的超类在底层数据存储中没有任何对应的表。 从映射的超类继承的实体定义表映射。 例如，在上面的代码示例中，基础表将是 FULLTIMEEMPLOYEE 和 PARTTIMEEMPLOYEE，但没有 EMPLOYEE 表。

### 非实体超类

实体可以具有非实体超类，并且这些超类可以是抽象的或具体的。 非实体超类的状态是非持久的，并且通过实体类从非实体超类继承的任何状态是非持久的。 非实体超类不能在EntityManager或Query操作中使用。 将忽略非实体超类中的任何映射或关系注解。

### 实体继承映射策略

您可以配置Java Persistence提供程序如何通过使用注解`javax.persistence.Inheritance`装饰层次结构的根类来将继承的实体映射到基础数据存储库。 以下映射策略用于将实体数据映射到底层数据库：

* 每个类层次结构的单个表
* 每个具体实体类的表
* “连接（join）”策略，其中特定于子类的字段或属性被映射到与父类共同的字段或属性不同的表

策略通过将`@Inheritance`的 strategy 元素设置为`javax.persistence.InheritanceType`枚举类型中定义的选项之一来配置：

```
public enum InheritanceType {
    SINGLE_TABLE,
    JOINED,
    TABLE_PER_CLASS
};
```

如果未在实体层次结构的根类上指定`@Inheritance`注解，则使用默认策略 InheritanceType.SINGLE_TABLE。

#### 单类表层次结构策略

使用此策略（对应于默认的InheritanceType.SINGLE_TABLE），层次结构中的所有类都将映射到数据库中的单个表。 该表具有包含标识该行所表示的实例所属的子类的值的鉴别器列。

可以通过在实体类层次结构的根上使用`javax.persistence.DiscriminatorColumn`注解来指定其元素如下表所示的鉴别器列。


类型 | 名称 | 描述
---- | ---- | ----
String | name | The name of the column to be used as the discriminator column. The default is DTYPE. This element is optional.
DiscriminatorType | discriminatorType | The type of the column to be used as a discriminator column. The default is DiscriminatorType.STRING. This element is optional.
String | columnDefinition | The SQL fragment to use when creating the discriminator column. The default is generated by the Persistence provider and is implementation-specific. This element is optional.
String | length | The column length for String-based discriminator types. This element is ignored for non-String discriminator types. The default is 31. This element is optional


`javax.persistence.DiscriminatorType` 枚举类型用于通过将`@DiscriminatorColumn`的 discriminatorType 元素设置为定义的类型之一来设置数据库中的鉴别器列的类型。 DiscriminatorType定义如下：

```
public enum DiscriminatorType {
    STRING,
    CHAR,
    INTEGER
};
```

如果未在实体层次结构的根上指定`@DiscriminatorColumn`，并且需要使用区分符列，则Persistence提供程序假定缺省列名为DTYPE，列类型为DiscriminatorType.STRING。

`javax.persistence.DiscriminatorValue`注解可用于为类层次结构中的每个实体设置输入到discriminator列的值。您可以使用`@DiscriminatorValue`仅装饰具体的实体类。

如果未在类层次结构中使用鉴别器列的实体上指定`@DiscriminatorValue`，那么持久性提供程序将提供缺省的特定于实现的值。如果`@DiscriminatorColumn`的discriminatorType元素为DiscriminatorType.STRING，则默认值为实体的名称。

此策略为实体和覆盖整个实体类层次结构的查询之间的多态关系提供良好的支持。但是，此策略要求包含子类的状态的列为可空的。


#### 每个具体类策略的表

在此策略中，对应于InheritanceType.TABLE_PER_CLASS，每个具体类都映射到数据库中的单独表。 类中的所有字段或属性（包括继承的字段或属性）都映射到数据库中类的表中的列。

此策略对多态关系提供较差的支持，通常需要对包含整个实体类层次结构的查询的每个子类的SQL UNION查询或单独的SQL查询。

对此策略的支持是可选的，并且可能不受所有 Java Persistence API  提供程序支持。 GlassFish Server中的默认Java Persistence API提供程序不支持此策略。

### 连接子类策略

在此策略中，对应于InheritanceType.JOINED，类层次结构的根由单个表表示，每个子类具有单独的表，其中仅包含特定于该子类的字段。也就是说，子类表不包含用于继承字段或属性的列。子类表还具有表示其主键的一个或多个列，其是超类表的主键的外键。

此策略为多态关系提供良好的支持，但需要在实例化实体子类时执行一个或多个连接操作。这可能导致广泛的类层次结构的性能差。类似地，覆盖整个类层次结构的查询需要子类表之间的连接操作，从而导致性能降低。

某些Java Persistence API提供程序（包括GlassFish Server中的默认提供程序）在使用连接的子类策略时需要与根实体对应的鉴别器列。如果您未在应用程序中使用自动表创建，请确保针对标识符列缺省值正确设置数据库表，或使用`@DiscriminatorColumn`注解来匹配数据库模式。 

## 管理实体

实体由实体管理器管理，它由`javax.persistence.EntityManager`实例表示。 每个EntityManager实例与持久上下文相关联：存在于特定数据存储中的一组被管实体实例。 持久化上下文定义了创建、持久化和删除特定实体实例的范围。 EntityManager接口定义用于与持久性上下文进行交互的方法。

### EntityManager 接口

EntityManager API创建和删除持久实体实例，通过实体的主键查找实体，并允许在实体上运行查询。

#### 容器管理的实体管理器

对于容器管理的实体管理器，EntityManager 实例的持久化上下文由容器自动传播到在单个 Java Transaction API (JTA) 事务中使用 EntityManager 实例的所有应用程序组件。

JTA事务通常涉及跨应用程序组件的调用。要完成JTA事务，这些组件通常需要访问单个持久性上下文。当EntityManager通过`javax.persistence.PersistenceContext`注解注入到应用程序组件时，会发生这种情况。持久化上下文自动地与当前JTA事务一起传播，并且映射到相同持久性单元的EntityManager引用提供对该事务内的持久性上下文的访问。通过自动传播持久化上下文，应用程序组件不需要将对EntityManager实例的引用彼此传递，以便在单个事务中进行更改。 Java EE容器管理容器管理的实体管理器的生命周期。

要获取EntityManager实例，请将实体管理器注入应用程序组件：

```
@PersistenceContext
EntityManager em;
```


#### 应用程序管理的实体管理器

另一方面，对于应用程序管理的实体管理器，持久化上下文不会传播到应用程序组件，并且EntityManager实例的生命周期由应用程序管理。

当应用程序需要访问不通过特定持久性单元中的 EntityManager 实例使用JTA事务传播的持久性上下文时，使用应用程序管理的实体管理器。在这种情况下，每个EntityManager创建一个新的，隔离的持久上下文。 EntityManager及其相关联的持久化上下文由应用程序显式创建和销毁。它们也用于直接注入EntityManager实例时无法完成，因为EntityManager实例不是线程安全的。 EntityManagerFactory实例是线程安全的。

在这种情况下，应用程序使用`javax.persistence.EntityManagerFactory`的createEntityManager方法创建EntityManager实例。

要获取EntityManager实例，首先必须通过`javax.persistence.PersistenceUnit`注解将EntityManagerFactory实例注入到应用程序组件中：

```
@PersistenceUnit
EntityManagerFactory emf;
```

从  EntityManagerFactory 获得 EntityManager :

```
EntityManager em = emf.createEntityManager();
```

应用程序管理的实体管理器不会自动传播JTA事务上下文。 当执行实体操作时，这样的应用程序需要手动地获得对JTA事务管理器的访问并且添加事务划分信息。 `javax.transaction.UserTransaction`接口定义用于开始、提交和回滚事务的方法。 通过创建用`@Resource`注解的实例变量来注入UserTransaction的实例：

```
@Resource
UserTransaction utx;
```

要开始事务，请调用UserTransaction.begin方法。 当所有实体操作完成时，调用UserTransaction.commit方法提交事务。 UserTransaction.rollback方法用于回滚当前事务。

以下示例显示如何在使用应用程序管理的实体管理器的应用程序中管理事务：

```
@PersistenceUnit
EntityManagerFactory emf;
EntityManager em;
@Resource
UserTransaction utx;
...
em = emf.createEntityManager();
try {
    utx.begin();
    em.persist(SomeEntity);
    em.merge(AnotherEntity);
    em.remove(ThirdEntity);
    utx.commit();
} catch (Exception e) {
    utx.rollback();
}
```

#### 使用EntityManager查找实体

EntityManager.find方法用于通过实体的主键在数据存储中查找实体：

```
@PersistenceContext
EntityManager em;
public void enterOrder(int custID, CustomerOrder newOrder) {
    Customer cust = em.find(Customer.class, custID);
    cust.getOrders().add(newOrder);
    newOrder.setCustomer(cust);
}
```

#### 管理实体实例的生命周期

通过EntityManager实例调用实体上的操作来管理实体实例。 实体实例处于四种状态之一：新建（new）、受管（managed）、分离（detached）或删除（removed）。

* 新建实体实例没有持久性标识，并且尚未与持久性上下文相关联。
* 受管实体实例具有持久性标识，并与持久性上下文相关联。
* 分离的实体实例具有持久性标识，并且当前不与持久性上下文相关联。
* 删除的实体具有持久性标识，与持久上下文相关联，并且被调度为从数据存储中移除。

#### 持久化实体实例

新建实体实例通过调用 persist 方法或通过从关系注解中设置了 cascade=PERSIST 或 cascade=ALL 元素的相关实体调用的级联 persist 操作来成为管理和持久性。 这意味着当与 persist 操作相关联的事务完成时，实体的数据被存储到数据库。 如果实体已经被管理，则 persist 操作将被忽略，尽管 persist 操作将级联到在关系注解中具有级联元素设置为PERSIST或ALL的相关实体。 如果在删除的实体实例上调用persist，则该实体将成为受管实体。 如果实体被分离，则persist将抛出IllegalArgumentException，否则事务提交将失败。 以下方法执行 persist 操作：

```
@PersistenceContext
EntityManager em;
...
public LineItem createLineItem(CustomerOrder order, Product product,
        int quantity) {
    LineItem li = new LineItem(order, product, quantity);
    order.getLineItems().add(li);
    em.persist(li);
    return li;
}
```

persist 操作传播到与关系注解中的 cascade  元素设置为ALL或PERSIST的调用实体相关的所有实体：

```
@OneToMany(cascade=ALL, mappedBy="order")
public Collection<LineItem> getLineItems() {
    return lineItems;
}
```

#### 删除实体实例

通过调用remove方法或通过从关系注解中设置了 cascade=REMOVE 或 cascade=ALL 元素的相关实体调用的级联 remove 操作来删除托管实体实例。 如果对新实体调用remove方法，则忽略remove操作，尽管remove将级联到在关系注解中将 cascade 元素设置为REMOVE或ALL的相关实体。 如果在分离的实体上调用remove，则remove将抛出IllegalArgumentException，否则事务提交将失败。 如果在已删除的实体上调用，则会忽略  remove 。 当事务完成或作为 flush 操作的结果时，实体的数据将从数据存储中移除。

在以下示例中，与订单关联的所有LineItem实体也被删除，因为CustomerOrder.getLineItems在关系注解中设置了 cascade=ALL：



```
public void removeOrder(Integer orderId) {
    try {
        CustomerOrder order = em.find(CustomerOrder.class, orderId);
        em.remove(order);
    }...
```


#### 将实体数据同步到数据库

当与实体相关联的事务提交时，持久实体的状态被同步到数据库。 如果被管实体与另一个被管实体具有双向关系，则数据将基于关系的所有者侧被持久化。

要强制将受管实体同步到数据存储，请调用EntityManager实例的flush方法。 如果实体与另一个实体相关，并且关系注解具有设置为PERSIST或ALL的 cascade 元素，则当调用flush时，相关实体的数据将与数据存储器同步。

如果实体被删除，调用flush将从数据存储中删除实体数据。

### 持久单元

持久单元（persistence uni）定义由应用程序中的 EntityManager 实例管理的所有实体类的集合。 这组实体类表示包含在单个数据存储中的数据。

持久单元由persistence.xml配置文件定义。 以下是persistence.xml文件的示例：

```xml
<persistence>
    <persistence-unit name="OrderManagement">
        <description>This unit manages orders and customers.
            It does not rely on any vendor-specific features and can
            therefore be deployed to any persistence provider.
        </description>
        <jta-data-source>jdbc/MyOrderDB</jta-data-source>
        <jar-file>MyOrderApp.jar</jar-file>
        <class>com.widgets.CustomerOrder</class>
        <class>com.widgets.Customer</class>
    </persistence-unit>
</persistence>
```

此文件定义名为 OrderManagement 的持久单元，该单元使用支持 JTA 的数据源`jdbc/MyOrderDB`。`jar-file`和`class`元素指定管理的持久类：实体类、可嵌入类和映射超类。 `jar-file`元素指定包装管理的持久类的打包持久单元可见的 JAR 文件，而 class 元素显式地命名管理的持久类。

jta-data-source（用于JTA感知的数据源）和  non-jta-data-source（用于非JTA感知的数据源）元素指定要由容器使用的数据源的全局JNDI名称。

META-INF 目录包含 persistence.xml的JAR文件或目录称为持单元的根。持久单元的范围由持久单元的根确定。每个持久单元必须使用对持久单元范围唯一的名称标识。

持久单元可以打包为WAR或EJB JAR文件的一部分，也可以打包为JAR文件，然后将其包含在WAR或EAR文件中。

* 如果将持久性单元打包为EJB JAR文件中的一组类，那么persistence.xml应放在EJB JAR的META-INF目录中。
* 如果将持久化单元打包为WAR文件中的一组类，persistence.xml应位于WAR文件的  WEB-INF/classes/META-INF 目录中。
* 如果将持久性单元打包到将包含在WAR或EAR文件中的JAR文件中，则JAR文件应位于
	* WAR的  WEB-INF/lib 目录
	* 或者EAR文件的库目录
 
注意：
在Java Persistence API 1.0中，JAR文件可以位于EAR文件的根位置，作为持久单元的根。 这个在新版本中不再支持。 便携式应用程序应该使用EAR文件的库目录作为持久单元的根。


## 查询实体
 

Java Persistence API提供以下用于查询实体的方法。

* Java 持久化查询语言（Java Persistence query language，简称 JPQL）是一种简单的基于字符串的语言，类似于用于查询实体及其关系的SQL。
* Criteria API用于使用Java编程语言API来创建类型安全查询，以查询实体及其关系。 

JPQL和Criteria API都有优点和缺点。

JPQL查询通常比 Criteria 查询更简洁和更可读。熟悉SQL的开发人员会发现很容易学习JPQL的语法。 JPQL命名查询可以在实体类中使用Java编程语言注解或在应用程序的部署描述符中定义。但是，JPQL查询不是类型安全的，并且在从实体管理器检索查询结果时需要转换。这意味着在编译时可能不会捕获类型转换错误。 JPQL查询不支持开放式参数。

Criteria 查询允许您在应用程序的业务层中定义查询。尽管使用JPQL动态查询也是可能的，但是Criteria查询提供更好的性能，因为每次调用JPQL动态查询时都必须解析。Criteria 查询是类型安全的，因此不需要转换，就像JPQL查询那样。 Criteria API只是另一种Java编程语言API，不需要开发人员学习另一种查询语言的语法。Criteria 查询通常比JPQL查询更详细，并且需要开发人员在向实体管理器提交查询之前创建多个对象并对这些对象执行操作。

## 数据库模式创建


持久化提供程序可以配置为在应用程序部署期间使用应用程序部署描述符中的标准属性自动创建数据库表、将数据加载到表中以及删除表。 这些任务通常在发布的开发阶段使用，而不是针对生产数据库。

以下是persistence.xml部署描述符的示例，它指定提供程序应使用提供的脚本删除所有数据库工件，使用提供的脚本创建工件，以及在部署应用程序时从提供的脚本加载数据：

```
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
 http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
  <persistence-unit name="examplePU" transaction-type="JTA">
    <jta-data-source>java:global/ExampleDataSource</jta-data-source>
    <properties>
        <property name="javax.persistence.schema-generation.database.action"
                  value="drop-and-create"/>
        <property name="javax.persistence.schema-generation.create-source"
                  value="script"/>
        <property name="javax.persistence.schema-generation.create-script-source"
                  value="META-INF/sql/create.sql" />
        <property name="javax.persistence.sql-load-script-source"
                  value="META-INF/sql/data.sql" />
        <property name="javax.persistence.schema-generation.drop-source"
                  value="script" />
        <property name="javax.persistence.schema-generation.drop-script-source"
                  value="META-INF/sql/drop.sql" />
    </properties>
  </persistence-unit>
</persistence>
```

### 配置应用程序以创建或删除数据库表

javax.persistence.schema-generation.database.action属性用于指定部署应用程序时持久化提供程序所采取的操作。 如果未设置属性，持久化提供程序将不会创建或删除任何数据库工件。
 

设置	| 描述
---- | ----
none | 不会创建或删除模式。
create | 提供程序将在应用程序部署上创建数据库工件。 应用程序重新部署后，工件将保持不变。
drop-and-create | 数据库中的任何工件将被删除，并且提供程序将在部署时创建数据库工件。
drop | 应用程序部署时将删除数据库中的任何工件。



在此示例中，持久化提供程序将删除任何剩余的数据库工件，然后在部署应用程序时创建工件：

```
<property name="javax.persistence.schema-generation.database.action"
           value="drop-and-create"/>
```


默认情况下，持久单元中的对象/关系元数据用于创建数据库工件。 您还可以提供供应商用来创建和删除数据库工件的脚本。`javax.persistence.schema-generation.create-source`和`javax.persistence.schema-generation.drop-source`属性控制提供程序将如何创建或删除数据库工件。



设置	| 描述
---- | ----
metadata | 使用应用程序中的对象/关系元数据创建或删除数据库工件。
script |使用提供的脚本创建或删除数据库工件。
metadata-then-script | 使用对象/关系元数据的组合，然后使用用户提供的脚本创建或删除数据库工件。
script-then-metadata  | 使用用户提供的脚本，然后使用对象/关系元数据来创建和删除数据库工件。


在此示例中，持久化提供程序将使用在应用程序中打包的脚本来创建数据库工件：

```
<property name="javax.persistence.schema-generation.create-source"
           value="script"/>
```

如果在 create-source 或 drop-sourc e中指定脚本，请使用　javax.persistence.schema-generation.create-script-source　或　javax.persistence.schema-generation.drop-script-source　属性指定脚本的位置 。 脚本的位置是相对于持久单元的根：

```
<property name="javax.persistence.schema-generation.create-script-source"
           value="META-INF/sql/create.sql" />
```

在上面的示例中，create-script-source 设置为  META-INF/sql 目录中名为create.sql的SQL文件，相对于持久单元的根。

###　使用SQL脚本加载数据

如果要在应用程序加载之前使用数据填充数据库表，请在　javax.persistence.sql-load-script-source　属性中指定加载脚本的位置。 此属性中指定的位置是相对于持久单元的根。

在此示例中，加载脚本是　 META-INF/sql　目录中名为data.sql的文件，相对于持久性单元的根目录：