# How to filter properties

## 1. Introduction

本节我们将学习如何使用 Jackson 对属性进行过滤以及如何自定义过滤策略。

## 2. Preparation

让我们准备如下两个实体：

```java
public class Student {
    private Integer sid;
    private String name;
    private Course course;

    public Student(){}

    public Student(Integer sid, String name, Course course) {
        this.sid = sid;
        this.name = name;
        this.course = course;
    }
    //...ommit setters/getters
}

public class Course {
   private Integer cid;
   private String cname;
   
   public Course(){}
    
   public Course(Integer cid, String cname) {
       this.cid = cid;
       this.cname = cname;
   }
   //...ommit setters/getters
}
```

## 3. Using @JsonIgnore

`@JsonIgnore` 是 Jackson 提供的用于忽略属性的注解，被它标识属性在序列化和反序列化时都将被过滤。该注解可声明在 field/getter/setter 上，它们之间是共享的。假设我们想要忽略 Course 的 cname 属性：

```java
public class Course {
   private Integer cid;
   @JsonIgnore
   private String cname;
   //...
}
```

当执行序列化操作时将不再输出 cname 属性：

```java
Course course = new Course(1, "C");
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(course);
assertThat(json).doesNotContain("cname");
//json:
{
  "cid" : 1
}
```

同样当我们执行反序列化时，sname 属性也不会被赋值：

```java
// json:
{
   "cid" : 1,
   "cname" : "C"
}

Course student = new ObjectMapper().readValue(json, Course.class);
assertThat(student.getCname()).isNullOrEmpty();
```

## 4. Using @JsonIgnoreProperties

`@JsonIgnoreProperties` 与 `@JsonIgnore` 类似，但它提供了更加丰富的功能。

### 4.1. Ignore multiple properties

允许指定多个要忽略的属性，如同时忽略 *cname，cid* 属性：

```java
@JsonIgnoreProperties({"cid", "cname"})
public class Course {
   //...
}
```

分别测试序列化和反序列化：

```java
//测试序列化
Course course = new Course(1, "C");
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(course);
assertThat(json).doesNotContain("cid", "cname");
//json:
{}

//测试反序列化
//json:
{
   "cid" : 1,
   "cname" : "C"
}
Course student = new ObjectMapper().readValue(json, Course.class);
assertThat(student).hasAllNullFieldsOrPropertiesExcept("cid", "cname");
```

### 4.2. Ignore unknown properties

反序列化时允许忽略未知的属性。默认当反序列化时存在未知的属性时，Jackson 将抛出异常，如下：

```java
//json
{
   "cid" : 1,
   "cname" : "C",
   "teacher": "Tony"
}

Course student = new ObjectMapper().readValue(json, Course.class);
//com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "teacher" (class com.jackin.pojo.Course), not marked as ignorable (0 known properties: ]) at [Source: (String)"{"cid" : 1,"cname" : "C", "teacher":"Tony"}"; line: 1, column: 38] (through reference chain: com.jackin.pojo.Course["teacher"])
```

此时我们可以设置 `@JsonIgnoreProperties` 的 *ignoreUnknown* 属性为 true 来解决这个问题：

```java
@JsonIgnoreProperties(value = {"cid", "cname"}, ignoreUnknown = true)
public class Course {
//....
}
```

### 4.3. Allow Getter or Setter

默认 `@JsonIgnoreProperties` 忽略的属性会同时影响序列化和反序列化，可以通过设置 *allowGetters* 和 *allowSetters* 来指定影响的范围。

*allowGetters* 表示不忽略属性 getter 方法，即在序列化时不受影响正常输出：

```java
@JsonIgnoreProperties(value = {"cid", "cname"}, ignoreUnknown = true, allowGetters = true)
public class Course {
//....
}

Course course = new Course(1, "C");
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(course);
//json:
{
   "cid" : 1,
   "cname" : "C"
}
```

*allowSetters* 表示不忽略属性 setter 方法，即可以正常反序列化属性。

## 5. Using @JsonIgnoreType

`@JsonIgnoreType` 用于声明在类上，表示忽略该类的所有字段。

```java
@JsonIgnoreType
public class Course {
   //...
}
```

## 6. Using @JsonFilter

`@JsonFilter` 是一种更高级的用法，它提供了更加灵活的过滤功能，允许我们在执行**序列化操作**时再指定想要过滤的属性，且允许自定义复杂的过滤策略。

**NOTE: `@JsonFilter` 并不会影响反序列化！**

### 6.1. Using it on Class

它的用法很简单。首先在需要过滤的类上声明该注解并指定它的名称，名称用于与具体的过滤器相关联。

```java
@JsonFilter("stuFilter")
public class Student {
  //...
}
```

然后创建过滤器指定过滤策略，并将过滤器和注解相关联：

```java
// First: 创建过滤器指定过滤策略，默认提供以下三种过滤策略：
// 1.保留所有属性
SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter.serializeAll();
// 2.过滤指定属性
filter = SimpleBeanPropertyFilter.serializeAllExcept(...);
// 3.保留指定的属性
filter = SimpleBeanPropertyFilter.filterOutAllExcept(...);

// Second: 将过滤器和注解相关联
SimpleFilterProvider filterProvider = new SimpleFilterProvider().addFilter("stuFilter", filter).addFilter(...)...;

```

