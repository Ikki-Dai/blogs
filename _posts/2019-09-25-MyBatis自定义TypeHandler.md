# Mybatis自定义TypeHandler

## 背景

1. 在关系型数据库中希望更灵活的管理一些仅仅展示的数据
2. 未完全考虑全面的业务字段，需要随时拓展
3. 这些数据没有关联关系，或不希望新建表维护少量的关联关系


## 坑

- 自定义的TypeHandler 必须不是接口，不是抽象类，有无参构造函数, 不能是匿名类

```java
//  mybatis-spring-boot-starter 2.0.1
if (hasLength(this.typeHandlersPackage)) {
    scanClasses(this.typeHandlersPackage, TypeHandler.class).stream()
        .filter(clazz -> !clazz.isInterface())
        .filter(clazz -> !Modifier.isAbstract(clazz.getModifiers()))
        .filter(clazz -> ClassUtils.getConstructorIfAvailable(clazz) != null)
        .forEach(targetConfiguration.getTypeHandlerRegistry()::register);
}
//  mybatis-spring-boot-starter 2.1.0
if (hasLength(this.typeHandlersPackage)) {
    scanClasses(this.typeHandlersPackage, TypeHandler.class).stream()
        .filter(clazz -> !clazz.isAnonymousClass())
        .filter(clazz -> !clazz.isInterface())
        .filter(clazz -> !Modifier.isAbstract(clazz.getModifiers()))
        .filter(clazz -> ClassUtils.getConstructorIfAvailable(clazz) != null)
        .forEach(targetConfiguration.getTypeHandlerRegistry()::register);
}
```

- 针对某一个父类下的类型处理时，`@MappedType` 注解里只能使用类，不能使用接口，否则会导致对应的自定义TypeHandler 无法找到

```java

  private Map<JdbcType, TypeHandler<?>> getJdbcHandlerMapForSuperclass(Class<?> clazz) {
    Class<?> superclass =  clazz.getSuperclass();
    if (superclass == null || Object.class.equals(superclass)) {
      return null;
    }
    Map<JdbcType, TypeHandler<?>> jdbcHandlerMap = typeHandlerMap.get(superclass);
    if (jdbcHandlerMap != null) {
      return jdbcHandlerMap;
    } else {
      return getJdbcHandlerMapForSuperclass(superclass);
    }
  }
```




## 使用Map 处理 Json类型

- 如果需要在业务中解析Json 某个字段的值，使用`JsonNode`更加方便

```java
MappedTypes({AbstractMap.class, JsonNode.class})
public class JsonTypeHandler<T> extends BaseTypeHandler {

    private ObjectMapper objectMapper = new ObjectMapper();
    private Class<T> type;
    private String typeName;
    
    public JsonTypeHandler() { }

    public JsonTypeHandler(Class type) {
        typeName = type.getTypeName();
        this.type = type;
    }

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
        try {
            ps.setString(i, objectMapper.writeValueAsString(parameter));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Object getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String value = rs.getString(columnName);
        if(null != value){
            try {
                if("java.util.AbstractMap".equals(typeName)) {
                    return objectMapper.readValue(value,Map.class);
                }
                if("com.fasterxml.jackson.databind.JsonNode".equals(typeName)) {
                    return objectMapper.readTree(value);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    @Override
    public Object getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        String value = rs.getString(columnIndex);
        if(null!=value){
            try {
                if("java.util.AbstractMap".equals(typeName)) {
                    return objectMapper.readValue(value,Map.class);
                }
                if("com.fasterxml.jackson.databind.JsonNode".equals(typeName)) {
                    return objectMapper.readTree(value);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    @Override
    public Object getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        String value = cs.getString(columnIndex);
        if(null!=value){
            try {
                if("java.util.AbstractMap".equals(typeName)) {
                    return objectMapper.readValue(value,Map.class);
                }
                if("com.fasterxml.jackson.databind.JsonNode".equals(typeName)) {
                    return objectMapper.readTree(value);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
    
}
```

## 使用List处理 String 数组

- TypeHandler 中无法获取 List 泛型，所以只能强制处理 String 类型的数组

- List<String> 落库格式 'A,B,C' 方便MySQL `FIND_IN_SET()` 函数使用

- List 中 String 元素不能含有 ',' 否则会导致反序列化后元素个数与预期的不一致

```java


@MappedTypes({AbstractList.class})
public class ListTypeHandler extends BaseTypeHandler<AbstractList<String>> {

    public static final String SPLITER = ",";

    public ListTypeHandler() { }

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, AbstractList<String> parameter, JdbcType jdbcType) throws SQLException {
        StringBuilder stringBuilder = new StringBuilder();
        for (String s : parameter) {
            stringBuilder.append(s);
            stringBuilder.append(SPLITER);
        }
        stringBuilder.delete(stringBuilder.length() - 1, stringBuilder.length());
        ps.setString(i, stringBuilder.toString());
    }

    @Override
    public AbstractList<String> getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String value = rs.getString(columnName);
//        List<String> list =
        AbstractList<String> list = new LinkedList<>();
        if (null != value) {
            StringTokenizer stringTokenizer = new StringTokenizer(value, SPLITER);
            while (stringTokenizer.hasMoreTokens()) {
                list.add(stringTokenizer.nextToken());
            }
            return list;
        }
        return list;
    }

    @Override
    public AbstractList<String> getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        String value = rs.getString(columnIndex);
        AbstractList<String> list = new LinkedList<>();
        if (null != value) {
            StringTokenizer stringTokenizer = new StringTokenizer(value, SPLITER);
            while (stringTokenizer.hasMoreTokens()) {
                list.add(stringTokenizer.nextToken());
            }
            return list;
        }
        return list;
    }

    @Override
    public AbstractList<String> getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        String value = cs.getString(columnIndex);
        AbstractList<String> list = new LinkedList<>();
        if (null != value) {
            StringTokenizer stringTokenizer = new StringTokenizer(value, SPLITER);
            while (stringTokenizer.hasMoreTokens()) {
                list.add(stringTokenizer.nextToken());
            }
            return list;
        }
        return list;
    }
}
```



##  举个栗子

```java
@Getter
@Setter
public class Student {
    private String name;
    private List<String> courses;
    private Map items;
}
```

- MyBatis 在设置参数时，会将 courses 和item 字段 分别处理成 'A,B,C' 和 json 格式写入到数据库
- 对应的数据库字段设置成字符串类型即可


```properties
mybatis.type-handlers-package=com.xxx.handler
```




