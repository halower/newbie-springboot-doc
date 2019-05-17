# JPA补充说明
支持所有的JPA功能,用法不变，建议优先使用毕竟非常简单,复杂场景再考虑`QueryDsl`扩展接口

!> JPA的部分不做太多的说明，以下对新人可能遇到的一些场景网上不方便找到的问题做一些解决方案，当然可以查看官网有限参考也是最全面的

## 如何处理复合主键
> 假设我们之前用到的`Blog`示例中，需要使用 `title`和 `url`作为复合主键，我们就需要创建以下类

```
@Data
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class BlogKey implements Serializable {
    private String title;
    private String url;
}
```

然后我们在对`Blog`实体类加上`@IdClass`注解就可以,这里需要注意的是两个主键都需要加`@Id`注解

```
@Data
@Entity
@IdClass(BlogKey.class)
@EqualsAndHashCode(callSuper = false)
public class Blog extends BaseAudited {
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name ="id", nullable = false)
    private int blogId;

    @Id
    @Column(name = "title", nullable = false, length = 50)
    private String title ;

    @Id
    @Column(name = "url", nullable = false, length = 150)
    private String url;

    @OneToMany(mappedBy ="blog",cascade=CascadeType.ALL,fetch=FetchType.LAZY)
    private List<Post> posts ;

    public void addPost(Post post){
        this.getPosts().add(post);
    }
}
```
## ID生成策略

> JPA提供四种标准用法,由`@GeneratedValue`的源代码可以明显看出.

JPA提供的`GenerationType`四种标准用法为`TABLE`,`SEQUENCE`,`IDENTITY`,`AUTO`.

1. `TABLE`：使用一个特定的数据库表格来保存主键。
1. `SEQUENCE`：根据底层数据库的序列来生成主键，条件是数据库支持序列。
1. `IDENTITY`：主键由数据库自动生成（主要是自动增长型）
1. `AUTO`：主键由程序控制(也是默认的,在指定主键时，如果不指定主键生成策略，默认为AUTO)

###  常见数据库的支持情况如下
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;数据库&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;支持项&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; |
| ------------   | ------------|
|*MYSQL*| `GenerationType.TABLE`<br> `GenerationType.AUTO`<br> `GenerationType.IDENTITY`<br> ~~`GenerationType.SEQUENCE`~~<br>|
|*ORACLE*|`GenerationType.TABLE`<br> `GenerationType.AUTO`<br>~~`GenerationType.IDENTITY`<br>~~ `GenerationType.SEQUENCE`<br>|

### 使用方法
在我们的应用中，一般选用`@GeneratedValue(strategy=GenerationType.AUTO)`这种方式，自动选择主键生成策略，以适应不同的数据库移植。也可以使用`Hibernate`对主键生成策略的扩展，通过`Hibernate`的`@GenericGenerator`实现。即
```
    @Id
    @GeneratedValue
    private String id;
```

> 下面使我们正在使用一个例子

```
    @Id
     // 声明一个策略通用生成器，name为"system-uuid",策略strategy为"uuid"
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
     // 用generator属性指定要使用的策略生成器
    @GeneratedValue(generator = "system-uuid")
    @Column(name = "log_id")
    private String id;
```
这是`Log`模型使用的一种方式，生成32位的字符串，适用于所有数据库。

## 如何进行部分字段查询
?> `JPA`为此提供了三种解决方案，但是考虑到`最少知识` 原则，直接复用`Dto`即可

```
public BlogTitlesOnlyDto {
    private String title;
    public BlogTitlesOnlyDto(String title) {
        this.title = title;
    }
    ...
}
```
使用也很简单  
```
public interface BlogRepository extends CrudRepository<Blog,Integer> {
    List<BlogTitlesOnlyDto> findAllByTitle(String title); 
}
```
> 但是这个场景好像只是拿到很单一的结果`BlogTitlesOnlyDto`，谁知道未知场景下程序员要什么结果，其实我们也可以返回一个泛型对象，只要稍加改动

```
public interface BlogRepository extends CrudRepository<Blog,Integer> {
   <T> List<T> findAllByTitle(String title, Class<T> clazz); 
}
```
这样你依旧可以用

```
  List<Blog> blogs = repository.findAllByTitle(Blog.class);
```

## 参考命名规则


|    关键字     |   示例    |   JPQL片段    |
|------------|-----------------------|----------------------------|
|And   |findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
|Or  |findByLastnameAndFirstname   | … where x.lastname = ?1 or x.firstname = ?2 |
|Is,Equals|findByFirstname,findByFirstnameIs,findByFirstnameEquals   | … where x.firstname = ?1 |
|Between  |findByStartDateBetween   | … where x.startDate between ?1 and ?2 |
|LessThan  |findByAgeLessThan   | … where x.age < ?1 |
|LessThanEqual  |findByAgeLessThanEqual   |… where x.age <= ?1|
|GreaterThan  |findByAgeGreaterThan   | … where x.age > ?1 |
|GreaterThanEqual  |findByAgeGreaterThanEqual   |… where x.age >= ?1|
|After  |findByStartDateAfter   | … where x.age >= ?1 |
|Before  |findByStartDateBefore   |… where x.startDate < ?1 |
|IsNull  |findByAgeIsNull   | … where x.age is null |
|IsNotNull,NotNull  |findByAge(Is)NotNull   | … where x.age not null|
|Like  |findByFirstnameLike   | … where x.firstname like ?1 |
|NotLike |findByFirstnameNotLike   | … where x.firstname not like ?1 |
|StartingWith|findByFirstnameStartingWith | … where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith  |findByFirstnameEndingWith   | … where x.firstname like ?1 (parameter bound with prepended %) |
|Containing  |findByFirstnameContaining   |… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy  |findByAgeOrderByLastnameDesc   | … where x.age = ?1 order by x.lastname desc |
|Not  |findByLastnameNot   | … where x.lastname <> ?1 |
|In  |findByAgeIn(Collection<Age> ages)   | … where x.age in ?1|
|NotIn  |findByAgeNotIn(Collection<Age> age)   | … where x.age not in ?1 |
|True  |findByActiveTrue()   | … where x.active = true |
|False  |findByActiveFalse()   |… where x.active = false |
|IgnoreCase  |findByFirstnameIgnoreCase()   |… where UPPER(x.firstame) = UPPER(?1)|