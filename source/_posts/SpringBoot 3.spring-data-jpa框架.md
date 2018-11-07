---
title: SpringBoot 3.spring-data-jpa框架
date: 2018-06-05 10:40:58
tags: [springboot,jpa]
categories: springboot
---

# spring-data-jpa的使用

## spring-data-jpa介绍

### JPA是什么？

JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。
Sun引入新的JPA ORM规范出于两个原因：其一，简化现有Java EE和Java SE应用开发工作；其二，Sun希望整合ORM技术，实现天下归一。

PA的总体思想和现有Hibernate、TopLink、JDO等ORM框架大体一致。总的来说，JPA包括以下3方面的技术：

1. ORM映射元数据
	JPA支持XML和JDK5.0注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中；
2. API
	用来操作实体对象，执行CRUD操作，框架在后台替代我们完成所有的事情，开发者从繁琐的JDBC和SQL代码中解脱出来。
3. 查询语言
	这是持久化操作中很重要的一个方面，通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。

<!-- more -->

### spring-data-jpa

Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

## spring-data-jpa使用

### 配置

添加pom依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

application.properties添加配置
```
#数据库链接
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

#spring data jpa 配置
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql= true
```

hbm2ddl.auto参数的作用主要用于：自动创建|更新|验证数据库表结构,有四个值：

> 1. create： 每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。
2. create-drop ：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
3. update：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据 model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等 应用第一次运行起来后才会。
4. validate ：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

dialect 主要是指定生成表名的存储引擎为InneoDB

show-sql 控制台打印sql，方便调试

### 使用

#### 数据库层代码

BaseEntity.java

```java
import javax.persistence.*;
import java.io.Serializable;
import java.util.Date;

//@MappedSuperclass注解（或mapped-superclass XML描述符元素）来指定映射超类。映射超类不会生成单独的表，它的映射信息作用于继承自它的实体类。
@MappedSuperclass
public class BaseEntity implements Serializable{
    private static final long serialVersionUID = -6845215436041679253L;

    @Id
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private Long id;

    @Column(name= "is_delete", nullable = false)
    private Integer isDelete = 0;

    <!-- 省略getset方法 -->
}

```

User.java

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Table;

@Entity
@Table(name="mars_user")
public class User extends BaseEntity{

    //账户
    @Column(name="name", nullable = false, unique = true)
    private String name;

    //邮箱
    @Column(name="email", nullable = false, unique = true)
    private String email;

    //加密密码
    @Column(name="password", nullable = false)
    private String password;

    <!-- 省略getset方法 -->
}

```

#### 测试

1.继承JpaRepository

```java
public interface UserRepository  extends JpaRepository<User, Long>{
}
```

2.使用JpaRepository默认方法

```java

@Autowired
private UserRepository userRepository;

@Test
public void testUser(){
	User user = new User();
	user.setName("abc");
	user.setEmail("abc@qq.com");
	user.setPassword(MD5Util.encrypt("mimahenjiandan"));

	userRepository.save(user);
	//userRepository.findAll();
	//userRepository.delete(user);
	//userRepository.count();
	//...等等
}
```

3.启动成功创建表与数据

![spring data jpa](/images/2018-06-05/springboot_jpa_1.png)

![spring data jpa](/images/2018-06-05/springboot_jpa_2.png)


### 扩展

#### 自定义查询

自定义的简单查询就是根据方法名来自动生成SQL，主要的语法是findXXBy,readAXXBy,queryXXBy,countXXBy, getXXBy后面跟属性名称：

```java
User findByName(String name);
```

也使用一些加一些关键字And、 Or

```java
User findByNameOrEmail(String name, String email);
```

修改、删除、统计也是类似语法

```java
Long deleteById(Long id);

Long countByName(String name)
```

基本上SQL体系中的关键词都可以使用，例如：LIKE、 IgnoreCase、 OrderBy。

```java
List<User> findByEmailLike(String email);

User findByNameIgnoreCase(String name);
    
List<User> findByNameOrderByEmailDesc(String email);
```

** 具体的关键字，使用方法和生产成SQL如下表所示 **

|Keyword	| Sample	|JPQL snippet|
| - | :-: | - | 
|And	|findByLastnameAndFirstname	|… where x.lastname = ?1 and x.firstname = ?2|
|Or	|findByLastnameOrFirstname	|… where x.lastname = ?1 or x.firstname = ?2|
|Is,Equals	|findByFirstnameIs,findByFirstnameEquals	|… where x.firstname = ?1|
|Between	|findByStartDateBetween	|… where x.startDate between ?1 and ?2|
|LessThan	|findByAgeLessThan	|… where x.age < ?1|
|LessThanEqual	|findByAgeLessThanEqual	|… where x.age ⇐ ?1|
|GreaterThan	|findByAgeGreaterThan	|… where x.age > ?1|
|GreaterThanEqual	|findByAgeGreaterThanEqual	|… where x.age >= ?1|
|After	|findByStartDateAfter	|… where x.startDate > ?1|
|Before	|findByStartDateBefore	|… where x.startDate < ?1|
|IsNull	|findByAgeIsNull	|… where x.age is null|
|IsNotNull,NotNull	|findByAge(Is)NotNull	|… where x.age not null|
|Like	|findByFirstnameLike	|… where x.firstname like ?1|
|NotLike	|findByFirstnameNotLike	|… where x.firstname not like ?1|
|StartingWith	|findByFirstnameStartingWith	|… where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith	|findByFirstnameEndingWith	|… where x.firstname like ?1 (parameter bound with prepended %)|
|Containing	|findByFirstnameContaining	|… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy	|findByAgeOrderByLastnameDesc	|… where x.age = ?1 order by x.lastname desc|
|Not	|findByLastnameNot	|… where x.lastname <> ?1|
|In	|findByAgeIn(Collection ages)	|… where x.age in ?1|
|NotIn	|findByAgeNotIn(Collection age)	|… where x.age not in ?1|
|TRUE	|findByActiveTrue()	|… where x.active = true|
|FALSE	|findByActiveFalse()	|… where x.active = false|
|IgnoreCase	|findByFirstnameIgnoreCase	|… where UPPER(x.firstame) = UPPER(?1)|

#### 分页查询

分页查询在实际使用中非常普遍了，spring data jpa已经帮我们实现了分页的功能，在查询的方法中，需要传入参数Pageable ,当查询中有多个参数的时候Pageable建议做为最后一个参数传入

> Pageable 是spring封装的分页实现类，使用的时候需要传入页数、每页条数和排序规则

```java
int page=1,size=10;
Sort sort = new Sort(Direction.DESC, "id");
Pageable pageable = new PageRequest(page, size, sort);

Page<User> findALL(Pageable pageable);
    
Page<User> findByName(String name,Pageable pageable);
```

#### 限制查询

有时候我们只需要查询前N个元素，或者只取前一个实体。

```java
ser findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

#### 自定义SQL查询

其实Spring data 觉大部分的SQL都可以根据方法名定义的方式来实现，但是由于某些原因我们想使用自定义的SQL来查询，spring data也是完美支持的；在SQL的查询方法上面使用@Query注解，如涉及到删除和修改在需要加上@Modifying.也可以根据需要添加 @Transactional 对事物的支持，查询超时的设置等

```java
@Modifying
@Query("update User u set u.name = ? where u.id = ?")
int modifyByIdAndUserId(String  name, Long id);
	
@Transactional
@Modifying
@Query("delete from User where id = ?")
void deleteByUserId(Long id);
  
@Transactional(timeout = 10)
@Query("select u from User u where u.email = ?")
User findByEmail(String email);
```