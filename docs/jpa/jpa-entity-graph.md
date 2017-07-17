# 使用实体图（entity graph）

本章介绍如何使用实体图（entity graph）来创建Java Persistence API操作和查询的访存计划。

实体图是特定持久化查询或操作的模板。在创建提取计划时使用它们，或者在同时检索的持久字段组中使用它们。应用程序开发人员使用提取计划将相关的持久字段分组在一起，以提高运行时性能。

默认情况下，实体字段或属性是懒加载方式的提取。开发人员指定字段或属性作为提取计划的一部分，持久化提供程序会以急加载方式提取它们。

例如，将邮件存储为 EmailMessage 实体的电子邮件应用程序优先获取某些字段而不是其他字段。在邮箱视图中发件人、主题和日期将被最频繁地查看。 EmailMessage 实体具有相关 EmailAttachment 实体的集合。出于性能原因，不应直接提取附件，而是真正需要它们的时候，才去提取，但附件的文件名称很重要。开发此应用程序的开发人员可能会抓取来自 EmailMessage 和 EmailAttachment 的重要字段，从而懒加载获取较低优先级的数据。

本章节主要讨论以下主题：

* 实体图基础
* 使用命名实体图
* 在查询操作中使用实体图

## 实体图基础

您可以通过使用注解或部署描述符，或通过使用标准接口动态创建实体图。

您可以通过 EntityManager.find 方法使用实体图，或者作为 JPQL 或 Criteria API 查询的一部分，通过将实体图指定为操作或查询的提示来使用。

实体图具有对应于在查找或查询操作期间将被热切提取的字段的属性。 实体类的主键和版本字段始终被提取，并且不需要显式地添加到实体图。

### 默认实体图

默认情况下，实体中的所有字段将被以懒加载方式提取，除非实体元数据的 fetch 属性设置为 javax.persistence.FetchType.EAGER。 默认实体图包括实体的所有字段，其字段设置为急加载提取。

例如，以下 EmailMessage 实体指定某些字段将被急加载地提取：

```
@Entity
public class EmailMessage implements Serializable {
    @Id
    String messageId;
    @Basic(fetch=EAGER)
    String subject;
    String body;
    @Basic(fetch=EAGER)
    String sender;
    @OneToMany(mappedBy="message", fetch=LAZY)
    Set<EmailAttachment> attachments;
    ...
}
```


此实体的默认实体图表将包含 messageId、subject 和 sender 字段，但不包含正文或附件字段。

### 在持久化操作中使用实体图

实体图通过调用命名实体图的 EntityManager.getEntityGraph 或创建动态实体图的 EntityManager.createEntityGraph 来创建 javax.persistence.EntityGraph 接口的实例。

命名实体图（named entity graph）是由应用于实体类的 ``@NamedEntityGraph``注解指定的实体图，或者是应用程序部署描述符中的  named-entity-graph 元素。在部署描述符中定义的命名实体图形会覆盖任何基于注解的具有相同名称的实体图形。

创建的实体图可以是提取图（fetch graph）或负载图（load graph）。

#### 提取图

要指定提取图，请在执行 EntityManager.find 或查询操作时设置 javax.persistence.fetchgraph 属性。提取图仅由在 EntityGraph 实例中显式指定的字段组成，并忽略默认实体图设置。

在以下示例中，默认实体图被忽略，并且只有 body 字段包含在动态创建的提取图中：

```
EntityGraph<EmailMessage> eg = em.createEntityGraph(EmailMessage.class);
eg.addAttributeNodes("body");
...
Properties props = new Properties();
props.put("javax.persistence.fetchgraph", eg);
EmailMessage message = em.find(EmailMessage.class, id, props);
```


#### 加载图

要指定加载图，请在执行 EntityManager.find 或查询操作时设置 javax.persistence.loadgraph 属性。 加载图由在 EntityGraph 实例中显式指定的字段加上默认实体图中的任何字段组成。

在以下示例中，动态创建的负载图包含默认实体图中的所有字段以及正文字段：


```
EntityGraph<EmailMessage> eg = em.createEntityGraph(EmailMessage.class);
eg.addAttributeNodes("body");
...
Properties props = new Properties();
props.put("javax.persistence.loadgraph", eg);
EmailMessage message = em.find(EmailMessage.class, id, props);
```

