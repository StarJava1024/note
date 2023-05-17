# [**属性名**MyBatis-Plus](https://baomidou.com/pages/223848/ "MyBatis-Plus")

## 1、配置

### 1）配置MyBatis日志

```yml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 控制台打印SQL语句
```

### 2）设置MyBatis-Plus的全局配置

```yml
// 设置MyBatis-Plus的全局配置
mybatis-plus:
  global-config:
    db-config:
      # 设置实体类所对应的表的统一前缀
      table-prefix: t_
```



## 2、BaseMapper方法实现

### 1）查询

#### （1）List<T> selectList

```Java
/**
* 根据 entity 条件，查询全部记录
* @param queryWrapper 实体对象封装操作类（可以为 null）
*/
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

```Java
// SELECT id,name,age,email FROM user
userMapper.selectList(null).forEach(System.out::println);
```

### 2）插入

#### （1）int insert(T entity);

```Java
/**
 * 插入一条记录
 * @param entity 实体对象
 */
int insert(T entity);
```

```java
// INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
userMapper.insert(entity);
```

### 3）删除

#### （1）int deleteById(Serializable id)

```Java
/**
* 根据 ID 删除
* @param id 主键ID
*/
int deleteById(Serializable id);
```

```Java
// DELETE FROM user WHERE id=?
userMapper.deleteById(1656492855965331457L);
```

#### （2）int deleteByMap

```Java
/**
* 根据 columnMap 条件，删除记录
* @param columnMap 表字段 map 对象
*/
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```

```Java
// DELETE FROM user WHERE name = ? AND age = ?
HashMap<String, Object> map = new HashMap<>();
map.put("name", "张三");
map.put("age", 23);
int result = userMapper.deleteByMap(map);
System.out.println("result: " + result);
```

#### （3）int deleteBatchIds

```Java
/**
 * 删除（根据ID 批量删除）
 * @param idList 主键ID列表(不能为 null 以及 empty)
 */
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
```

```Java
List<Long> asList = new ArrayList<>();
// SELECT id,name,age,email FROM user WHERE (name = ?)
List<User> users = userMapper.selectList(new LambdaQueryWrapper<User>().eq(User::getName, "张三"));
System.out.println("users: " + users.get(0).getId());
users.forEach(item -> {
    asList.add(item.getId());
});
// DELETE FROM user WHERE id IN ( ? , ? , ? )
userMapper.deleteBatchIds(asList);
```

### 4）修改

#### （1）int updateById

```Java
/**
* 根据 ID 修改
* @param entity 实体对象
*/
int updateById(@Param(Constants.ENTITY) T entity);
```

```Java
@Test
public void testUpdate() {
	User user = new User();
    List<User> users = userMapper.selectList(new LambdaQueryWrapper<User>().eq(User::getName, "张三"));
    users.forEach(item -> {
		user.setId(item.getId());
		user.setName("李四");
		user.setEmail("lisi@xing.com");
		// UPDATE user SET name=?, email=? WHERE id=?
		int result = userMapper.updateById(user);
		System.out.println("result: " + result);
	});
}
```

### 5）子查询

#### （1）inSql

```
/**
 * 字段 IN ( sql语句 )
 * <p>!! sql 注入方式的 in 方法 !!</p>
 * <p>例1: inSql("id", "1, 2, 3, 4, 5, 6")</p>
 * <p>例2: inSql("id", "select id from table where id &lt; 3")</p>
 *
 * @param condition 执行条件
 * @param column    字段
 * @param inValue   sql语句
 * @return children
 */
