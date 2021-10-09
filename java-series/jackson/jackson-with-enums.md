# How To Serialize and Deserialize Enums

## 1. Introduction

本节我们将学习如何使用 Jackson 对枚举进行序列化和反序列化及定制它的表现形式。

## 2. Preparation

让我们准备一个枚举类，如下：

```java
public enum  Week {
    MONDAY(1, "Mon"),
    TUESDAY(2, "Tues"),
    WEDNESDAY(3, "Wed"),
    THURSDAY(4, "Thur"),
    FRIDAY(5, "Fri"),
    SATURDAY(6, "Saturday"),
    SUNDAY(0, "Sat");

    private int id;
    private String simpleName;

    Week(int id, String simpleName) {
        this.id = id;
        this.simpleName = simpleName;
    }
    //省略 getter
}
```

## 3. Serialize to JSON

### 3.1. Default Enum Representation
当我们我们使用 Jackson 对枚举进行序列化，默认的序列化输出为枚举的名称：

```java
String json = new ObjectMapper().writeValueAsString(Week.MONDAY);
// json:
"MONDAY"
```

假设我们想得到的输出不是枚举的名称，而是如下格式，很明显，此时我们需要定制枚举的表现形式，那该如何做？

```json
{"id":1,"simpleName":"Mon"}
```


### 3.2. Using @JsonFormat

Jackson 提供了 @JsonFormat 用于改变输出的形式，我们只需在类上标记该注解即可：

```java
// 表示以 json 对象格式输出
@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum  Week {
 //...
}
```

当执行序列化时，我们将得到如下输出：

```json
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(Week.MONDAY);
//json
{
  "id" : 1,
  "simpleName" : "Mon"
}
```

### 3.3. Using @JsonValue

另一种更改输出的形式的简单方法是使用 @JsonValue，被 @JsonValue 标记的方法或者字段将作为序列化结果输出。此时我们可以定制预期的格式输出：

```java
public enum  Week {
    //...
    @JsonValue
    public Map<String, Object>  toJson(){
        Map<String, Object> map = new LinkedHashMap<>();
        map.put("id", getId());
        map.put("simpleName", getSimpleName());
        return map;
    }
}
```

输出结果如下：

```java
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(Week.MONDAY);
//json: 
{
  "id" : 1,
  "simpleName" : "Mon"
}
```

### 3.4. Custom Serializer for Enum

我们也可以通过自定义序列化器来控制序列化结果的表现形式，如下：

```java
public class WeekStdSerializer extends StdSerializer<Week>  {

    public WeekStdSerializer() {
        super(Week.class);
    }

    public WeekStdSerializer(Class<Week> t) {
        super(t);
    }

    @Override
    public void serialize(Week value, JsonGenerator gen, SerializerProvider provider) throws IOException {
        gen.writeStartObject();
        gen.writeNumberField("id", value.getId());
        gen.writeStringField("simpleName", value.getSimpleName());
        gen.writeEndObject();
        gen.close();
    }

}
```

在 Week 类上添加 @JsonSerialize 来让自定义序列化器工作，如下：

```java
@JsonSerialize(using = WeekStdSerializer.class)
public enum  Week {
 //...
}
```

当执行序列化时将以预期结果输出：

```java
 String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(Week.MONDAY);
//json:
{
  "id" : 1,
  "simpleName" : "Mon"
}
```

## 4. Deserialize from JSON to Enum

定义一个 Item 类，其包含一个 Week 属性：

```java
public class Item {
    private Week week;
    //省略getter/setter
}
```

### 4.1. Default BeHavior

默认 Jackson 使用枚举名称进行反序列化：

```java
Item item = new ObjectMapper().readValue("{\"week\":\"MONDAY\"}", Item.class);
assertThat(item.getWeek()).isEqualTo(Week.MONDAY);
```

### 4.2. Using @JsonValue

我们也可以使用 @JsonValue 标记的序列化结果进行反序列化，这种方式只在枚举类型中有用，原因是：枚举类型的值是常量，可以确定唯一实例。

```java
public enum  Week {
    @JsonValue
    public Map<String, Object>  toJson(){
        Map<String, Object> map = new LinkedHashMap<>();
        map.put("id", getId());
        map.put("simpleName", getSimpleName());
        return map;
    }
}
```

