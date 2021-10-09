# Jackson-Serialize

## 1. Introduction

本节我们将学习如何使用 Jackson 进行 JSON 序列化以及如何定制序列化规则。

## 2. Standard Serialization

首先，让我们准备两个简单的实体，来体验一下 Jackson 如何将其序列化为 JSON 。

```java
public class User {
    private Integer id;
    private String username;
    // 省略 getters/setters
}

public class Item {
    private Integer id;
    private String name;
    private User owner;
    // 省略 getters/setters
}
```

然后使用 ObjectMapper 对其进行序列化，ObjectMapper 是 Jackson 提供的一个便于操作序列化(写入)和反序列化(读取)的类。

```java
Item item = new Item(1, "item01", new User(100, "zs"));
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(item);
// json:
{
  "id" : 1,
  "name" : "item01",
  "owner" : {
    "id" : 100,
    "username" : "zs"
  }
}
```

## 3. Custom Serialization

Jackson 允许我们自定义序列化器来改变它的序列化规则，只需简单几步。假设序列化 Item 的时候，我们希望它的 owner 属性显示的是用户名称，而不是整个 User 对象，像这样：

```json
{
  "id" : 1,
  "name" : "item01",
  "owner" : "zs"
}
```

此时我们可以通过自定义序列化器来解决这个问题。

### 3.1. Custom Serializer on the ObjectMapper

Jackson 提供了 `StdSerializer` 类用于用户自定义序列化器。我们通过继承它并实现 `serialize`方法来定义自己的序列化规则：

```java
public class ItemStdSerializer extends StdSerializer<Item> {

    public ItemStdSerializer(){
        this(null);
    }

    public ItemStdSerializer(Class<Item> t) {
        super(t);
    }

    @Override
    public void serialize(Item value, JsonGenerator gen, SerializerProvider provider) throws IOException {
        gen.writeStartObject();
        gen.writeNumberField("id", value.getId());
        gen.writeStringField("name", value.getName());
        gen.writeStringField("owner", value.getOwner().getUsername());
        gen.writeEndObject();
    }
}
```

接下来，我们只需通过 ObjectMapper 向 Jackson 注册我们的自定义处理器即可，当 Jackson 序列化遇到 `Item` 类型时，就会将其交给我们自定义的序列化器 ItemStdSerializer 进行处理：

```java
Item item = new Item(1, "item01", new User(100, "zs"));

SimpleModule simpleModule = new SimpleModule();
// 添加自定义序列化器，将 Item 和 ItemStdSerializer 相关联
// 相当于告诉 Jackson，序列化遇到 Item 类型时交给 ItemStdSerializer 处理。
simpleModule.addSerializer(Item.class, new ItemStdSerializer());
// 向 ObjectMapper 注册模块
String json = new ObjectMapper().registerModule(simpleModule).writerWithDefaultPrettyPrinter().writeValueAsString(item);
// json:
{
  "id" : 1,
  "name" : "item01",
  "owner" : "zs"
}
```

### 3.2. Custom Serializer on the Class

上述我们自定义序列化器是通过 ObjectMapper 注册的，它在整个 ObjectMapper 范围都有效，我们也可以直接在 Class 上面使用 `@JsonSerialize` 注解进行注册: 

```java
@JsonSerialize(using = ItemStdSerializer.class)
public class Item {
   //....
}
```

当执行序列化时，Jackson 会自动调用自定义的序列化器 `ItemStdSerializer`对 `Item` 进行序列化：

```java
Item item = new Item(1, "item01", new User(100, "zs"));
String json = new ObjectMapper().writerWithDefaultPrettyPrinter().writeValueAsString(item);
//json:
{
  "id" : 1,
  "name" : "item01",
  "owner" : "zs"
}
```

有一点需要注意：`@JsonSerialize` 还可以在 Field（字段）和 method（方法，通常是 getter/setter）parameter（参数，通常是 create/build）上使用，但此时仅对当前标记的位置有效。

如下，我们将 `Item`作为 `Product` 的属性，在该属性上标记了`@JsonSerialize` 而不是在 `Item` 类上：

```java
public class Product {
    private int pid;
    @JsonSerialize(using = ItemStdSerializer.class)
    private Item item2;
}

public class Item {
    //...省略
}
```

分别执行对 `Product` 和 `Item` 的序列化，可以看到在序列化 `Product` 时，其中的 `Item` 属性使用了自定义序列化器，而单独对 Item 进行序列化时，并没有应用自定义序列化器。

```java
ObjectWriter mapper = new ObjectMapper().writerWithDefaultPrettyPrinter();
Item item = new Item(1, "item2", new User(100, "zs"));
Product product = new Product(100, item);

String productJson = mapper.writeValueAsString(product);
// productJson
{
    "pid" : 100,
    "item2" : {
        "id" : 1,
        "name" : "item2",
        "owner" : "zs"
    }
}
String item2Json = mapper.writeValueAsString(item);
//item2Json
{
  "id" : 1,
  "name" : "item2",
  "owner" : {
    "id" : 100,
    "username" : "zs"
  }
}
```

### 3.3. Custom Serialization Using BeanSerializerModifier

我们也可以通过继承 `BeanSerializerModifier` 来定制序列化规则，这是一个更加底层的类。

```java
public class ItemBeanSerializerModifier extends BeanSerializerModifier {
    @Override
    public List<BeanPropertyWriter> changeProperties(SerializationConfig config, BeanDescription beanDesc, List<BeanPropertyWriter> beanProperties) {
        //是不是对 Item 进行序列化
        if(Item.class.isAssignableFrom(beanDesc.getBeanClass())){
            for(int i = 0; i < beanProperties.size(); i++){
                BeanPropertyWriter writer = beanProperties.get(i);
                //修改 owner 属性的输出格式
                if("owner".equals(writer.getName())){
                    beanProperties.set(i, new UserBeanPropertyWriter(writer));
                }
            }
        }
        return beanProperties;
    }
    
    static class UserBeanPropertyWriter extends BeanPropertyWriter{
        private BeanPropertyWriter writer;

        public UserBeanPropertyWriter(BeanPropertyWriter base) {
            super(base);
            this.writer = base;
        }

        @Override
        public void serializeAsField(Object bean, JsonGenerator gen, SerializerProvider prov) throws Exception {
             
            //将 owner 属性的值修改为 username
            gen.writeStringField("owner", item.getOwner().getUsername());
        }
    }
}
```

想让 `ItemBeanSerializerModifier` 工作，我们同样需要向 Jackson 注册它：

```java
Item item = new Item(1, "item2", new User(100, "zs"));
// 方式一
ObjectMapper mapper = mapper.registerModule(new SimpleModule().setSerializerModifier(new ItemBeanSerializerModifier()));
// or
// 方式二
ObjectMapper mapper = new ObjectMapper()
    .setSerializerFactory(BeanSerializerFactory.instance.withSerializerModifier(new ItemBeanSerializerModifier()));
String json = mapper.writeValueAsString(item);
//json: 
{
  "id" : 1,
  "name" : "item2",
  "owner" : "zs"
}
```

## 4. Conclusion

本节我们学习了如何使用 Jackson 进行序列化及如何自定义序列化规则，其中通过 `BeanSerializerModifier` 定制序列化是一种更底层的用法，它的功能非常强大，其提供了多个方法以便于在 Jackson 序列化的各个阶段进行应用，值得进一步学习！

​	