Children inSql(boolean condition, R column, String inValue);
```

```Java
@Test
public void test01() {
	QueryWrapper<User> queryWrapper = new QueryWrapper<>();
	queryWrapper.inSql("id", "select id from `user` where id <= 3");
	// SELECT id,name,age,email FROM user WHERE (id IN (select id from `user` where id <= 3))
	userMapper.selectList(queryWrapper);
}
```







## 3、注解

### 1、@MapperScan：用于扫描指定包下的mapper接口

### 2、@Repository：将一个或接口标识为持久层

### 3、[@TableName](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableName.java) ：设置实体类所对应的表名
|       属性       |  类型   | 是否<br />必须<br />指定 | 默认值 | 描述                                                         |
| :--------------: | :-----: | :----------------------: | :----: | :----------------------------------------------------------- |
|      value       | String  |            否            |   ""   | 表名<br />若不指定，则默认为驼峰转下划线。例：类名为UserStudent，则对应表名为user_student |
|      schema      | String  |            否            |   ""   | schema                                                       |
| keepGlobalPrefix | boolean |            否            | false  | 是否保持使用全局的 tablePrefix 的值(如果设置了全局 tablePrefix 且自行设置了 value 的值) |
|    resultMap     | String  |            否            |   ""   | xml 中 resultMap 的 id                                       |
|  autoResultMap   | boolean |            否            | false  | 是否自动构建 resultMap 并使用(若设置 resultMap 则不进行 resultMap 的自动构建并注入)。<br/>        mp会自动构建一个 ResultMap 并注入到 mybatis 里(一般用不上)。<br/>        因为mp底层是 mybatis，所以一些mybatis的常识你要知道，mp 只是帮你注入了常用 crud 到 mybatis 里 注入之前可以说是动态的(根据你 entity 的字段以及注解变化而变化)，但是注入之后是静态的(等于你写在xml的东西) 而对于直接指定 typeHandler，mybatis 只支持你写在2个地方：<br/>1.定义在resultMap里,只作用于select查询的返回结果封装<br/>2.定义在 insert 和 updatesql 的 #{property} 里的 property 后面(例：#{property,typehandler=xxx.xxx.xxx})，只作用于设置值 而除了这两种直接指定 typeHandler，mybatis 有一个全局的扫描你自己的 typeHandler 包的配置,这是根据你的 property 的类型去找 typeHandler 并使用 |

>
>value
>
>若数据库中所有表都有表名前缀，比如想让 t_user_info 表对应 UserInfo 实体类，可以添加如下全局配置设置表名前缀：
>
>```
>mybatis-plus.global-config.db-config.table-prefix=t_
>```
>
>若所有表名都不是下划线命名（但能跟类名对应上），比如想让 userinfo 表对应 UserInfo 实体类，可以添加如下全局配置，表示数据库表不使用下划线命名：
>
>```
>mybatis-plus.global-config.db-config.table-underline=false
>```

> value
>
> 若数据库中所有表都有表名前缀，比如想让 t_user_info 表对应 UserInfo 实体类，可以添加如下全局配置设置表名前缀：
>
> ```
> mybatis-plus.global-config.db-config.table-prefix=t_
> ```
>
> 若所有表名都不是下划线命名（但能跟类名对应上），比如想让 userinfo 表对应 UserInfo 实体类，可以添加如下全局配置，表示数据库表不使用下划线命名：
>
> ```
> mybatis-plus.global-config.db-config.table-underline=false
> ```

### 4、[@TableId](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableId.java) ：将属性所对应的字段指定为主键

| 属性  |  类型  | 是否<br />必须<br />指定 |   默认值    |     描述     |
| :---: | :----: | :----------------------: | :---------: | :----------: |
| value | String |            否            |     ""      | 主键表字段名 |
| type  |  Enum  |            否            | IdType.NONE |   主键类型   |

> @TableId(value="uid")   <--- value对应的是表的字段名
> private Long id;

#### [IdType](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/IdType.java) 

|    属性名     | 描述                                                         |
| :-----------: | ------------------------------------------------------------ |
|     AUTO      | 数据库ID自增。<br />需要将数据库ID设为自增，否则会报错。     |
|     NONE      | 未设置主键类型（注解里等于跟随全局（全局里默认为ASSIGN_ID)） |
|     INPUT     | 用户输入ID。（该类型可自己注册自动填充插件进行填充）。<br />若自己指定ID的值，则用指定的；若未指定ID的值且数据库ID是自增的，则用数据库的自增ID。 |
|   ASSIGN_ID   | 分配ID（主键类型为Number(Long和Integer)或String）(since 3.3.0)。<br/>若自己指定了ID的值，则用指定的；若未指定ID的值，则程序会自动生成。<br/>使用接口IdentifierGenerator的方法nextId（默认实现类为DefaultIdentifierGenerator雪花算法） |
|  ASSIGN_UUID  | 分配UUID，主键类型为String(since 3.3.0)。<br />若自己指定了ID的值，则用指定的；若未指定ID的值，则程序会自动生成。<br />使用接口IdentifierGenerator的方法nextUUID。32位UUID字符串(please use ASSIGN_UUID) |
|     UUID      | 32位UUID字符串(please use ASSIGN_UUID)                       |
|   ID_WORKER   | 分布式全局唯一ID 长整型类型(please use ASSIGN_ID)            |
| ID_WORKER_STR | 分布式全局唯一ID 字符串类型(please use ASSIGN_ID)            |

### 5、[@TableField](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableField.java) ：指定属性所对应的字段名

|       属性       |             类型             | 是否<br />必须<br />指定 |          默认值          | 描述                                                         |
| :--------------: | :--------------------------: | :----------------------: | :----------------------: | ------------------------------------------------------------ |
|      value       |            String            |            否            |            ""            | 数据库字段名                                                 |
|        el        |            String            |            否            |            ""            | 映射为原生 #{ ... } 逻辑,相当于写在 xml 里的 #{ ... } 部分   |
|      exist       |           boolean            |            否            |           true           | 是否为数据库表字段                                           |
|    condition     |            String            |            否            |            ""            | 字段 where 实体查询比较条件,有值设置则按设置的值为准,没有则为默认全局的 %s=#{%s},[参考](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlCondition.java) |
|      update      |            String            |            否            |            ""            | 字段 update set 部分注入, 例如：update="%s+1"：表示更新时会set version=version+1(该属性优先级高于 el 属性) |
|  insertStrategy  |             Enum             |            否            |         DEFAULT          | @TableField(insertStrategy= FieldStrategy.IGNORED)           |
|  updateStrategy  |             Enum             |            否            |         DEFAULT          | 举例：@TableField(updateStrategy = FieldStrategy.IGNORED)    |
|  whereStrategy   |             Enum             |            否            |         DEFAULT          | 举例：@TableField(whereStrategy= FieldStrategy.IGNORED)      |
|       fill       |             Enum             |            否            |    FieldFill.DEFAULT     | 字段自动填充策略                                             |
|      select      |           boolean            |            否            |           true           | 是否进行 select 查询                                         |
| keepGlobalFormat |           boolean            |            否            |          false           | 是否保持使用全局的 format 进行处理                           |
|     jdbcType     |           JdbcType           |            否            |    JdbcType.UNDEFINED    | JDBC类型 (该默认值不代表会按照该值生效)。<br />若不配合@TableName#autoResultMap = true一起使用,只生效于 update 的sql。 |
|   typeHandler    | Class<? extends TypeHandler> |            否            | UnknownTypeHandler.class | 类型处理器 (该默认值不代表会按照该值生效)。<br />若不配合@TableName#autoResultMap = true一起使用,只生效于 update 的sql。<br />如果你的字段类型和set进去的类型为equals关系,则只需要让你的typeHandler让Mybatis加载到即可,不需要使用注解。 |
|   numericScale   |            String            |            否            |            ""            | 指定小数点后保留的位数。<br />numericScale只生效于 update 的sql |

#### [FieldStrategy](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/FieldStrategy.java) 

|  属性值   | 描述                                                        |
| :-------: | ----------------------------------------------------------- |
|  IGNORED  | 忽略判断。即：无论是否是NULL都会写入。默认：NULL时不写入    |
| NOT_NULL  | 非NULL判断。即：操作前判断是否是NULL，若是则不操作。        |
| NOT_EMPTY | 非空判断。(只对字符串类型字段,其他类型字段依然为非NULL判断) |
|  DEFAULT  | 追随全局配置。（默认为NOT_NULL）                            |

#### [FieldFill](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/FieldFill.java) 

|     属性      | 描述                 |
| :-----------: | -------------------- |
|    DEFAULT    | 默认不处理           |
|    INSERT     | 插入时填充字段       |
|    UPDATE     | 更新时填充字段       |
| INSERT_UPDATE | 插入和更新时填充字段 |

### 6、[@Version](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/Version.java) ：标识乐观锁版本号字段、标记 @Verison 在字段上

> 说明：
>
> - **支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime** 
> - 整数类型下 `newVersion = oldVersion + 1` 
> - `newVersion` 会回写到 `entity` 中
> - 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
> - **在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!** 

### 7、[@EnumValue](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/EnumValue.java) ：将注解所标识的属性的值存储到数据库中(注解在枚举字段上)

### 8、[@TableLogic](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableLogic.java) ：表字段逻辑处理注解(逻辑删除)

|  属性  |  类型  | 是否<br />必须<br />指定 | 默认值 | 描述                                                         |
| :----: | :----: | :----------------------: | :----: | ------------------------------------------------------------ |
| value  | String |            否            |  "0"   | 逻辑未删除值。                                               |
| delval | String |            否            |  "1"   | 逻辑删除值。<br />也可写为字段名，如：id，这样删除时会将id的值写到逻辑删除字段 |

```Java
// 但要注意设置逻辑删除值为uid字段后is_deleted字段的类型要跟uid字段的类型一致
// @TableLogic(value = "0", delval = "uid")  // 逻辑未删除值为0，逻辑删除值为uid字段的值
@TableLogic(value = "0", delval = "1")  // 逻辑未删除值为0（默认值），逻辑删除值为1（默认值）
private Integer isDeleted;
```

### 9、[@SqlParser](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlParser.java) ：租户注解,支持method上以及mapper接口上

|  属性  |  类型   | 是否<br />必须<br />指定 | 默认值 | 描述                                                         |
| :----: | :-----: | :----------------------: | :----: | ------------------------------------------------------------ |
| filter | boolean |            否            | flase  | true: 表示过滤SQL解析，即不会进入ISqlParser解析链，否则会进解析链并追加例如tenant_id等条件 |

### 10、[@KeySequence](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/KeySequence.java) 

> 描述：序列主键策略 oracle
> 属性：value、resultMap

| 属性  |  类型  | 是否<br />必须<br />指定 |   默认值   | 描述                                                         |
| :---: | :----: | :----------------------: | :--------: | ------------------------------------------------------------ |
| value | String |            否            |     ""     | 序列名                                                       |
| clazz | Class  |            否            | Long.class | id的类型, 可以指定String.class，这样返回的Sequence值是字符串"1" |

### 11、@DS("master")：指定所操作的数据源

#### 配置多数据源

```yml
spring:
  # 配置数据源信息
  datasource:
    dynamic:
      # 设置默认的数据源或者数据源组,默认值即为master
      primary: master
      # 严格匹配数据源,默认false.true未匹配到指定数据源时抛异常,false使用默认数据源
      strict: false
      datasource:
        master:
          url: jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-8&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          username: root
          password: `8888`
        slave_1:
          url: jdbc:mysql://localhost:3306/mybatis_plus_1?characterEncoding=utf-8&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          username: root
          password: `8888`
