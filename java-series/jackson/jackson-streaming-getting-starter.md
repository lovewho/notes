# Streaming-Module-getting-starter

## 1. Introduction

Straming 项目包含了 Jackson 数据处理使用的核心底层增量(流式)解析器和生成器的抽象。它也包含了用于处理 JSON 格式的解析器和生成器的默认实现。这个核心抽象并不是特定于 JSON 的，由于历史原因，尽管命名确实许多地方存在 JSON，但是只有专门包含 “json” 单词的包是专门针对 JSON的。这个包是  [Jackson data-binding](https://github.com/FasterXML/jackson-databind) 构建的基础。

另外其他的数据格式实现（如 Smile（二进制 JSON）、XML、CSV、Protobuf 和 CBOR）也建立在这个基础包之上，实现了核心接口，使得无论底层数据格式如何，都可以使用标准数据绑定包。

Jackson 将数据抽象为 Stream-of-Events(事件流)，让我们看看 Jackson 如何使应用程序通过 Stream-of-Events 抽象来处理 JSON 内容。

## 2. Installation

要使用 Streaming 的功能，您需要引入 `com.fasterxml.jackson.core` 依赖。

**Maven**

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>x.x.x</version>
</dependency>
```

**Gradle**

```groovy
dependencies {
  compile("com.fasterxml.jackson.core:jackson-core:x.x.x")
}
```

## 3. Reading from Stream-of-Events

由于 Stream-of-Events(事件流) 只是一个逻辑上的抽象，并不是一个具体的东西，首先要决定的是如何实现它。有多种做法，这里有常用的三种方式：

- 作为可迭代的事件对象流。这是 **Stax Event API** 使用的方式，好处包括访问的简单性和允许在处理过程中保留事件对象的对象封装。
- 作为事件发生时的回调，将所有数据作为回调参数传递。 这是 SAX API 使用的方法。它是高性能和类型安全的（每个回调方法，每个事件类型一个，可以有不同的参数），但从应用程序的角度来看，使用起来可能很麻烦。
- 作为一个逻辑游标，每次访问对应一个事件的具体数据：这是 Stax Cursor API 采用的方法，这种方式比起第一种(事件对象流)的方式主要优势是性能(类似于回调方法)：框架没有构造额外的对象，如果应用程序有任何需要可以自己创建对象。与第二种(callback)相比最主要的好处是应用程序访问简单了：不需要注册回调处理程序，没有 **Hollywood 原则**(不要找我，我们找你)，只是使用游标对事件进行简单的迭代。

Jackson 使用第三种方式，实现了一个叫 `JsonParser` 的对象作为逻辑游标，这个选择是基于便利性和效率两方面组合的(其他选择将提供其中之一，但不能同时提供两者)， 用作游标的对象被命名为 “parser(解析器，而不是叫 reader 啥的)”，是为了与 JSON 规范更紧密结合(毕竟 reader 听起来就像读取文件的对象)，并且 Jackson 的其他 API 也遵循 JSON 规范，如将结构化 key/value 字段称为 Object(对象)，一组连续的值称为 Array(数组)。

为了迭代这个流，应用程序通过调用 `JsonParser.nextToken()` 来推进游标(Jackson 更愿意叫 token 而不是 event)，并且访问 `token` 指向的数据和属性。

所以基本的想法很简单。但是为了更好地了解细节，让我们举一个例子，如下是一段简单的 JSON:

```json
{
  "id":1125687077,
  "text":"@stroughtonsmith You need to add a \"Favourites\" tab to TC/iPhone. Like what TwitterFon did. I can't WAIT for your Twitter App!! :) Any ETA?",
  "fromUserId":855523, 
  "toUserId":815309,
  "languageCode":"en"
}
```

为了包函从这个 JSON 内容解析的数据，我们定义一个简单的 Bean：

```java
public class TwitterEntry
{
  private Long _id;  
  private String text;
  private Integer fromUserId;
  private Integer toUserId;
  private String languageCode;

  public TwitterEntry() { }
  // 省略 getters/setters...
  public String toString() {
    return "[Tweet, id: "+id+", text='";+text+"', from: "+fromUserId+", to: "+toUserId+", lang: "+languageCode+"]";
  }
}
```

接下来，让我们尝试读取上述的示例数据到 TwitterEntry 对象吧。

首先，这是一个可以通过事件流读取 Json 内容并填充 bean 的方法：

```java
TwitterEntry read(JsonParser jp) throws IOException {
  // Sanity check: verify that we got "Json Object":
  if (jp.nextToken() != JsonToken.START_OBJECT) {
    throw new IOException("Expected data to start with an Object");
  }
  TwitterEntry result = new TwitterEntry();
  // Iterate over object fields:
  while (jp.nextToken() != JsonToken.END_OBJECT) {
   String fieldName = jp.getCurrentName();
   // Let's move to value
   jp.nextToken();
   if (fieldName.equals("id")) {
    result.setId(jp.getLongValue());
   } else if (fieldName.equals("text")) {
    result.setText(jp.getText());
   } else if (fieldName.equals("fromUserId")) {
    result.setFromUserId(jp.getIntValue());
   } else if (fieldName.equals("toUserId")) {
    result.setToUserId(jp.getIntValue());
   } else if (fieldName.equals("languageCode")) {
    result.setLanguageCode(jp.getText());
   } else { // ignore, or signal error?
    throw new IOException("Unrecognized field '"+fieldName+"'");
   }
  }
  jp.close(); // important to close both parser and underlying File reader
  return result;
 }
```

然后我们通过如下方式来使用它：

```java
// 首先我们需要创建一个可重复使用、线程安全的 JsonFactory
//第一种：2.10+
JsonFactory factory = JsonFactory.builder()
	// 配置，如果您需要的话
     .enable(JsonReadFeature.ALLOW_JAVA_COMMENTS)
     .build();

//第二种：older 2.x
JsonFactory factory = new JsonFactory();
factory.enable(JsonReadFeature.ALLOW_JAVA_COMMENTS);

//第三种：您也可以使用 `ObjectMapper` 进行创建，如果您使用了 Jackson Databind package
JsonFactory factory = objectMapper.getFactory();

//读取：我们需要使用 JsonParser(或者它的子类，如果是 JSON 以外的数据格式)，其实例由 JsonFactory 构造。
 JsonParser jp = factory.createJsonParser(new File("input.json"));
 TwitterEntry entry = read(jp);
```

好的，现在这是一个相对简单的操作但相当多的代码，好的方面来说，遵循它很简单：即使您从未使用过 Jackons 或 json 格式，也应该很容易掌握正在发生的事情并根据需要修改代码。所以基本上它是“猴子代码”——易于阅读、编写、修改，但乏味、乏味并且以自己的方式容易出错（因为无聊）。

另一个可能更重要的好处是，这实际上非常快：开销很小，如果您费心对其进行基准测试，它确实运行得很快。最后，处理是完全流式处理的：解析器（和生成器也是如此）只跟踪逻辑游标当前指向的数据（以及一些用于嵌套、输入行号等的上下文信息）。

上面的示例提示了使用“原始”流访问 Json 的可能用例：性能真正重要的地方。另一种情况可能是内容结构非常不规则，更自动化的方法不起作用或者数据和对象的结构具有高阻抗。

## 4. Writing to Stream-of-Events

使用 Stream-of-Events 读取内容是一个简单但费力的过程，写入内容大致也是如此，尽管可能少了一点不必要的工作。鉴于我们现在有一个由 Json 内容构建的 Bean，我们不妨尝试将其写回（在中间进行修改之后）。

```java
  private void write(JsonGenerator jg, TwitterEntry entry) 抛出 IOException { 
    jg.writeStartObject(); 
    // 可以分别使用 "jg.writeFieldName(...) + jg.writeNumber()"，或者：
    jg.writeNumberField("id", entry.getId()); 
    jg.writeStringField("text", entry.getText()); 
    jg.writeNumberField("fromUserId", entry.getFromUserId()); 
    jg.writeNumberField("toUserId", entry.getToUserId()); 
    jg.writeStringField("langugeCode", entry.getLanguageCode()); 
    jg.writeEndObject(); 
    jg.close(); 
  }
```

这里是调用它的代码：

```java
//写入一个文件，使用 UTF-8 编码
JsonGenerator jg = factory.createJsonGenerator(new File("result.json"), JsonEncoding.UTF8);
jg.useDefaultPrettyPrinter(); // enable indentation just to make debug/testing easier
TwitterEntry entry = write(jg, entry);
```

## 5. **Conclusions**

所以从上面来看，使用基本的 Stream-of-Events 是处理 Json 内容的非常原始的方式。这会带来好处（非常快速、完全流式传输 [无需在内存中构建或保留对象层次结构] 很容易看到正在发生的事情）和缺点（代码冗长、重复）。

但不管你是否会使用这个 API，至少要知道它是如何工作的：这是因为其他接口建立在此基础上的：数据映射和树构建都在内部使用原始流 API 来读取和写入 Json 内容。