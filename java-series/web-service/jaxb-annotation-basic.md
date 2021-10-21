# Basic Annotation

## 1.Intro

本节介绍 JAXB（Java Architecture for XML Binding）中的基本注解。

## 2. XmlRootElement

@XmlRootElement 用于支持将类映射为 XML 中的元素。在需要被映射的类上声明该注解，默认只映射 getter/setter 都存在属性。

如下，name 属性由于没有 getter/setter 方法则不会被映射：

```java
@XmlRootElement
@AllArgsConstructor
@NoArgsConstructor
public class Item {
    @Getter
    @Setter
    private int id;
    private String name;
    @Getter
    @Setter
    private BigDecimal size;
}

// test
JAXB.marshal(new Item(1, "item", BigDecimal.ONE), System.out);

// output:
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<item>
    <id>1</id>
    <size>1</size>
</item>
```