```

#### 创建用户service

```Java
@DS("master") //指定所操作的数据源
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService { }
```

#### 创建商品

```Java
@DS("slave_1")
@Service
public class ProductServiceImpl extends ServiceImpl<ProductMapper, Product> implements ProductService { }
```

## Java实体类中的类型对应mybatis的jdbcType

| 实体类  | jdbcType  |
| :-----: | :-------: |
| String  |  VARCHAR  |
|  Date   | TIMESTAMP |
|   int   |  INTEGER  |
| Integer |  NUMERIC  |
| double  |  NUMERIC  |

## 分页插件：PaginationInnerInterceptor

### 支持的数据库

- mysql，oracle，db2，h2，hsql，sqlite，postgresql，sqlserver，Phoenix，Gauss ，clickhouse，Sybase，OceanBase，Firebird，cubrid，goldilocks，csiidb，informix，TDengine，redshift
- 达梦数据库，虚谷数据库，人大金仓数据库，南大通用(华库)数据库，南大通用数据库，神通数据库，瀚高数据库，优炫数据库

### 属性介绍

|  属性名  |   类型   | 默认值 |                             描述                             |
| :------: | :------: | :----: | :----------------------------------------------------------: |
| overflow | boolean  | false  | 溢出总页数后是否进行处理(默认不处理,参见 `插件#continuePage` 方法) |
| maxLimit |   Long   |        |  单页分页条数限制(默认无限制,参见 `插件#handlerLimit` 方法)  |
|  dbType  |  DbType  |        | 数据库类型(根据类型获取应使用的分页方言,参见 `插件#findIDialect` 方法) |
| dialect  | IDialect |        |          方言实现类(参见 `插件#findIDialect` 方法)           |

