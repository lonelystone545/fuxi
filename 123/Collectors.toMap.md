



[TOC]

## Collectors.toMap异常

在使用Collectors.toMap方法时，会有两个常见的问题：1. 键冲突，导致抛出异常 2. 值为NULL，导致抛出空指针异常

下面给出上面两种问题的解决方案。

### 键冲突 ？

有三种策略，分别如下：

1. 用后面的value覆盖前面的value

   ```java
   Map<String, String> map = list.stream.collect(Collectors.toMap(Student::getName, Student::getAge, (key1, key2)->key2));
   ```

   同理，如果 `(key1, key2)->key1`则表示用前面的value覆盖后面的value，即保持不变。

2. 将重复key的value进行拼接

   ```java
   Map<String,String> map = list.stream.collect(Collectors.toMap(Student::getName, Student::getAge, (key1, key2)->key1 + "," + key2))
   ```

3. 将重复key的value作为list存储

   ```java
   Map<Long, List<String>> map = list.stream().collect(Collectors.toMap(Person::getId,
           person -> {
               List<String> list1 = new ArrayList<>();
               list1.add(person.getName());
               return list1;
           },
           (List<String> value1, List<String> value2) -> {
               value1.addAll(value2);
               return value1;
           }));
   ```

### 空指针异常 ？

对于HashMap来说，value是允许为null的，但是当用java8的stream将list转换为map时，默认是不允许值为null的，通过Collectors.toMap源码可知，

```java
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                            Function<? super T, ? extends U> valueMapper,
                            BinaryOperator<U> mergeFunction,
                            Supplier<M> mapSupplier) {
    BiConsumer<M, T> accumulator
            = (map, element) -> map.merge(keyMapper.apply(element),
                                          valueMapper.apply(element), mergeFunction);
    return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
}
```

采用了map.merge方法进行处理，merge方法源码如下：

```java
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

可以看到源码中对value值进行了限定不能为null，否则抛出空指针异常，`Objects.requireNonNull(value)`，merge方法的作用可以用javadoc注释进行解释：

```java
map.merge(key, msg, String::concat)；
V oldValue = map.get(key);
V newValue = (oldValue == null) ? value :
remappingFunction.apply(oldValue, value);
if (newValue == null)
     map.remove(key);
 else
     map.put(key, newValue);
```

常见的解决方法如下：

1. ```java
   Map<Long, String> memberMap = list.stream().collect(HashMap::new, (m, v) ->
           m.put(v.getId(), v.getName()), HashMap::putAll);
   ```

2. 自定义map集合，对list进行foreach遍历，依次取出其对应的属性存放为map的key和value。

3. 继承Collector，手动实现toMap方法，然后调用我们自己封装的toMap方法就可以了。

   有关实现Collector，[参考链接](http://blog.jobbole.com/104067/)

（另，对于key冲突抛出异常，已经在jdk9中得到解决，[参考](https://bugs.openjdk.java.net/browse/JDK-8173464)链接）