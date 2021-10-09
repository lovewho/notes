# Jackson-Deserialize

## 1. Introduction

本节将学习如何使用 Jackson 进行 JSON 反序列化以及如何对反序列化规则进行定制。

## 2. Standard Deserialization

最常的用法就是提取一段 JSON，让我们看看 Jackson 如何将其呈现到实体中。首先准备一段 JSON:

*student.json*

```json
{
    "stuNo":"10001",
    "name":"jackin",
    "gender":"male",
    "course":{
        "id":"cs001",
        "name":"Data Structure"
    }
}
```

接下来定义我们想要用于呈现 JSON 的实体，类似于这样：

```json
public class Student {
    private String stuNo;
    private String name;
    private String gender;
    private Course course;
    // 省略 getters/setters
}

public class Course {
    private String id;
    private String name;
    // 省略 getters/setters
}
```

最后，我们使用 Jackson 将 JSON 反序列化至对应的实体：

```java
Student student = new ObjectMapper()
                .readValue(new File("src/test/resources/student.json"), Student.class);
assertThat(student.getStuNo()).isEqualTo("10001");
```

## 3. Custom Deserialization

### 3.1. Using StdDeserializer on ObjectMapper

Jackson 允许我们对反序列化规则进行定制，只需要简单几步。这在很多情况下都非常有用，如 JSON 与实体结构不能完美匹配时。假定我们的 JSON 结构是这样：

*student-not-match.json*

```json
{
    "id":"10001",
    "name":"jackin",
    "gender":"male"
}
```

当我们将其反序列化至 `Student` 实体时，默认它将抛出一个异常。如下：

```
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "id" (class com.jackin.pojo.Student), not marked as ignorable (4 known properties: "gender", "name", "stuNo", "course"])
 at [Source: (File); line: 2, column: 9] (through reference chain: com.jackin.pojo.Student["id"])
```

此时我们可以通过自定义反序列化来解决这个问题。首先我们需要继承 `StdDeserializer`：

```java
public class StudentDeserializer extends StdDeserializer<Student> {

    public StudentDeserializer() {
        this(null);
    }

    public StudentDeserializer(Class<Student> t) {
        super(t);
    }

    @Override
    public Student deserialize(JsonParser p, DeserializationContext ctx) throws IOException, JsonProcessingException {
        Student student = new Student();
        ObjectMapper mapper = (ObjectMapper)p.getCodec();
        JsonNode tree = mapper.readTree(p);
        student.setStuNo(tree.get("id").asText());
        student.setName(tree.get("name").asText());
        student.setGender(tree.get("gender").asText());
        return student;
    }
    
}
```

在 `StudentDeserializer` 中我们使用了 `ObjectMapper` 将 JSON 转换为 `JsonNode`，这样我们可以很便捷的提取信息并构建我们自己的实体。当然如果您想的话，也完全可以基于 `Streaming(jackson-core)` 读取信息并构建实体，类似于：

```java
public Student deserialize(JsonParser p, DeserializationContext ctx) throws IOException {
    // note: 游标已经打开，不是 JsonToken.START_OBJECT
    Student student = new Student();
    // 移动游标
    while (p.nextToken() != JsonToken.END_OBJECT){
        // 拿到字段名称
        String fieldName = p.getCurrentName();
        // 移动游标
        p.nextToken();
        //p.getText() 拿到字段值
        if("id".equals(fieldName)){
            student.setStuNo(p.getText());
        }else if("name".equals(fieldName)){
            student.setName(p.getText());
        }else if("gender".equals(fieldName)){
            student.setGender(p.getText());
        }else{
            throw new IOException("Unrecognized field '"+ fieldName +"'");
        }
    }
    return student;
}
```

接下来，我们只需注册自定义的反序列化器并使用它即可，Jackson 提供了一个名叫 Module 的功能来支持用户为其附加功能：

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule();
module.addDeserializer(Student.class, new StudentDeserializer());
mapper.registerModule(module);

