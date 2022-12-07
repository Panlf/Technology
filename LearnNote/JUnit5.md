# JUnit5

## JUnit5常用注解

- @Test 表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会Jupiter提供额外测试。
- @ParameterizedTest 表示方法是参数化测试
- @RepeatedTest 表示方法可重复执行
- @DisplayName 表示为测试类或者测试方法设置展示名称
- @BeforeEach 表示在每个单元测试之前执行
- @AfterEach 表示在每个单元测试之后执行
- @BeforeAll 表示在所有单元测试之前执行
- @AfterAll 表示在所有单元测试之后执行
- @Tag 表示单元测试类别，类似于Junit4中的@Categories
- @Disabled 表示测试类或者测试方法不执行，类似于JUnit4中的@Ignore
- @Timeout 表示测试方法运行如果超过了指定时间将会返回错误
- @ExtendWith 表示为测试类或者测试方法提供扩展类引用

## 断言

- assertEquals 判断两个对象或者两个原始类型是否相等
- assertNotEquals 判断两个对象或者两个原始类型是否不相等
- assertSame 判断两个对象引用是否指向同一个对象
- assertNotSame 判断两个对象引用是否指向不同的对象
- assertTrue 判断给定的布尔值是否为true
- assertFalse 判断给定的布尔值是否为false
- assertNull 判断给定的对象引用是否为null
- assertNotNull 判断给定的对象引用是否不为null

## 参数化测试

- @ValueSource 为参数化测试指定入参来源，支持八大基础类以及String类型，Class类型
- @NullSource 为参数化测试提供一个null的入参
- @EnumSource 为参数化测试提供一个枚举的入参
- @CsvFileSource 读取指定CSV文件内容作为参数化测试入参
- @MethodSource 读取指定方法的返回值作为参数化测试入参（注意方法返回需要是一个流）