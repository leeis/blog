---
title: 在Spring Boot中使用MyBatis
date: 2018-09-17 20:13:48
tags: 
  - Spring Boot
  - MyBatis
categories:
comments: true
---

#### 1. pom中引入MyBatis

到Maven仓库, 搜索spring boot mybatis, 复制最新的[maven](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/1.3.2)到`pom.xml`文件中

```
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>

```

#### 2. 创建Mapper

引入了statrer之后，我们不需要做其他额外的配置和依赖，直接创建Mapper, 这里创建一个`ShopAccountMapper`

```java
@Mapper
@Component
public interface ShopAccountMapper {
}
```

这里使用注解`@Mapper`来表示这是一个Mybatis的Mapper类，也可以直接在启动文件中指定mapper类所在的包，这样就可以避免在每个Mapper类上加入`@Mapper`注解

```java
@MapperScan(basePackages = "shop.leshare.mapper")
```

##### 简单查询

在Sql中引用参数需要用`#{}`来表示, 我们在参数中用`@Param`来指定参数名称，否则Mybatis默认会以`arg0`, `arg1`, `param0`, `param1`这样的名字来命名

对于返回结果的处理，如果存在实体属性和数据库字段不一致时，需要用`@Result`注解指定, 其中`property`是表示java实体属性, `column`表示数据库字段

```java
    @Select("select * from shop_account where shop_id=#{shopId} and sub_shop_id=#{subShopId}")
    @Results({
            @Result(property = "shopId", column = "shop_id"),
            @Result(property = "shopName", column = "shop_name"),
            @Result(property = "subShopId", column = "sub_shop_id"),
            @Result(property = "subShopName", column = "sub_shop_name"),
            @Result(property = "createTime", column = "create_time"),
            @Result(property = "updateTime", column = "update_time"),
            @Result(property = "recvAccountType", column = "recv_account_type"),
            @Result(property = "recvValue", column = "recv_value"),
            @Result(property = "recvValue2", column = "recv_value2"),
            @Result(property = "checkName", column = "check_name")
    })
    List<ShopAccount> findAccounts(@Param("shopId") int shopId, @Param("subShopId") int subShopId);
```

##### 条件查询

有时候我们需要根据不同的参数条件进行sql拼接，比如这里，仅当`subShopId`大于0的时候，才纳入查询条件，这时候就要用到`@SelectProvider`注解，`type`指定一个类，`method`指定该类的方法. 

`Provider`中方法返回的是一个String， 也就是说，在方法里根据给定的参数进行判断，返回sql语句即可。**当Mapper中参数大于等于2个时, Provider中方法接收的参数为`Map<String, Object>`**, Key就是在`@Param`中指定的名称

> 注意，当Provider参数包含对象时，要取对象中的某一个属性，可以直接用A.B的方式，比如取User中的name, 直接#{user.name}, 如果是多参数, 这个user要先从map中取出

```java
    @SelectProvider(type = ShopAccountProvider.class, method = "findList")
    @Results({
            @Result(property = "shopId", column = "shop_id"),
            @Result(property = "shopName", column = "shop_name"),
            @Result(property = "subShopId", column = "sub_shop_id"),
            @Result(property = "subShopName", column = "sub_shop_name"),
            @Result(property = "createTime", column = "create_time"),
            @Result(property = "updateTime", column = "update_time"),
            @Result(property = "recvAccountType", column = "recv_account_type"),
            @Result(property = "recvValue", column = "recv_value"),
            @Result(property = "recvValue2", column = "recv_value2"),
            @Result(property = "checkName", column = "check_name")
    })
    List<ShopAccount> findAccounts(@Param("shopId") int shopId, @Param("subShopId") int subShopId);
```

```java
public class ShopAccountProvider {
    public String findList(Map<String, Object> param){
        int subShopId = (int)param.get("subShopId");
        String sql = "select * from shop_account where shop_id=#{shopId}\n";
        if(subShopId > 0){
            sql += "and sub_shop_id=#{subShopId}";
        }
        return sql;
    }
}
```

##### 新增

用`@Insert`注解进行新增操作, 如果需要自动赋值自增长的ID, 需要用到`@Options`注解

```java
@Insert("insert into shop_account (\n" +
            "  shop_id,\n" +
            "  shop_name,\n" +
            "  sub_shop_id,\n" +
            "  sub_shop_name,\n" +
            "  balance,\n" +
            "  returns,\n" +
            "  create_time,\n" +
            "  update_time,\n" +
            "  recv_account_type,\n" +
            "  recv_value,\n" +
            "  recv_value2,\n" +
            "  check_name\n" +
            ") value (\n" +
            "  #{shopId},\n" +
            "  #{shopName},\n" +
            "  #{subShopId},\n" +
            "  #{subShopName},\n" +
            "  #{balance},\n" +
            "  #{returns},\n" +
            "  now(),\n" +
            "  now(),\n" +
            "  #{recvAccountType},\n" +
            "  #{recvValue},\n" +
            "  #{recvValue2},\n" +
            "  #{checkName}\n" +
            ")")
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
    void addAccount(ShopAccount account);
```

其他如更新用`@Update`注解, 删除用`@Delete`注解，使用方式同`@Insert`

当Mapper创建完成之后，直接在需要使用的Service中引入即可

```java
    @Autowired
    private ShopAccountMapper shopAccountMapper;
```