###  自定义的 mapper#method 使用分页

```Java
IPage<UserVo> selectPageVo(IPage<?> page, Integer state);
// or (class MyPage extends Ipage<UserVo>{ private Integer state; })
MyPage selectPageVo(MyPage page);
// or
List<UserVo> selectPageVo(IPage<UserVo> page, Integer state);
```

```xml
<select id="selectPageVo" resultType="xxx.xxx.xxx.UserVo">
    SELECT id,name FROM user WHERE state=#{state}
</select>
```

> 如果返回类型是 IPage 则入参的 IPage 不能为null,因为 返回的IPage == 入参的IPage; 如果想临时不分页,可以在初始化IPage时size参数传 <0 的值;
> 如果返回类型是 List 则入参的 IPage 可以为 null(为 null 则不分页),但需要你手动 入参的IPage.setRecords(返回的 List);
> 如果 xml 需要从 page 里取值,需要 `page.属性` 获取

###  其他

> 生成 countSql 会在 `left join` 的表不参与 `where` 条件的情况下,把 `left join` 优化掉
> 所以建议任何带有 `left join` 的sql,都写标准sql,即给于表一个别名,字段也要 `别名.字段`

> <span style="color:red; font-weight:bold">注意！</span>
>
> - 多个插件使用的情况，请将分页插件放到 `插件执行链` 最后面。如在租户插件前面，会出现 `COUNT` 执行 `SQL` 不准确问题。

