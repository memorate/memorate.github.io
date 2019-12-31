---
layout: post
title: Java-Spring Boot集成Spring Data JPA
tags:
- Java 
- Spring Boot
- Study
categories: Java
description: Spring Data JPA使用简介
---  
**Spring Boot集成Spring Data JPA实操步骤**

<!-- more -->
### 一、简介  
　　Spring Data JPA是 Spring 基于 ORM 框架、JPA 规范封装的一套 JPA 应用框架，
可使开发者用极简的代码即可实现对数据库的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！
（[官方文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)、[GitHub地址](https://github.com/spring-projects/spring-data-jpa)）  
### 二、使用  
#### 1.导入包
在pom.xml文件中引入下述依赖（版本可自主选择）。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.15</version>
</dependency>
```
#### 2.基本配置
在**resources**文件夹下创建`application.yml`文件并进行下述配置。  
`注`：application.yml与application.properties作用相同，但前者更加简洁、层次分明、更便于阅读，因此使用application.yml。  
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dataBaseName?serverTimezone=Shanghai&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: anchor#123
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      ddl-auto: update
    show-sql: true
```
`其中`：  
　1）"**url: jdbc:mysql://`localhost:3306`/`dataBaseName`?serverTimezone=Shanghai&useUnicode=true&characterEncoding=utf-8&useSSL=true**"     
　　 "localhost:3306"为数据库IP及端口（端口一般为3306），"dataBaseName"为要连接的数据库名称，其余不做修改；  
　2）"**username**"和"**password**"为数据库账号和密码（若忘记可[重置](https://blog.csdn.net/hero_hope/article/details/82868046)）；    
　3）"**driver-class-name**"：[说明](https://blog.csdn.net/superdangbo/article/details/78732700)；  
　4）"**database-platform**"：数据库引擎（[介绍](https://blog.csdn.net/ls5718/article/details/52248040)）；  
　5）"**ddl-auto**"：[属性介绍](https://blog.csdn.net/fengyuhan123/article/details/80264795)，一般使用update；  
　6）"**show-sql**"：是否在日志中打印真实操作的sql语句；   
#### 3.项目结构
　![]({{ "/assets/img/springJPA1.png"}})  
`四类约定文件`：  
　1)**model**：存放数据库中表对应的Java类；  
　2)**repository**：每个Repository接口负责对一张表进行CURD，是Java应用操作数据库的工具；  
　3)**service**：业务操作，普遍一个Service类中封装一个Repository的相关操作，对Repository查到的数据进行业务处理；  
　4)**controller**：与外界（请求）交互的门户，接收外界请求并调用service层的方法处理该请求；  
　创建类时请遵循上述约定将对应的类放在对应的package下！  
### 三、细解
　　　　　　　![]({{ "/assets/img/springJPA2.png"}})![]({{ "/assets/img/springJPA3.png"}})  
　　　　　　　　　　　　　　(users表)　　　　　　　　　　　　　　　　　　　　　(detail表)
#### 1.Model
默认情况下，创建实体类时注意**类名**要与数据库中表名相同，类中各**属性名**与表中字段名相同。（符合驼峰也可匹配）  
```java
@Entity                  /*用于标注该类对应数据库的表*/                          @Entity 
@Table(name = "users")   /*用于映射表名和实体类名*/                              public class Detail {
public class User {                                                                  @Id
    @Id                  /*用于标注该字段为User类中的主键*/                            @GeneratedValue
    @GeneratedValue      /*用于标注主键的生成策略*/                                    private long id;
    private long id;                                                                  private String position;
    private String name;                                                              private String phoneNumber;   //驼峰
    private int age;                                                                  @Column(name = "education")
    private String description;                                                       private String edu;  
}     /*省略getter、setter方法*/                                                 }       //省略getter、setter方法     
```
1）`@Entity`  
　用于告诉spring boot该类对应数据库的一张表。  
2）`@Table`  
　当实体类类名与其映射的数据库表名**不相同**时需要使用@Table注解指定实体类所对应的数据库表。  
　**用法**：@Table(name = "tableName")  
3）`@Column`  
　当类属性名与数据库字段名**不相同**时需要使用@Column注解指定属性所对应的字段。  
　**用法**：@Column(name = "columnName")  
4）`@GeneratedValue`  
　其意义主要是为一个实体生成一个唯一标识的主键，通过strategy属性指定生成方式，共有四种策略。    
　**用法**：@GeneratedValue(strategy=GenerationType.AUTO)  
　①IDENTITY：采用数据库ID自增长的方式来自增主键字段，Oracle 不支持这种方式；  
　②AUTO：JPA自动选择合适的策略，是默认选项（**不指定strategy时默认使用AUTO**）；  
　③SEQUENCE：通过数据库的序列产生主键，通过@SequenceGenerator注解指定序列名，MySql不支持这种方式；  
　④TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植；  
#### 2.Repository
　　Repository作为对数据库进行实际操作的类（接口），JPA已经提供了大量常用的CURD方法，封装在JpaRepository、PagingAndSortingRepository、
CrudRepository等接口中，一般情况继承JpaRepository<T,ID>即可使用所有现有方法。  
　　**JpaRepository<T,ID>**：其中T为数据库表对应的Java类，ID为Java类中主键的类型。  
```java
@Repository         //用于标注数据库访问组件（也可不写，Spring Boot也可正常工作）
public interface UsersRepository extends JpaRepository<User,Long> {    //用于操作数据库"users"表
}
```
　　**默认方法开箱即可使用，约定方法只需根据约定起好方法名，自定义方法需自己编写[HQL](https://blog.csdn.net/qq_28633249/article/details/77884062)/SQL语句。**  
##### 1）<span id="default">默认方法</span>  
以下方法为三个接口中自带，可直接使用。  
![]({{ "/assets/img/springJPA4.png"}})  
**a.新增和更新**  
　新增和更新单条记录都使用`S save(S var1)`方法。多条记录时使用重载方法`saveAll(Iterable<S> var1)`，注意两个方法返回值不同。   
　①当var1变量的主键为空时，执行新增操作；  
　②当主键不为空、但数据库中主键对应的数据不存在时，执行新增操作；  
　③当主键不为空、且数据库中主键对应的数据存在时，执行更新操作；  
　当使用了@GeneratedValue注解后，即使id赋了具体的值(例id=999)，save时还是会使用自增长的方式，而不会在数据库中将id的值存为999。  
**b.排序和分页**  
　重载方法findAll()可以传入Sort或实现Pageable的对象来进行分页或排序。  
　①排序  
　spring-data-commons-2.2.0.RELEASE.jar中不再提供Sort的public构造方法，改为使用Sort.by()来创建Sort实例。  
　Sort.Direction.ASC是升序，对应Sort.Direction.DESC是降序，"id"为Java类中的属性名，**注意不是数据库表中的字段名**。  
```java
public List<User> singleSort(){     //单属性排序
    return usersRepository.findAll(Sort.by(Sort.Direction.ASC, "id"));
}
```  
```java
public List<User> multiSort() {    //多属性排序
    Sort.Order idOrder = Sort.Order.asc("id");
    Sort.Order ageOrder = Sort.Order.desc("age");
    return usersRepository.findAll(Sort.by(idOrder, ageOrder));  //先按id升序再按age降序进行排序
}
```
　②分页  
　PageRequest是实现了Pageable接口的类，使用PageRequest.of()来创建PageRequest实例。  
　PageRequest.of()有三个重载方法，可按需选用。  
```java
public List<User> page(int pageNum, int pageSize){              //pageNum表示第几页，pageSize表示每页有几条数据
    Sort sort = Sort.by(Sort.Direction.ASC, "id");              //根据id进行排序
    PageRequest pageRequest = PageRequest.of(pageNum, pageSize, sort);
    Page<User> userPage = usersRepository.findAll(pageRequest);
    return userPage.getContent();                               //getContent()方法可获取查询结果，
}                                                               //实际情况下至少还需返回查询总数：userPage.getTotalElements()
```
**c.按例查询**  
　这里的"例"是Example，给repository传入一个Example对象，JPA会根据这个例子去查询符合条件的数据。  
　个人感觉按例查询比约定方法使用更为繁杂，不建议使用。想了解可自行google。  
##### 2）约定方法  
①Spring Data JPA框架可根据约定好的方法名自动生成相应的SQL语句，主要的语法是findXXBy后面跟属性名称。  
②由于Create和Update一般是操作一组具体数据，删除是根据主键，默认提供的方法已足够使用，因此约定方法基本用来Read。  
③[使用方法](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
，根据表格中关键字生成相应的查询方法。一般使用findXX，但也可用readXX、queryXX、getXX。
```java
@Repository
public interface UsersRepository extends JpaRepository<User,Long> {
    User findByNameAndAge(String name, int age);
    List<User> readByAgeLessThanEqual(int age);
    List<User> queryByNameStartingWith(String word);
    List<User> getByDescriptionIsNotNull();

    @Transactional
    @Modifying
    int deleteByName(String name);    

    List<User> findByAge(Sort sort);             //约定方法排序。方法名findByAge后加"BySort"会报错

    Page<User> findByAge(Pageable pageable);     //约定方法分页。方法名findByAge后加"ByPageable"会报错
}
```    
##### 3）自定义方法  
当上述两类方法都无法满足对数据库的操作需求时，可以使用**@Query**注解来自定义操作方法。（自定义查询语句）  
@Query可用于新增、更新、查询、删除操作。但不建议用于新增，[默认方法](#default)中新增更加便捷。  
当方法用于新增、更新、删除时，需再加注解@Modifying和@Transactional。  
**a.使用HQL**  
普通@Query是使用hql进行操作。hql语句中的操作对象是Java类名和类的属性名，而不是数据库表的表名和字段名。     
```java
public interface UsersRepository extends JpaRepository<User,Long> {
    @Query("SELECT u from User u where u.name = ?1 and u.age = ?2")       //"?1"代表第一个参数，"?2"代表第二个参数，以此类推。
    User findByNameAndAge(String name, int age);

    @Query("select u from User u where u.name = :myName and u.age = :myAge")         //":参数名"的方式指定方法中的参数
    User findByNameAndAge(@Param("myName") String name, @Param("myAge") int age);    //使用@Param注解注入参数
}
```
个人认为使用"?序号"的方式注入参数更加简洁。  
**b.使用SQL**  
在@Query注解中设置"nativeQuery = true"来使用原生sql。  
```java
public interface UsersRepository extends JpaRepository<User,Long> {
    @Query(value = "SELECT * from users where name= ?1", nativeQuery = true)   //value中为普通的sql语句
    User findByName(String name);
}     //也可使用@Param注解来注入参数，此处未写出
```
**c.排序和分页**  
```java
@Query("select u from User u where u.age = ?1")
List<User> findByAge(int age, Sort sort);           //自定义方法的排序

@Query("select u from User u where u.age = ?1")
Page<User> findByAge(int age, Pageable pageable);   //自定义方法的分页
```
**d.根据值长度排序**  
```java
public List<User> sortByLength(int age) {
    JpaSort sort = JpaSort.unsafe(Sort.Direction.ASC, "LENGTH(name)");     //LENGTH(属性名)。必须使用JpaSort.unsafe，使用Sort.by()报错
    return usersRepository.findByAge(age, sort);
}
```
`注意事项`：  
①当使用@Query注解时，约定方法会失效。如下findByName方法实际查询条件是@Query中的age，而非方法名中的ByName。  
　即***@Query注解优先级高于约定。***    
```java
@Query(value = "SELECT * from users where age= ?1", nativeQuery = true)
User findByName(int age);
```    
②若进行新增、更新、删除操作不添加**@Modifying**和**@Transactional**，会分别抛出如下异常。  
```text
java.sql.SQLException: Can not issue data manipulation statements with executeQuery().
```
```text
javax.persistence.TransactionRequiredException: Executing an update/delete query
```
③更新、删除操作方法的返回值只能是**void**或**int/Integer**，否则会抛出如下异常。  
```text
java.lang.IllegalArgumentException: Modifying queries can only use void or int/Integer as return type!
```
④只有自定义方法才可以根据值长度进行排序，默认方法和约定方法使用"LENGTH(属性名)"排序会报错。  
```text
org.springframework.data.mapping.PropertyReferenceException: No property LENGTH(description) found for type User!
```
⑤三种关于HQL的`错误`写法   
 ```java
 @Query("SELECT u from users u where u.name = ?1 and u.age = ?2")  //无法识别users。hql操作实体类，而不是数据库表（应是User类，而不是users表）
 User findByNameAndAge(String name, int age);
 
 @Query("SELECT * from User where name = ?1 and age = ?2")            //hql不支持"*"写法  
 User findByNameAndAge(String name, int age);
 
 @Query("SELECT User from User u where u.name = ?1 and u.age = ?2")   //启动时报一个空指针错误(It confuses me)
 User findByNameAndAge(String name, int age);
 ```
#### 3.Service
在Service层注入Repository后即可使用其中定义的各类方法。  
文末代码仓库中有使用示例。  
```java
@Component
public class UsersService {

    @Resource                                     //推荐使用@Resource，不推荐@Autowired(具体原因自行google)
    private UsersRepository usersRepository;

    /**
    * 使用默认方法 - 新增或更新一条数据
    */
    public User saveUser(User user) {
        return usersRepository.save(user);
    }

    //...
}
```
#### 4.Controller
常规使用  
```java
@RestController
@RequestMapping("/user")
public class UsersController {

    @Resource
    private UsersService usersService;

    @RequestMapping("/all")
    public List<User> getAllUsers() {
        return usersService.getAllByDefault();
    }

    //类似调用UsersService中方法即可
    //...
}
```
<br/>  
本文中使用的所有代码：[https://github.com/memorate/SpringBootJPA](https://github.com/memorate/SpringBootJPA)，此工程中所有代码皆可正常运行。






