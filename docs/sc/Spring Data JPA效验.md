一、在已有项目中导入`spring-boot-starter-data-jpa`依赖：

- 若是`Maven`项目，则导入以下依赖：
  
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <version>2.0.6.RELEASE</version>
    </dependency>
```
- 若是`Gradle`项目，则导入以下依赖：

```
compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa', version: '2.0.6.RELEASE'
```

二、常用注解说明如下：

1.@Table(name="",catalog="",schema="")
只能标注在实体的class定义处,表示实体对应的数据库表的信息
name:可选,表示表的名称.默认地,表名和实体名称一致,只有在不一致的情况下才需要指定表名
catalog:可选,表示Catalog名称,默认为Catalog("").
schema:可选,表示Schema名称,默认为Schema("").

2.@Id、@GeneratedValue(strategy=GenerationType, generator="")
这两个注解通常组合使用，作用在主键上面，指定主键生成策略。
@GeneratedValue有四种主键生成策略：有AUTO、INDENTITY、SEQUENCE 和 TABLE 4种,分别表示让ORM框架自动选择、根据数据库的Identity字段生成、根据数据库表的Sequence字段生成和根据一个额外的表生成主键,默认为AUTO

示例:

```
@Data
public class User {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private int id;
    private String name;
    //....
}
```

3.@Column

name:表示数据库表中该字段的名称,默认情形属性名称一致。
nullable:表示该字段是否允许为null,默认为true。
unique:表示该字段是否是唯一标识,默认为false。
length:表示该字段的大小,仅对String类型的字段有效。 
insertable:表示在ORM框架执行插入操作时,该字段是否应出现INSETRT语句中,默认为true。
updateable:表示在ORM框架执行更新操作时,该字段是否应该出现在UPDATE语句中,默认为true。

```
    @Column(name ="name",updatable = true ,insertable = true)
    private String name;
```

4.@Transient

表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。 
如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。

```
    @Transient
    private String name;
```

5.@Null	
表示被注释的属性必须为 null。

6.@NotNull
被注释的属性必须不为 null。

7.@AssertTrue
被注释的元素必须为 true

8.@AssertFalse
被注释的元素必须为 false

9.@Min(value)
被注释的元素必须是一个数字，其值必须大于等于指定的最小值

10.@Max(value)	
被注释的元素必须是一个数字，其值必须小于等于指定的最大值

11.@DecimalMin(value)	被注释的元素必须是一个数字，其值必须大于等于指定的最小值

12.@DecimalMax(value)	被注释的元素必须是一个数字，其值必须小于等于指定的最大值

13.@Size(max=, min=)
被注释的元素的大小必须在指定的范围内

14.@Digits (integer, fraction)	被注释的元素必须是一个数字，其值必须在可接受的范围内
15.@Past
被注释的元素必须是一个过去的日期

16.@Future
被注释的元素必须是一个将来的日期
17.@Pattern(regex=,flag=)
被注释的元素必须符合指定的正则表达式

18.@NotBlank(message =)	验证字符串非null，且长度必须大于0
19.@Email
被注释的元素必须是电子邮箱地址

20.@Length(min=,max=)
被注释的字符串的大小必须在指定的范围内

21.@NotEmpty
被注释的字符串的必须非空

22.@Range(min=,max=,message=)
被注释的元素必须在合适的范围内

三、如何使配置生效

在`controller`的方法参数前面添加`@valid`注解使验证生效

```
    @GetMapping("/{message}")
    public String getMessage(@PathVariable @Valid String message) {
        return "service1" + providerService.getMessage(message);
    }
```



















  