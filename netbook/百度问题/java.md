## 2019.9.10

### java中集合的map分类

### 类型介绍

Java 自带了各种 Map 类。这些 Map 类可归为三种类型：

\1. 通用Map，用于在应用程序中管理映射，通常在 java.util 程序包中实现

HashMap、Hashtable、Properties、LinkedHashMap、IdentityHashMap、TreeMap、WeakHashMap、ConcurrentHashMap

\2. 专用Map，通常我们不必亲自创建此类Map，而是通过某些其他类对其进行访问

java.util.jar.Attributes、javax.print.attribute.standard.PrinterStateReasons、java.security.Provider、java.awt.RenderingHints、javax.swing.UIDefaults

\3. 一个用于帮助我们实现自己的Map类的抽象类

AbstractMap



### 类型区别`

HashMap

最常用的Map,它根据键的HashCode 值存储数据,根据键可以直接获取它的值，具有很快的访问速度。HashMap最多只允许一条记录的键为Null(多条会覆盖);允许多条记录的值为 Null。非同步的。

TreeMap

能够把它保存的记录根据键(key)排序,默认是按升序排序，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。TreeMap不允许key的值为null。非同步的。 
Hashtable

与 HashMap类似,不同的是:key和value的值均不允许为null;它支持线程的同步，即任一时刻只有一个线程能写Hashtable,因此也导致了Hashtale在写入时会比较慢。 
LinkedHashMap

保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.在遍历的时候会比HashMap慢。key和value均允许为空，非同步的。 



### Map 初始化

```
Map<String, String> map = new HashMap<String, String>();
```



### 插入元素

```
map.put("key1", "value1");
```



### 获取元素

```
	map.get("key1")
```



### 移除元素

```
map.remove("key1");
```



### 清空map

```
map.clear();
```



## 2四种常用Map插入与读取性能比较



### 测试环境

jdk1.7.0_80



### 测试结果

|               | 插入10次平均(ms) | 读取10次平均(ms) |      |      |      |      |
| ------------- | ---------------- | ---------------- | ---- | ---- | ---- | ---- |
|               | 1W               | 10W              | 100W | 1W   | 10W  | 100W |
| HashMap       | 56               | 261              | 3030 | 2    | 21   | 220  |
| LinkedHashMap | 25               | 229              | 3069 | 2    | 20   | 216  |
| TreeMap       | 29               | 295              | 4117 | 5    | 103  | 1446 |
| Hashtable     | 24               | 234              | 3275 | 2    | 22   | 259  |