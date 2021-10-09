# Type Infomation

由于运行时类型擦除机制，使得我们在做泛型相关的反序列化无法获取到对应的类型信息，这一节我们将学习如何使用 Jackson 解决类型擦除问题。

## 1. Preparation

预先定义以下实体。

```java
public class Item {
    private Integer id;
    private String name;
    //... 省略 getter/setter/construct
}
```

## 2. Type erasure

首先我们定义了如下的 json，包含了一组 Item 元素：

```json
[
	{
		"id" : 1,
    	"name" : "zs"
	},
	{
		"id" : 2,
    	"name" : "lisi"
	}
]
```

提取这段json，将其反序列化为 List 集合：

```java
String json = "[{ \"id\" : 1, \"name\" : \"zs\",},{ \"id\" : 2, \"name\" : \"lisi\" }]";
List list = new ObjectMapper().readValue(json, List.class);
```

如您所见，由于运行时类型信息擦除，我们得到是一个原生的 List 集合，尽管我们知道它持有的是 Item 的信息，但是我们不得不将其元素显示转换后才能作为 Item 使用！我们希望得到的是一个带有类型信息的 List 集合。

## 3. Using JavaType

JavaType 是 Jackson 提供的用于构建类型信息的基类，其下包含了多种子类实现，分别用于构建不同的数据类型信息，如：

- ArrayType: 对应数组类型信息构建
- CollectionType：集合类型信息构建
- MapType: Map 类型信息构建

Jackson 提供了一个名为 TypeFactory 的类用于快速构建 JavaType, 如构建 List 集合类型信息:

```java
// 参数一：集合的类型 -> List
// 参数二：集合中元素的类型 -> Item
CollectionType collectionType = new ObjectMapper().getTypeFactory().constructCollectionType(List.class, Item.class);
```

构建 Map 类型信息：

```java
// 参数一：Map 的类型
// 参数二：Key 的类型
// 参数三：Value 的类型
MapType mapType = new ObjectMapper().getTypeFactory().constructMapType(HashMap.class, String.class, Item.class);
```

让我们重新提取上个案例中的 json，并使用 JavaType 指定它的类型信息，此时我们可以得到一个带有类型信息的 List 集合：

```java
ObjectMapper mapper = new ObjectMapper();
CollectionType type = mapper.getTypeFactory().constructCollectionType(List.class, Item.class);
// 使用 CollectionType
List<Item> items = mapper.readValue(json, type);
assertThat(items.get(0)).extracting("id").isEqualTo(1);
```

## 4. Using TypeReference

另一种解决类型信息擦除有效的方法就是使用 TypeReference，TypeReference 是一个不包含抽象方法的抽象性类，通常我们直接通过匿名内部类的方式创建它，它有一个泛型参数用于指定我们的类型信息。

构建 List 类型信息，并指定泛型参数类型为 `List<Item>`：

```java
TypeReference<List<Item>> typeReference = new TypeReference<List<Item>>() {}
```

构建 Map 类型信息：

```java
TypeReference<List<Item>> typeReference = new TypeReference<new HashMap<String, Item>>() {}
```

使用 TypeReference 提取 json:

```java
List<Item> items = new ObjectMapper().readValue(json, new TypeReference<List<Item>>() {});
assertThat(items.get(0)).extracting("id").isEqualTo(1);
```

## 5. Conclusion

本节我们学习了如何使用 JavaType 和 TypeReference 解决 Jackson 中的类型擦除问题。相比于 JavaType 方案 TypeReference 看起来是一种更直观和简便的方式。