### Page

> 该类继承了 `IPage` 类，实现了 `简单分页模型` 如果你要实现自己的分页模型可以继承 `Page` 类或者实现 `IPage` 类

|       **属性名**       | **类型** | **默认值** |                          **默认值**                          |
| :--------------------: | :------: | :--------: | :----------------------------------------------------------: |
|        records         |   List   | emptyList  |                         查询数据列表                         |
|         total          |   Long   |     0      |                       查询列表总记录数                       |
|          size          |   Long   |     10     |                   每页显示条数，默认 `10`                    |
|        current         |   Long   |     1      |                            当前页                            |
|         orders         |   List   | emptyList  | 排序字段信息，允许前端传入的时候，注意 SQL 注入问题，可以使用 `SqlInjectionUtils.check(...)` 检查文本 |
|    optimizeCountSql    | boolean  |    true    | 自动优化 COUNT SQL 如果遇到 `jSqlParser` 无法解析情况，设置该参数为 `false` |
| optimizeJoinOfCountSql | boolean  |    true    |         自动优化 COUNT SQL 是否把 join 查询部分移除          |
|      searchCount       | boolean  |    true    | 是否进行 count 查询，如果指向查询到列表不要查询总记录数，设置该参数为 `false` |
|        maxLimit        |   Long   |            |                       单页分页条数限制                       |
|        countId         |  String  |            |          `xml` 自定义 `count` 查询的 `statementId`           |


## 乐观锁插件

### OptimisticLockerInnerInterceptor

> 当要更新一条记录的时候，希望这条记录没有被别人更新
> 乐观锁实现方式：
>
> - 取出记录时，获取当前 version
> - 更新时，带上这个 version
> - 执行更新时， set version = newVersion where version = oldVersion
> - 如果 version 不对，就更新失败

### 1. 配置插件

spring xml 方式：

```xml
<bean class="com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor" id="optimisticLockerInnerInterceptor"/>

<bean id="mybatisPlusInterceptor" class="com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor">
    <property name="interceptors">
        <list>
            <ref bean="optimisticLockerInnerInterceptor"/>
        </list>
    </property>
</bean>
```