最后在序列化时应用指定的 *SimpleFilterProvider* 即可：

```java
new ObjectMapper().writer(filterProvider)..writeValueAsString(...);
```

如下是一个过滤 `Student`的 *sid* 属性的完整案例：

```java
Student student = new Student(10, "jackin", new Course(1, "C"));
SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter.serializeAllExcept("sid");
SimpleFilterProvider filterProvider = new SimpleFilterProvider().addFilter("stuFilter", filter);
String json = new ObjectMapper().writer(filterProvider).withDefaultPrettyPrinter().writeValueAsString(student);
//json:
{
  "name" : "jackin",
  "course" : {
    "cid" : 1,
    "cname" : "C"
  }
}
```

### 6.2. Using it on Field

如果在类上声明了 `@JsonFilter ` 注解，那么所有引用到该类的地方都会生效。有时我们想做的仅仅只是针对该类的字段进行过滤，此时我们可以直接在需要过滤的字段上声明该注解。如针对 course 属性进行过滤：

```java
public class Student {
  	//...
  	@JsonFilter("stu.courseFilter")
  	private Course course;
    //...
}
```

然后和之前一样使用它即可。如下将过滤 Course 的 cname 属性，但仅对于 Student 生效：

```java
Student student = new Student(10, "jackin", new Course(1, "C"));
SimpleFilterProvider filterProvider = new SimpleFilterProvider().addFilter("stu.courseFilter", SimpleBeanPropertyFilter.serializeAllExcept("cname")); 			
String json = new ObjectMapper().writer(filterProvider).withDefaultPrettyPrinter().writeValueAsString(student);
//json: 
{
  "sid" : 10,
  "name" : "jackin",
  "course" : {
    "cid" : 1
  }
}
```

有一点需要注意：如果实体声明了多个 `@JsonFilter`，您必须在使用时将其逐个关联，否则 Jackson 将抛出异常。如：

```java
@JsonFilter("filter1")
public class Student {
  	@JsonFilter("filter2")
  	private Course course;
    //...
}

SimpleFilterProvider filterProvider = new SimpleFilterProvider().addFilter("filter1", filter1)
    .addFilter("filter1", filter2); 			
```

### 6.3. Custom Filter

前面我们使用的 `SimpleBeanPropertyFilter.serializeAllExcept...`等过滤策略其实由 Jackson 内置过滤器 `SerializeExceptFilter` 和 `FilterExceptFilter`提供的。

我们也可以通过继承 `SimpleBeanPropertyFilter` 来定制自己的过滤器，通过重写 `include` 或 `serializeAsField` 方法来实现过滤逻辑。

#### 6.3.1. Overwrite include method

通过重写 `include` 方法来实现过滤逻辑，返回 true 表示该属性可以被序列化输出。如定制一个前缀过滤器，只有当属性包含指定前缀时才能被序列化输出：

```java
public class PrefixFilter extends SimpleBeanPropertyFilter implements Serializable {

    private static final long serialVersionUID = 1L;
    private final String prefix;

    public PrefixFilter(String prefix) {
        this.prefix = prefix;
    }

    @Override
    protected boolean include(BeanPropertyWriter writer) {
        return writer.getName().startsWith(prefix);
    }

    @Override
    protected boolean include(PropertyWriter writer) {
        return writer.getName().startsWith(prefix);
    }
}
```

应用自定义的过滤器，如只输出前缀为 *"s"* 的属性：

```java
@JsonFilter("stuFilter")
public class Student {
    //...
}

Student student = new Student(10, "jackin", new Course(1, "C"));
SimpleFilterProvider filterProvider = new SimpleFilterProvider().addFilter("stuFilter", new PrefixFilter("s"));
String json = new ObjectMapper().writer(filterProvider).withDefaultPrettyPrinter().writeValueAsString(student);
//json:
{
  "sid" : 10
}

```

#### 6.3.2. Overwrite serializeAsField method

我们也可以通过重写 `serializeAsField` 方法实现过滤逻辑，该方法可以获取更多参数信息：

```java
public class PrefixFilter extends SimpleBeanPropertyFilter implements Serializable {

    private static final long serialVersionUID = 1L;
    private final String prefix;

    public PrefixFilter(String prefix) {
        this.prefix = prefix;
    }

    @Override
    public void serializeAsField(Object pojo, JsonGenerator gen, SerializerProvider provider, PropertyWriter writer) throws Exception {
        if(writer.getName().startsWith(prefix)){
            writer.serializeAsField(pojo, gen, provider);
        }
    }
}
```

## 7. Custom Serialize and Deserialize

我们也可以通过定制序列化和反序列来实现属性的过滤，请参考如下文章：

- [Jackson serialize and custom serializer (Jackson 序列化及自定义序列化器)](./jackson-serialize.md)
- [Jackson deserialize and custom deserializer (Jackson 反序列化及自定义反序列化器)](./jackson-deserialize.md)













