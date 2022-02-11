
## 添加依赖
```xml
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </dependency>
```

## 调整格式

```java
/**
* 将字段作为 xml 标签的属性，并指定属性名称
* <element nm="xxx"></element>
*
*/
@JacksonXmlProperty(isAttribute = true, localName = "nm")
private String name;

/**
* 将字段包裹在标签中间
*/
@JacksonXmlText
private String xmlText;
```