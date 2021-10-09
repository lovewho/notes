# Jackson in N minutes

## 1. Introduction

虽然 Jackson 最初是使用在 JSON 数据绑定，但它现在也可以用于读取其他数据格式的编码的内容，只要存在 Parser(解析器) 和 Generator(生成器) 的实现，类的命名在很多地方都使用了“JSON”这个词，尽管实际上并没有对 JSON 格式的硬依赖。

Jackson 数据绑定的核心类是 ObjectMapper，ObjectMapper 是 Jackson 提供的用于支持 JSON 读/写的一个类，它支持 JSON 与 POJOs/JsonNode(通用的 JSON 树模型) 之间的序列化和反序列化，以及执行相关转换的功能。并且它是线程安全的，可以作为静态单例重复使用。

## 2. :clock4:1 minute tutorial: POJOs to JSON and back

最常用的用法就是提取一段 JSON。创建一个简单的对象，如下：

```java
// 注意: 也可以使用 getters/setters，这里我们直接使用 public 字段
public class Person {
  public String name;
  public int age;
  // 注意: 如果使用 getters/setters，您可以使字段为`protected` or `private`
}
```

我们需要一个 `com.fasterxml.jackson.databind.ObjectMapper` 实例，用于所有的数据绑定，现在让我们来构造它：

```java
// The instance is thread safe
ObjectMapper mapper = new ObjectMapper(); // create once, reuse
```

现在让我们来简单使用它吧，读取一段 JSON，用法很简单：

```java
Person person = mapper.readValue(new File("person.json"), Person.class);
//or
person = mapper.readValue(new URL("file:///D:/project/Java/java-util-series/jackson-starter/src/test/resources/person.json"), Person.class);
//or
person = mapper.readValue("{\"name\":\"Bob\", \"age\":13}", Person.class);
```

如果我们想写 JSON，我们应该做相反的事情：

```java
mapper.writeValue(new File("result.json"), myResultObject);
// or:
byte[] jsonBytes = mapper.writeValueAsBytes(myResultObject);
// or:
String jsonString = mapper.writeValueAsString(myResultObject);
```

## 3. :clock4:3 minute tutorial: Generic collection, Tree Model

上一节我们展示了如何处理简单的 POJOs，这一节我们将展示如何处理 List、Map 集合类型。

```java
// to list
Map<String, Integer> scoreByName = mapper.readValue(jsonSource, Map.class);
// to map
List<String> names = mapper.readValue(jsonSource, List.class);	
// 当然我们也可以写入
mapper.writeValue(new File("names.json"), names);
```

只要 JSON 结构匹配，并且简单，您不需要额外做任何工作。但如果您的集合有 POJO 的值，并且您想避免类型擦除，则需要指明实际类型。

```java
// to list
List<Person> person = mapper.readValue(jsonSource, new TypeReference<List<Person>>() {});
// to map
Map<String, Person> results = mapper.readValue(jsonSource, new TypeReference<Map<String, Person>>() {});

// 序列化不需要做额外的工作，无论泛型如何
mapper.writeValueAsString(person);
mapper.writeValueAsString(results);
```