当我们使用 @JsonValue 的结果进行反序列化时，需要把它的结果看做一个整体的字符串值，而不是 json 对象，简单来说就是要把 key-value 的`"` 去掉，并在最外层添加 `"`，如下：

```java
// 不能看做 json 对象
{"id" : 1, "simpleName" : "Mon"}
//而是要看做字符串整体
"{id:1,simpleName:Mon}"
  
Item item = new ObjectMapper().readValue("{\"week\":\"{id=1, simpleName=Mon}\"}", Item.class);
assertThat(item.getWeek()).isEqualTo(Week.MONDAY);
```

### 4.3. Using @JsonProperty

当反序列化名称和枚举实例的名称不一致时，@JsonProperty 是一种很好的解决方案：

```java
public enum  Week {
    @JsonProperty("Mon")
    MONDAY(1, "Mon"),
    //....
}

Item item = new ObjectMapper().readValue("{\"week\": \"Mon\"}", Item.class);
assertThat(item.getWeek()).isEqualTo(Week.MONDAY);
```

### 4.4. Using @JsonAlias

和 @JsonProperty 类似，都用于配置属性的名称，但区别在于 @JsonProperty 是替换了属性的名称，而 @JsonAlias 是给属性附加了一个别名，即原属性名称依然生效：

```java
public enum  Week {
    @JsonAlias("Mon")
    MONDAY(1, "Mon"),
    //....
}

Item item = new ObjectMapper().readValue("{\"week\": \"Mon\"}", Item.class);
//or
Item item = new ObjectMapper().readValue("{\"week\":\"MONDAY\"}", Item.class);
assertThat(item.getWeek()).isEqualTo(Week.MONDAY);
```

### 4.5. Using @JsonCreator

@JsonCreator 通常用于声明在构造方法或者工厂方法上，它会被 Jackson 调用用于创建类的实例，不管构造或者工厂方法是否是私有。

但是枚举是不能被构造的比构造私有更严格，但我们可以通过工厂方法的方式，而由于属性没有 setter 方法，所以我们必须在方法参数上使用 @JsonProperty 标识通过参数注入。

```java
public enum  Week {
	//...
    
   @JsonCreator
    public static Week create(@JsonProperty("id") int id, @JsonProperty("simpleName") String simpleName){
        for(Week week : Week.values()){
            if (Objects.equals(week.getId(), id) && Objects.equals(week.getSimpleName(), simpleName)){
                return week;
            }
        }
        return null;
    }
}
```

此时当我们以下 json 进行反序列化时我们可以得到正确的结果：

```java
// json: {"week": {"id":1,"simpleName":"Mon"}}
Item item = new ObjectMapper().readValue("{\"week\":{\"id\":1,\"simpleName\":\"Mon\"}}", Item.class);
assertThat(item.getWeek()).isEqualTo(Week.MONDAY);
```

### 4.6. Custom Serializer for Enum

我们也可以通过自定义反序列化器来控制生成实例的规则：

```java
public class WeekStdDeserializer extends StdDeserializer<Week> {

    public WeekStdDeserializer() {
        super(Week.class);
    }

    public WeekStdDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Week deserialize(JsonParser p, DeserializationContext ctx) throws IOException, JsonProcessingException {
        JsonNode node = p.getCodec().readTree(p);
        int id = node.get("id").asInt();
        String simpleName = node.get("simpleName").asText();
        for(Week week : Week.values()){
            if (Objects.equals(week.getId(), id) && Objects.equals(week.getSimpleName(), simpleName)){
                return week;
            }
        }
        return null;
    }
}
```

为了让自定义的反序列化器能够正常的工作，我们需要在 Week 类上添加 @JsonDeserialize 注解，如下：

```java
@JsonDeserialize(using = WeekStdDeserializer.class)
public enum  Week {
//....
}
```

让我们执行反序列化吧：

```java
Item item = new ObjectMapper().readValue("{\"week\":{\"id\":2,\"simpleName\":\"Tues\"}}", Item.class);
assertThat(item.getWeek()).isEqualTo(Week.TUESDAY);
```

## 5. Conclusion

在 Jackson 中有多种方式可以对枚举的序列化和反序列化的过程进行控制，您可以根据您的业务需求选择最适合的。



 