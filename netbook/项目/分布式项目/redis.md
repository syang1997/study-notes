# redis

## 导入依赖

spring-boot-starter-data-redis

## 优化查询

``` java
Object object=RedisTemplate.opsForValue().get("模块前缀"+id);
if(object==null){
    //去数据库中查询
    RedisTemplate.opsForValue().set("模块前缀"+id);
}
```



## RedisTemplate常用的方法

1. stringredistemp1ate, opsfor Value(),set("test","16",60*10, Timeunit. SECONDS);/向 redis里存入数据
2. stringredistemplate. opsforva1ue().get("test")/根据key获取缓存中的va1
3. stringredistemplate. boundvalueops("test").increment(-1); //val-1s1
4. stringredistemplate. boundvalueops("test").increment (1) //val +1
5. stringredistemp1ate, getexpire("test")//根据key获取过期时间I
6. stringredistemplate. getexpire("test", Timeunit. SECONDS)/根据key获取过期时间并换算成指定单位
7. stringredistemplate, delete("test");/根据key删除缓存
8. stringredistemplate, haskey("546545");//检查key是否存在,返回 boolean值
9. stringredistemplate. expire("red_123",160e, Timeunit.MエLLエ SECONDS);//设置过期时间
10. stringredistemplate. opsforset().add("red_123","1","2","3");//向指定key中存放set集合
11. stringredistemplate. opsforset(), i smember("red_123","1")/根据key査看集合中是否存在指定数据
12. stringredistemp1ate, opsforset(). members("red_123");//根据key获取set集合

