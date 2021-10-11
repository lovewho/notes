# Mybatis

## 1. MyBatis-Spring-Boot-Starter

### [1.1. Introduction](https://mybatis.org/mybatis-3/)

#### [1.1.1. What is MyBatis-Spring-Boot-Starter?](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)

MyBatis-Spring-Boot-Starter 帮助您在 Spring Boot 之上快速构建 MyBatis 应用程序。

通过使用此模块，您将实现：

- 构建独立的应用程序
- 将样板配置减少为零
- 更少的XML配置

#### 1.1.2. Requirements

| MyBatis-Spring-Boot-Starter | MyBatis-Spring                            | Spring Boot   | Java        |
| :-------------------------- | :---------------------------------------- | :------------ | :---------- |
| **2.2**                     | 2.0 (need 2.0.6+ for enable all features) | 2.5 or higher | 8 or higher |
| **2.1**                     | 2.0 (need 2.0.6+ for enable all features) | 2.1 - 2.4     | 8 or higher |
| **1.3**                     | 1.3                                       | 1.5           | 6 or higher |

### 1.2. Installation

要使用 MyBatis-Spring-Boot-Starter 模块，您只需在classpath下添加`mybatis-spring-boot-autoconfigure.jar` 及其依赖项（`mybatis.jar`、`mybatis-spring.jar` 等...... ) 。

**Maven**

如果你使用Maven，您只需在`pom.xml`中添加如下依赖：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

**Gradle**

使用Gradle则添加如下内容到`build.gradle`

```groovy
dependencies {
  compile("org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0")
}
```

### 1.3. Quick Setup

您可能已经知道，要将 MyBatis 与 Spring 一起使用，您至少需要一个 `SqlSessionFactory` 和至少一个`Mapper`接口。

 MyBatis-Spring-Boot-Starter 将：

- 自动检测现有`DataSource`.
- 将创建和注册 `SqlSessionFactory` 的实例，使用 `SqlSessionFactoryBean` 将该`DataSource`作为输入传递.
- 将创建并注册一个从` SqlSessionFactory` 中得到的 `SqlSessionTemplate` 的实例.
- 自动扫描`Mapper`，将它们链接到 `SqlSessionTemplate` 并将它们注册到 Spring 上下文，以便它们可以注入到您的 bean 中.

假设我们有如下`Mapper`:

```java
@Mapper
public interface CityMapper {

  @Select("SELECT * FROM CITY WHERE state = #{state}")
  City findByState(@Param("state") String state);

}
```

您只需要创建一个普通的 Spring 启动应用程序并让`Mapper`按如下方式注入（在 Spring 4.3+ 上可用）：

```java
@SpringBootApplication
public class SampleMybatisApplication implements CommandLineRunner {

  private final CityMapper cityMapper;

  public SampleMybatisApplication(CityMapper cityMapper) {
    this.cityMapper = cityMapper;
  }

  public static void main(String[] args) {
    SpringApplication.run(SampleMybatisApplication.class, args);
  }

  @Override
  public void run(String... args) throws Exception {
    System.out.println(this.cityMapper.findByState("CA"));
  }

}
```

这就是您所要做的。 您的应用程序现在可以作为普通的 Spring Boot 应用程序运行.

## 3. FQA