虽然处理 Maps、Lists 和其他“简单”的对象类型（String、Numbers、Boolean）可能很简单，但是对于一些复杂对象，可能处理起来比较麻烦(毕竟不可能所有JSON都有对应的Java结构)，这时 Jackson 提供的 [Tree model](https://github.com/FasterXML/jackson-databind/wiki/JacksonTreeModel) 可以派上用场。

```java
// 可以读为泛化 JsonNode，如果您已知它的类型，如 Object，您可以将其转换为 ObjectNode，如 Array，您可以将其转换为 ArrayNode 等等
JsonNode root = mapper.readTree("{\"name\":\"jackin\", \"age\":22}");
String name = root.get("name").asText();
int age = root.get("age").asInt();

// 转为 ObjectNode
ObjectNode node = (ObjectNode)root;

// 您也可以修改，如增加一个子对象作为属性，并且为子对象设置属性 "type"
node.with("other").put("type", "student");;
String json = mapper.writeValueAsString(node);

// 最后我们的结构就是这样的 json 串
// {
//   "name" : "Bob", "age" : 13,
//   "other" : {
//      "type" : "student"
//   }
// }
```

Tree Model 比数据绑定更加方便，特别是在结构高度动态或不能很好地映射到 Java 类的情况下。

## 4. :clock4:5 minute tutorial: Streaming parser, generator

还有一种更规范的处理模型可用，与数据绑定（to/from POJO）一样方便，和 Tree Model 一样灵活：增量模型(又叫流 `streaming` 模型)。它是数据绑定和 Tree Model 的底层处理模型，但是它也能暴露给想要最终性能和/或控制解析或生成细节的用户。

如需深入了解，请查看 [Jackson Core component](https://github.com/FasterXML/jackson-core)，在这里我们将让您有个简单了解：

```java
ObjectMapper mapper = new ObjectMapper();
// First: 编写简单的 JSON 输出
File jsonFile = new File("src/test/resources/test.json");
// note: 该方法是 Jackson 2.11 新增的，更早的版本需要使用 mapper.getFactory().createGenerator(...)
JsonGenerator generator = mapper.createGenerator(jsonFile, JsonEncoding.UTF8);
// 写入 JSON: { "message" : "Hello world!" }
generator.writeStartObject();
generator.writeStringField("message", "Hello world!");
generator.writeEndObject();
generator.close();

// Second: 读取 JSON 文件
try (JsonParser parser = mapper.createParser(jsonFile)) {
    // 默认使用 UTF8StreamJsonParser 进行解析
    // 这里是 JsonToken.START_OBJECT
    JsonToken t = parser.nextToken();
    // JsonToken.FIELD_NAME
    t = parser.nextToken();
    if ((t != JsonToken.FIELD_NAME) || !"message".equals(parser.getCurrentName())) {
        // handle error
    }

    //JsonToken.VALUE_STRING
    t = parser.nextToken();
    if (t != JsonToken.VALUE_STRING) {
        // handle error
    }
    String msg = parser.getText();
    assertThat(msg).isEqualTo("Hello world!");
}
```

:white_check_mark: Parser(解析) 就是写入的反过程，使用 JsonToken 对应着解析写入内容的每一步的操作类型，如写入时第一步是 `generator.writeStartObject()`，那么对应解析时第一次 JsonToken 的值是 `JsonToken.START_OBJECT`；第二步写入的是字段名，字段值，在解析JsonToken对应的值就是 `JsonToken.FIELD_NAME、JsonToken.VALUE_STRING`。反过来说，JsonToken 的取值定义了我们写入时可以做的操作。

## 5. :clock4:10 minute tutorial: configuration

这里是您可能会使用的两种入门级配置机制：[Features](https://github.com/FasterXML/jackson-databind/wiki/JacksonFeatures) and [Annotations](https://github.com/FasterXML/jackson-annotations)。

**Commonly used Features**

以下是您最有可能需要了解的配置功能示例。

让我们从更高级别的数据绑定配置开始吧：

```java
// SerializationFeature 用于改变 JSON 的编写方式
// 启用标准缩进(更美观的打印)
mapper.enable(SerializationFeature.INDENT_OUTPUT);
// 允许序列化 “empty” 的 POJOs对象(没有属性的对象)。没有这个设置将抛出异常
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
// 将 java.util.Date, Calendar 输出为时间戳
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
//DeserializationFeature 用于更改 JSON 作为 POJO 读取方式
// 允许解析未知的属性，否则异常抛出
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
// 将 JSON 空字符串 ("") 强制转换为 null 对象值
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
```

您还可能会更改属性名称的解析和生成规则：

```java
// 使用 kebab-case 对属性名解析和生成
mapper.setPropertyNamingStrategy(PropertyNamingStrategy.KEBAB_CASE);
// 使用 snake_case 对属性名解析和生成
mapper.setPropertyNamingStrategy(PropertyNamingStrategy.KEBAB_CASE);

```

除此之外，您可能需要更改一些低级 JSON 解析、生成细节：

```java
// ---- JsonParser.Feature 用于配置解析规则
// 允许在 JSON 中使用 C/C++ 风格的注释（非标准，默认禁用）
// note: 从 Jackson2.5 开始，您也可以使用 mapper.enable(feature) / mapper.disable(feature) 进行设置
mapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
// 允许 JSON 中使用(非校准)不带引号的字段名称
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
// 允许 JSON 中使用单引号的字段名称
apper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
// ---- JsonGenerator.Feature 用于配置生成规则
// 强制转义非 ASCII 字符
mapper.configure(JsonGenerator.Feature.ESCAPE_NON_ASCII, true);
```

完整的配置请参阅 [Jackson Features](https://github.com/FasterXML/jackson-databind/wiki/JacksonFeatures)！

**Annotations: changing property names**

最简单的基于注解的方式是使用`@JsonProperty`更改属性的名称:

```java
public class MyBean {
   private String _name;

   // 没有注解，我们得到的将是 "theName", 但我们想要的是 "name":
   @JsonProperty("name")
   public String getTheName() { return _name; }

   // 注意：我们只需在getter 或 setter 或 field 上添加注解就足够了；
   // 所以我们可以在这里省略它
   public void setTheName(String n) { _name = n; }
}
```

**Annotations: Ignoring properties**

我们可以使用 `@JsonIgnore` 和 `@JsonIgnoreProperties` 用于忽略属性(还有其他更多方式参见 [4. Annotation Module)](#4. Annotation Module)：

```java
// 意味着如果 JSON 中如果有 "foo" 和 "bar" 属性，它们将被忽略， 不管POJO有没有这样的属性
@JsonIgnoreProperties({ "foo", "bar" })
public class MyBean
{
   // 不会被写入 JSON；也不会从 JSON 中读取值
   @JsonIgnore
   public String internal;

   // 没有被标记注解，将正常写入和读取 JSON
   public String external;

   @JsonIgnore
   public void setCode(int c) { _code = c; }

   // note: 将被忽略，因为setter 标记了注解
   public int getCode() { return _code; }
}
```

与重命名(`@JsonProperty`)一样，请注意：注解在字段、getter 和 setter 之间“共享”，即如果有一个具有 @JsonIgnore，它会影响其他。 但也可以 “拆分” 注解，例如：

```java
public class ReadButDontWriteProps {
   private String _name;
   @JsonProperty public void setName(String n) { _name = n; }
   @JsonIgnore public String getName() { return _name; }
}
```

在这个案例中，“name” 属性将不会输出(因为 ‘getter’ 使用 `@JsonIgnore` 忽略)，但是，如果读取 JSON 时找到 “name” 属性，“name” 属性将被赋值。

有关序列化 JSON 时忽略属性的方法更完整的说明请参阅： ["Filtering properties"](http://www.cowtowncoder.com/blog/archives/2011/02/entry_443.html) 文章。

**Annotations: using custom constructor**

和其他大多数数据绑定包不同，Jackson 不需要您定义“默认构造函数”（无参构造函数）。 虽然它会在没有其他可用的情况下使用一个，但您可以轻松定义使用有参构造函数：

```java
public class CtorBean
{
  public final String name;
  public final int age;

  // 构造函数可以是公共的，私有的，任何
  @JsonCreator 
  private CtorBean(@JsonProperty("name") String name, @JsonProperty("age") int age){
      this.name = name;
      this.age = age;
  }
}
```

构造函数在支持使用 [Immutable objects](http://www.cowtowncoder.com/blog/archives/2010/08/entry_409.html)（即创建以后不希望被改变的对象，不对外公开 setter方法，只能通过构造进行初始化） 方面特别有用。

或者，你也可以定义工厂方法：

```java
public class FactoryBean
{
    // 省略了字段

    @JsonCreator
    public static FactoryBean create(@JsonProperty("name") String name) {
      // construct and return an instance
    }
}
```

请注意，使用 “creator method(创建者方法)”（带有 `@JsonProperty` 标记参数的 `@JsonCreator`）并不排除使用 setter：您可以将构造函数/工厂方法中的属性与通过 setter 或直接通过字段设置的属性混合和匹配。

## 6. Tutorial: fancier stuff, conversions

:white_check_mark: fancier stuff: 更高级的东西。

另一个有用的特性是Jackson 能够进行任意的 POJO-to-POJO 的转换。从概念上来说，您可以把转换过程视为两步：第一步：将 POJO 输出为 JSON，第二步：将 JSON 绑定到另一个 POJO。实现只是跳过 JSON 的实际生成，并使用更有效的中间表示。

任何兼容类型之间的转换都可以工作，调用非常简单：

```java
ResultType result = mapper.convertValue(sourceObject, ResultType.class);
```

并且只要源和结果类型兼容——也就是说，只要 to-JSON、from-JSON 成功 —— 事情就会“正常工作”。 但这里有几个可能有用的用例：

```java
// 将 List<Integer> 转换为 int[]
List<Integer> sourceList = ...;
int[] ints = mapper.convertValue(sourceList, int[].class);
// 将 POJO 转为 Map
Map<String,Object> propertyMap = mapper.convertValue(pojoValue, Map.class);
// 转回
PojoType pojo = mapper.convertValue(propertyMap, PojoType.class);
// 对 Base64 进行解码（默认使用 byte[] 表示 base64 字符串）
String base64 = "TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5IGhpcyByZWFzb24sIGJ1dCBieSB0aGlz";
byte[] binary = mapper.convertValue(base64, byte[].class);
```

基本上，Jackson 可以替代许多 Apache Commons 组件，用于 base64 编码/解码等任务，以及处理 “dyna bean”（映射到/来 POJO）。

## 7. Tutorial: Builder design pattern + Jackson

Builder 设计模式是一种创建者设计模式，能够用于逐步创建复杂对象。如果我们有一个复杂对象需要对其他依赖项进行多次检查，在这种情况下，最好使用构建器设计模式。

当我们使用 Builder 模式创建对象时，我们的对象一般是没有 getters/setters 方法的，且 field、构造也是 private。正常情况是无法读\写 JSON的。

让我们看看Jackson如何和 Builder 设计模式一起使用吧，构建一个 Person 结构， 包函如下可选的字段：

```java
public class Person {
    private final String name;
    private final Integer age;
 
    // getters
}
```

让我们看看如何在反序列化中利用它的能量，首先，让我们定义一个 private 的全参构造和一个 Builder 类：

```java
private Person(String name, Integer age) {
    this.name = name;
    this.age = age;
}
 
static class Builder {
    String name;
    Integer age;
    
    Builder withName(String name) {
        this.name = name;
        return this;
    }
    
    Builder withAge(Integer age) {
        this.age = age;
        return this;
    }
    
    public Person build() {
        return new Person(name, age);
    } 
}
```

然后，我们需要使用 `@JsonDeserialize` 标记我们的类(在这里是 Person)，传递 Builder 类的完全限定域名作为参数。最后，使用 `@JsonPOJOBuilder` 标记 Builder 类。

```java
@JsonDeserialize(builder = Person.Builder.class)
public class Person {
    //...
    
    @JsonPOJOBuilder
    static class Builder {
        //...
    }
}
```

一个简单的测试如下：

```java
String json = "{\"name\":\"Hassan\",\"age\":23}";
Person person = new ObjectMapper().readValue(json, Person.class);
 
assertEquals("Hassan", person.getName());
assertEquals(23, person.getAge().intValue());
```

如果您的实现的 Builder 设计模式使用的是其他前缀名称而不是 with 或使用了其他名称作为构建方法的名称而不是 build，Jackson 也为您提供了解决方法。

假设您实现的 Builder 设计模式使用 “set” 作为方法前缀，使用 “create” 代替 build，那么您可以使用如下的方式解决这个问题：

```java
// 使用 buildMethodName 指定您的构建方法的名称，使用 withPrefix 指定方法前缀
@JsonPOJOBuilder(buildMethodName = "create", withPrefix = "set")
static class Builder {
    String name;
    Integer age;
    
    Builder setName(String name) {
        this.name = name;
        return this;
    }
    
    Builder setAge(Integer age) {
        this.age = age;
        return this;
    }
    
    public Person create() {
        return new Person(name, age);
    } 
}
```

总的来说，Jackson 库在使用构建器模式反序列化对象方面非常强大。

