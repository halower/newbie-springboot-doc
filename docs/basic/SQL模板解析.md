# SQL模板解析

?> 在一些复杂的场景下,可能需要动态处理非常复杂的SQL语句，这里我们提供一个动态SQL模板的解析接口，默认的sql模板路径为`resources/sqls`

## 解析接口
```
    @Autowired
    SQLParser sqlParser;

    /**
     * 解析SQL模板
     * @param templateName 模板名称
     * @param methodName 方法名
     * @param map 参数键值对
     * @return  还原后的字符串
     */
    sqlParser.parse(String templateName, String methodName, Map<String,Object> map)
```