spring boot 注解方式（配置类）：

```Java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}
```

### 2. 在实体类的字段上加上`@Version`注解

```Java
@Version
private Integer version;
```

> 说明：
>
> - **支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime** 
> - 整数类型下 `newVersion = oldVersion + 1` 
> - `newVersion` 会回写到 `entity` 中
> - 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
> - **在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!** 

示例：

```JAva
// Spring Boot 方式
@Configuration
@MapperScan("按需修改")
public class MybatisPlusConfig {
    /**
     * 旧版
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }

    /**
     * 新版
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

## 多租户插件

### [TenantLineInnerInterceptor](https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-tenant) 

属性介绍

|      属性名       |       类型        | 默认值 |             描述              |
| :---------------: | :---------------: | :----: | :---------------------------: |
| tenantLineHandler | TenantLineHandler |        | 租户处理器（ TenantId 行级 ） |

```Java
public interface TenantLineHandler {

    /**
     * 获取租户 ID 值表达式，只支持单个 ID 值
     * <p>
     *
     * @return 租户 ID 值表达式
     */
    Expression getTenantId();

    /**
     * 获取租户字段名
     * <p>
     * 默认字段名叫: tenant_id
     *
     * @return 租户字段名
     */
    default String getTenantIdColumn() {
        // 如果该字段你不是固定的，请使用 SqlInjectionUtils.check 检查安全性
        return "tenant_id";
    }

    /**
     * 根据表名判断是否忽略拼接多租户条件
     * <p>
     * 默认都要进行解析并拼接多租户条件
     *
     * @param tableName 表名
     * @return 是否忽略, true:表示忽略，false:需要解析并拼接多租户条件
     */
    default boolean ignoreTable(String tableName) {
        return false;
    }
}
```

> 说明:
>
> 多租户 != 权限过滤,不要乱用,租户之间是完全隔离的!!!
> 启用多租户后所有执行的method的sql都会进行处理.
> 自写的sql请按规范书写(sql涉及到多个表的每个表都要给别名,特别是 inner join 的要写标准的 inner join)

## 防全表更新与删除插件

### BlockAttackInnerInterceptor

> 针对 update 和 delete 语句 作用: 阻止恶意的全表更新删除

注入MybatisPlusInterceptor类，并配置BlockAttackInnerInterceptor拦截器

```Java
@Configuration
public class MybatisPlusConfig {
	@Bean
	public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new BlockAttackInnerInterceptor());
        return interceptor;
	}
}
```

测试（全表更新）

```Java
@SpringBootTest
public class QueryWrapperTest {

	@Autowired
	private UserService userService;

	/**
	 * SQL：UPDATE user  SET name=?,email=?;
	 */
	@Test
	public void test() {
        User user = new User();
        user.setId(999L);
        user.setName("custom_name");
        user.setEmail("xxx@mail.com");
        // com.baomidou.mybatisplus.core.exceptions.MybatisPlusException: Prohibition of table update operation
        userService.saveOrUpdate(user, null);
	}
}
```

测试（部分更新）

```Java
@SpringBootTest
public class QueryWrapperTest {

    @Autowired
    private UserService userService;

	/**
	 * SQL：UPDATE user  SET name=?, email=? WHERE id = ?;
	 */
	@Test
	public void test() {
        LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>();
        wrapper.eq(User::getId, 1);
        User user = new User();
        user.setId(10L);
        user.setName("custom_name");
        user.setEmail("xxx@mail.com");
        userService.saveOrUpdate(user, wrapper);
	}
}
```

##　动态表名插件

### [DynamicTableNameInnerInterceptor](https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-dynamic-tablename) 

> <span style="color:red; font-weight:bold">注意事项：</span>
>
> - 原理为解析替换设定表名为处理器的返回表名，表名建议可以定义复杂一些避免误替换
> - 例如：真实表名为 user 设定为 mp_dt_user 处理器替换为 user_2019 等





<br/><br/>

# ******************** <span style="color:red; font-weight:bold">结束啦</span> ********************