## 使用命名实体图


使用应用于应用程序的部署描述符中的实体类或 named-entity-graph 元素及其子元素的注解来创建命名实体图。持久性提供程序将扫描应用程序中的所有命名实体图形（在注解和XML中定义）。使用注解的命名实体图集可以使用 named-entity-graph 来重写。

### 将命名实体图注解应用于实体类

javax.persistence.NamedEntityGraph 注解定义单个命名实体图并在类级别应用。可以通过在 javax.persistence.NamedEntityGraphs 类级别注解中添加多个``@NamedEntityGraph``注解来为类定义多个``@NamedEntityGraph``注解。

`@NamedEntityGraph`注解必须应用于实体图的根。也就是说，如果 EntityManager.find 或查询操作具有 EmailMessage 类的根实体，则必须在 EmailMessage 类中定义操作中使用的命名实体图：

```
@NamedEntityGraph
@Entity
public class EmailMessage {
    @Id
    String messageId;
    String subject;
    String body;
    String sender;
}
```

在这个例子中，EmailMessage 类有一个`@NamedEntityGraph`注解来定义一个命名实体图，默认为类的名称 EmailMessage。 `@NamedEntityGraph`注解中不包含作为属性节点的字段，并且这些字段未使用元数据进行注解以设置提取类型，因此在加载图或提取图中热切获取的唯一字段为 messageId。

命名实体图的属性是应该包括在实体图中的实体的字段。通过在`@NamedEntityGraph`的 attributeNodes 元素中使用 javax.persistence.NamedAttributeNode 注解指定字段，将字段添加到实体图中：

```
@NamedEntityGraph(name="emailEntityGraph", attributeNodes={
    @NamedAttributeNode("subject"),
    @NamedAttributeNode("sender")
})
@Entity
public class EmailMessage { ... }
```

在此示例中，命名实体图的名称为 emailEntityGraph，并包括 subject 和 sender 字段。

多个`@NamedEntityGraph`定义可以通过在`@NamedEntityGraphs`注解中将它们分组来应用于类。

在下面的示例中，在 EmailMessage 类上定义了两个实体图。一个是预览窗格，它只获取邮件的发件人、主题和正文。另一个是邮件的完整视图，包括所有邮件的附件：

```
@NamedEntityGraphs({
    @NamedEntityGraph(name="previewEmailEntityGraph", attributeNodes={
        @NamedAttributeNode("subject"),
        @NamedAttributeNode("sender"),
        @NamedAttributeNode("body")
    }),
    @NamedEntityGraph(name="fullEmailEntityGraph", attributeNodes={
        @NamedAttributeNode("sender"),
        @NamedAttributeNode("subject"),
        @NamedAttributeNode("body"),
        @NamedAttributeNode("attachments")
    })
})
@Entity
public class EmailMessage { ... }
```


### 从命名实体图获取 EntityGraph 实例

使用 EntityManager.getEntityGraph 方法，传入命名实体图形名称，以获取命名实体图的 EntityGraph 实例：

```
EntityGraph<EmailMessage> eg = em.getEntityGraph("emailEntityGraph");
```

## 在查询操作中使用实体图



要为类型化和非类型化查询指定实体图形，请对查询对象调用 setHint 方法，并指定 javax.persistence.loadgraph 或 javax.persistence.fetchgraph 作为属性名称，并指定 EntityGraph 实例作为值：

```
EntityGraph<EmailMessage> eg = em.getEntityGraph("previewEmailEntityGraph");
List<EmailMessage> messages = em.createNamedQuery("findAllEmailMessages")
        .setParameter("mailbox", "inbox")
        .setHint("javax.persistence.loadgraph", eg)
        .getResultList();
```

在此示例中，previewEmailEntityGraph 用于 findAllEmailMessages 命名查询。

输入的查询使用相同的技术：

```
EntityGraph<EmailMessage> eg = em.getEntityGraph("previewEmailEntityGraph");

CriteriaQuery<EmailMessage> cq = cb.createQuery(EmailMessage.class);
Root<EmailMessage> message = cq.from(EmailMessage.class);
TypedQuery<EmailMessage> q = em.createQuery(cq);
q.setHint("javax.persistence.loadgraph", eg);
List<EmailMessage> messages = q.getResultList();
```