Student student = mapper
    .readValue(new File("src/test/resources/student-not-match.json"), Student.class);
assertThat(student.getStuNo()).isEqualTo("10001");
```

### 3.2. Using StdDeserializer on the Class

上述在 ObjectMapper 上注册了自定义反序列化器，它在整个 ObjectMapper 范围都有效，当然我们也可以在 Class 上注册，它的作用范围仅仅限制于该类，Jackson 提供了 `@JsonDeserialize` 来实现该点。

```java
public class Item{
    private String id;
    @JsonDeserialize(using=StudentDeserializer.class)
    private Student student;
}
```

当使用 `@JsonDeserialize` 注解时，我们不再需要手动注册反序列化器，在反序列化时 Jackson 会自动找到它：

```java
Student student = new ObjectMapper().readValue(new File("src/test/resources/student-not-match.json"), Student.class);
assertThat(student.getStuNo()).isEqualTo("10001");
```

:white_check_mark: Note: Jackson 是怎么处理自定义反序列化的？

简单说，当注册反序列化器时，Jackson 使用 Map<ClassKey, JsonDeserializer<?>> 来存储类型和自定义的序列化器的关系，ClassKey 就是我们处理的类的信息(上面就是 Student.class)，当进行反序列化的时候，Jackson 会先去 Map 中查找是否注册了自定义的反序列化，如果有就使用自定义的，没有就使用默认的。

其实关键点在与注册时 Class 和自定义反序列化的关联，只要关联了 Jackson 就会使用自定义的，也就是我们可以把不同的实体都关联到同一个自定义的反序列化器，这在多态的情况下特别有用，如下：

```java
public class MyStudent extend Student{
}

module.addDeserializer(MyStudent.class, new StudentDeserializer());
//或
public class Item{
    private String id;
    @JsonDeserialize(using=StudentDeserializer.class)
    private MyStudent mystu;
}
```

此时在反序列化 MyStudent 时也会使用自定义的 StudentDeserializer.

```java
Student student = new ObjectMapper().readValue(new File("src/test/resources/student-not-match.json"), Student.MyStudent.class);
assertThat(student.getStuNo()).isEqualTo("10001");
```

### 3.3. Using BeanDeserializerModifier on ObjectMapper

我们也可以通过继承 `BeanDeserializerModifier` 来定制反序列化规则。

```java
public class CustomBeanDeserializerModifier extends BeanDeserializerModifier {

    @Override
    public List<BeanPropertyDefinition> updateProperties(DeserializationConfig config, BeanDescription beanDesc, List<BeanPropertyDefinition> propDefs) {
        if(Student.class.isAssignableFrom(beanDesc.getBeanClass())){
            for (int i = 0; i < propDefs.size(); i++) {
                BeanPropertyDefinition propDef = propDefs.get(i);
                // 将 stuNo 重命名为 id
                if("stuNo".equals(propDef.getName())){
                    propDefs.set(i, propDef.withSimpleName("id"));
                }
            }
        }
        return propDefs;
    }
}
```

然后我们需要向 ObjectMapper 注册它：

```java
 ObjectMapper mapper = new ObjectMapper();
 SimpleModule module = new SimpleModule();
 module.setDeserializerModifier(new CustomBeanDeserializerModifier());
 mapper.registerModule(module);
 Student student = mapper
 .readValue(new File("src/test/resources/student-not-match.json"), Student.class);
 assertThat(student.getStuNo()).isEqualTo("10001");
```

## 4. Conclusion

本节我们主要介绍了 Jackson 的反序列化，包括如何使用其读取标准 JSON 输入以及如何自定义反序列化器读取非标准 JSON 输入。

参考博文：

- [jackson: ObjectMapper.registerModule()](https://codingdict.com/sources/java/com.fasterxml.jackson.databind/11414.html)
- [Getting Started with Custom Deserialization in Jackson](https://www.baeldung.com/jackson-deserialization)



