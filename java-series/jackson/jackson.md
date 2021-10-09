# [Jackson](https://github.com/FasterXML/jackson)-Series

## 1. Introduction

> Jackson 被称为 “Java JSON 库” 或 “Java 的最佳 JSON 解析器”。 但是其实它的功能不仅仅如此，更重要的是，Jackson 是一套适用于Java（和JVM 平台）的数据处理工具，包括标志性的流式 JSON 解析器/生成器库，匹配数据绑定库（POJO 与 JSON 的转换）和额外的数据格式模块以处理 [Avro](https://github.com/FasterXML/jackson-dataformats-binary/blob/master/avro), [BSON](https://github.com/michel-kraemer/bson4jackson), [CBOR](https://github.com/FasterXML/jackson-dataformats-binary/blob/master/cbor), [CSV](https://github.com/FasterXML/jackson-dataformats-text/blob/master/csv), [Smile](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/smile), [(Java) Properties](https://github.com/FasterXML/jackson-dataformats-text/blob/master/properties), [Protobuf](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/protobuf), [TOML](https://github.com/FasterXML/jackson-dataformats-text/blob/2.13/toml), [XML](https://github.com/FasterXML/jackson-dataformat-xml) or [YAML](https://github.com/FasterXML/jackson-dataformats-text/blob/master/yaml) 编码的数据，甚至有大量的数据模块用于支持被广泛使用的数据类型，例如 [Guava](https://github.com/FasterXML/jackson-datatypes-collections), [Joda](https://github.com/FasterXML/jackson-datatype-joda), [PCollections](https://github.com/FasterXML/jackson-datatypes-collections) 等等。

### 1.1. Core Modules

Jackson 包函如下三个核心模块，核心模块是其他扩展模块的构建基础：

- [Streaming](https://github.com/FasterXML/jackson-core)([docs](https://github.com/FasterXML/jackson-core/wiki)): ("jackson-core") 定义较底层的流处理 API，包括了特定的JSON实现。

- [Annotations](https://github.com/FasterXML/jackson-annotations)([docs](https://github.com/FasterXML/jackson-annotations/wiki)): ("jackson-annotations") 包函标准的 Jackson 注解 ([JAX-RS provider](https://github.com/FasterXML/jackson-jaxrs-providers))。
- [Databind](https://github.com/FasterXML/jackson-databind)([docs](https://github.com/FasterXML/jackson-databind/wiki)): ("jackson-databind") 基于 `Streaming` 实现了数据绑定和对象序列化的支持，它依赖于 `Streaming` 和 `Annotations`。默认主要为 JSON 提供实现，其它格式由额外的数据格式模块(**见1.4节**)支持。

### 1.2. [Thrid-party datatype modules](https://github.com/FasterXML/jackson)

Jackson 以插件方式提供了第三方模块的拓展(通过 ObjectMapper.registerModule())，并且添加各种常用数据类型的 Java 库支持。并且增加了序列化与反序列化支持以便于 Jackson databind 包(`ObjectMapper`、`ObjectReader`、`ObejctWriter`) 能够读取和写入这些数据类型。

Jackson 团队直接维护的数据类型模块支持分为以下几类。

- **标准的集合数据类型模块**
  - [jackson-datatype-eclipse-collections](https://github.com/FasterXML/jackson-datatypes-collections/tree/master/eclipse-collections): 支持 [Eclipse Collections](https://www.eclipse.org/collections/) (新增在Jackson 2.10!)。
  - [jackson-datatype-guava](https://github.com/FasterXML/jackson-datatypes-collections/tree/master/guava): 支持多种 [Guava](http://code.google.com/p/guava-libraries/) 数据类型。
  - [jackson-datatype-hppc](https://github.com/FasterXML/jackson-datatypes-collections/tree/master/hppc): 支持 [High-Performance Primitive Containers](http://labs.carrotsearch.com/hppc.html) 容器。
  - [jackson-datatype-pcollections](https://github.com/FasterXML/jackson-datatypes-collections/tree/master/pcollections): 支持 [PCollections](http://pcollections.org/) 数据类型 (since Jackson 2.7)。
- [Hibernate](https://github.com/FasterXML/jackson-datatype-hibernate): 支持 Hibernate 功能（lazy-loading 延迟加载, proxies 代理）
- [Java 8 Modules](https://github.com/FasterXML/jackson-modules-java8): 通过三个单独的模块支持 JDK8 功能 和数据类型。
  - jackson-module-parameter-names: 添加对 JDK8 新特性的支持，能够访问构造函数和方法参数的名称，以允许省略 @JsonProperty。
  - jackson-datatype-jsr310: 支持 `Java8 Dates`。
  - jackson-datatype-jdk8: 支持 JDK8 数据类型不仅是 date/time 类型，也包括 `Optional`。
- Joda 数据类型
  - [jackson-datatype-joda](https://github.com/FasterXML/jackson-datatype-joda): 支持 [Joda-Time](https://www.joda.org/joda-time/)库的日期和时间数据类型。
  - [jackson-datatype-joda-money](https://github.com/FasterXML/jackson-datatypes-misc/tree/master/joda-money): 支持[Joda-Money](https://www.joda.org/joda-money/)数据类型(`Money`, `CurrencyUnit`)。
- JSON-P ("json processing"): “旧”（javax.json）和 “新”（jakarta.json）的两个数据类型模块：
  - [jackson-datatype-jakarta-jsonp](https://github.com/FasterXML/jackson-datatypes-misc/jakarta-jsonp): ：支持 jakarta.json 中的“新” JSON-P 类型（在 Jackson 2.12.2 中添加）。
  - [jackson-datatype-jsr353](https://github.com/FasterXML/jackson-datatypes-misc/jsr-353): 支持 javax.json 中的“旧” JSON-P 类型。
- [jackson-datatype-json-org](https://github.com/FasterXML/jackson-datatypes-misc/json-org): 支持 org.json 库类型，如 JSONObject、JSONArray。
- [jackson-lombok](https://github.com/xebia/jackson-lombok): 更好地支持 Lombok 类。

### 1.3. Providers for JAX-RS

[Jackson JAX-RS Providers](https://github.com/FasterXML/jackson-jaxrs-providers) 有处理程序来为 JAX-RS 实现（如 Jersey、RESTeasy、CXF）添加数据格式支持。提供者实现 MessageBodyReader 和 MessageBodyWriter。 目前支持的格式包括 `JSON`、`Smile`、`XML`、`YAML` 和 `CBOR`。

### 1.4. Data format modules

数据格式模块对 JSON 格式之外的数据格式提供了支持，它们中的大多数模块只是简单实现了`Streaming`API抽象，因此可以按原样使用数据绑定组件；有些提供（并且很少需要）额外的数据绑定级别功能来处理模式之类的事情。

目前以下数据格式模块是完全可用和支持的（如果没有说明版本则是从2.0 开始包含）：

- [Avro](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/avro): 支持 [Avro](http://en.wikipedia.org/wiki/Apache_Avro) 数据格式，具有`streaming`实现以及对 Avro Schemas 的额外数据绑定级支持。
- [CBOR](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/cbor): 支持 [CBOR](http://tools.ietf.org/search/rfc7049) 数据格式 (二进制JSON变体)。
- [CSV](https://github.com/FasterXML/jackson-dataformats-text/blob/master/csv): 支持 [Comma-separated values](http://en.wikipedia.org/wiki/Comma-separated_values)(逗号分隔值) 格式-- `streaming` API，具有可选的便利`databind`添加。
- [Ion](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/ion) (2.9): support for [Amazon Ion](https://amznlabs.github.io/ion-docs/) binary data format (similar to CBOR, Smile, i.e. binary JSON - like)
- [(Java) Properties](https://github.com/FasterXML/jackson-dataformats-text/blob/master/properties) (2.8): 根据隐含符号创建嵌套结构（默认为点，可配置），在序列化时类似地展平。
- [Protobuf](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/protobuf) (2.6): 支持类似于 `Avro`。
- [Smile](https://github.com/FasterXML/jackson-dataformats-binary/tree/master/smile): supports [Smile (binary JSON)](https://github.com/FasterXML/smile-format-specification) -- 100% API/logical model compatible via `streaming` API, no changes for `databind`
- [XML](https://github.com/FasterXML/jackson-dataformat-xml): 支持 XML；提供 `streaming` 和 `databind` 实现，类似于 JAXB 的 “code-first” 模式（不支持 “XML Schema first”，但可以使用 JAXB bean）
- [YAML](https://github.com/FasterXML/jackson-dataformats-text/blob/master/yaml): 支持 [YAML](http://en.wikipedia.org/wiki/Yaml), 与 JSON 类似，完全支持简单的 `streaming` 实现。

:white_check_mark: JAXB: Java Architecture for XML Binding(XML 绑定的 Java 体系结构)，允许[Java](https://zh.wikipedia.org/wiki/Java)开发人员将 Java 类映射为 [XML](https://zh.wikipedia.org/wiki/XML)表示方式。JAXB提供两种主要特性：将一个 Java 对象序列化为XML，以及反向操作，将XML解析成Java对象。换句话说，JAXB允许以XML格式存储和读取数据，而不需要程序的类结构实现特定的读取XML和保存XML的代码。

JAXB 是[Java EE](https://zh.wikipedia.org/wiki/Java_EE)平台的[API](https://zh.wikipedia.org/wiki/应用程序接口)之一，同时是[Java Web服务开发包](https://zh.wikipedia.org/w/index.php?title=Java_Web服务开发包&action=edit&redlink=1)（[JWSDP](https://en.wikipedia.org/wiki/Java_Web_Services_Development_Pack)）的一部分。JAXB也是[Web服务互操作性技术](https://zh.wikipedia.org/w/index.php?title=Web服务互操作性技术&action=edit&redlink=1)（[WSIT](https://en.wikipedia.org/wiki/Web_Services_Interoperability_Technology)）的基础之一. JAXB是J2SE1.6的一部分。

## 2. Installation

**Using Maven**	

```xml
<dependency>
    <!-- contained jackson-core、jackson-annotation -->
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
</dependency>
```

## 3. Getting Starter

- [Jackson in N minutes (Jackson 快速入门)](./jackson-in-N-minutes.md) 
- [How to solve type erasure with Jackson(Jackson - 解决类型擦除)](./jackson-type-erasure.md)
- [How to filter properties with Jackson (使用 Jackson 如何过滤属性)](./jackson-filter-properties.md) 
- [Serialization and custom serializer with Jaskson (Jackson 序列化及自定义序列化器)](./jackson-serialize.md)
- [Deserialization and custom deserializer with Jaskson (Jackson 反序列化及自定义反序列化器)](./jackson-deserialize.md)
- [How to serialize and deserialize Enums with Jackson（使用 Jackson 怎样序列化/反序列化枚举)](./jackson-with-enums.md)

## 4. Streaming Moudle

- [Jackson-streaming-getting-starter (Jackson 流处理模块快速入门)](./jackson-streaming-getting-starter.md)

## 5. F.Q.A.

### 6.1. Q. List

1. [Jackson 的默认属性检测规则是什么？]()

### 6.2. A. List

1. Jackson 的默认属性检测(发现)规则是什么？

- 所有的 `public` 字段。
- 所有的 `public getters`方法。
- 所有的 `setter` 方法，无论访问修饰符的可见性是什么。

但如果以上的规则对您都不适用，如果您想，您可以使用 `@JsonAutoDetect` 注解改变可见性级别。如需要自动检测所有字段，您可以这样做：

```java
@JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.ANY)
public class POJOWithFields {
  private int value;
}
```

或者，去完全禁用字段的自动检测：

```java
@JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.NONE)
public class POJOWithNoFields {

  // 将不会被包括在内，除非有访问“getValue()”
  public int value;
}
```

除了使用 `fieldVisibility` 改变字段的可见性之外，我们还可以分别使用 `setterVisibility、getterVisibility、isGetterVisibility、creatorVisibility` 属性去改变 `setter、getter、is-getter(boolean)、创建者模式(工厂方法)`的可见性。	