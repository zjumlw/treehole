# Stream学习笔记
## 1. Stream的定义
A sequence of elements supporting sequential and parallel aggregate operations.
翻译过来就是：Stream是元素的集合，这点让Stream看起来用些类似Iterator；可以支持顺序和并行的对原Stream进行汇聚的操作；
> 可以将Stream当成一个高级版本的Iterator。

## 2. 简单的例子
举一个简单的Stream使用的例子：获取一个List中元素不为null的个数。
```Java
@Test
public void testListElementCnt() {
    List<Integer> nums = Lists.newArrayList(1, null, 3, 4, null, 6);
    long cnt = nums.stream().filter(Objects::nonNull).count();
    System.out.println(cnt);
}
```

## 3. 语法剖析
@import "https://upload-images.jianshu.io/upload_images/1233356-920bd60dc4f70d94.jpg" {title="语法剖析"}

红色部分是Stream开始的地方，负责创建一个Stream实例；
绿色部分是Stream注入灵魂的地方，负责把一个Stream转换成另一个Stream；
蓝色部分是Stream收获的地方，负责把Stream里面包含的内容按照某种算法汇聚起来。
> Stream的基本步骤就是：1.创建Stream；2.转换Stream，每次转换原有Stream对象不改变，返回一个新的Stream对象；3.对Stream进行聚合操作，获取想要的结果。

## 4. 创建Stream
创建Stram有两种途径：通过Stream接口的静态工厂方法；通过Collection接口的默认方法，把一个Collection对象转换成Stream对象。
### 4.1 Stream静态方法
#### 4.1.1 of方法
有两个重载方法，一个接收变长参数，一个接收单一值。
```Java
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4);
Stream<String> stringStream = Stream.of("xiaomi");
```
#### 4.1.2 generator方法
生成无限长度的Stream，其元素的生成是通过给定的Supplier（这个接口可以看成一个对象的工厂，每次调用返回一个给定类型的对象）。
```Java
Stream.generate(new Supplier<Double>() {
    @Override
    public Double get() {
        return Math.random();
    }
});
// Lambda形式
Stream.generate(() -> Math.random());
Stream.generate(Math::random);
```
三条语句的作用都是一样的，只是使用了lambda表达式和方法引用的语法来简化代码。每条语句其实都是生成一个无限长度的Stream，其中值是随机的。这个无限长度Stream是懒加载，一般这种无限长度的Stream都会配合Stream的limit()方法来用。
#### 4.1.3 iterate方法
生成无限长度的Stream，和generator不同的是，其元素的生成是重复对给定的种子值(seed)调用用户指定函数来生成的。其中包含的元素可以认为是：seed，f(seed),f(f(seed))无限循环
```Java
Stream.iterate(1, item -> item + 1).limit(10).forEach(System.out::println);
```
这段代码就是先获取一个无限长度的正整数集合的Stream，然后取出前10个打印。千万记住使用limit方法，不然会无限打印下去。

### 4.2 通过Collection子类获取Stream
这个在本文的第一个例子中就展示了从List对象获取其对应的Stream对象，如果查看Java doc就可以发现Collection接口有一个stream方法，所以其所有子类都都可以获取对应的Stream对象。
```Java
public interface Collection<E> extends Iterable<E> {
//其他方法省略

default  Stream<E> stream() {
return StreamSupport.stream(spliterator(), false);
}
}
```

## 5. 转换Stream
转换Stream其实就是把一个Stream通过某些行为转换成一个新的Stream。Stream接口中定义了几个常用的转换方法，下面我们挑选几个常用的转换方法来解释。
### 5.1 distinct
对于Stream中包含的元素进行去重操作（去重逻辑依赖元素的equals方法），新生成的Stream中没有重复的元素；
![distinct](http://img04.taobaocdn.com/imgextra/i4/90219132/T2K0lnXPRXXXXXXXXX_!!90219132.jpg)
### 5.2 filter
对于Stream中包含的元素使用给定的过滤函数进行过滤操作，新生成的Stream只包含符合条件的元素；
![filter](http://img03.taobaocdn.com/imgextra/i3/90219132/T2OxXnXPlXXXXXXXXX_!!90219132.jpg)
### 5.3 map
对于Stream中包含的元素使用给定的转换函数进行转换操作，新生成的Stream只包含转换生成的元素。这个方法有三个对于原始类型的变种方法，分别是：mapToInt，mapToLong和mapToDouble。这三个方法也比较好理解，比如mapToInt就是把原始Stream转换成一个新的Stream，这个新生成的Stream中的元素都是int类型。之所以会有这样三个变种方法，可以免除自动装箱/拆箱的额外消耗；
![map](http://img03.taobaocdn.com/imgextra/i3/90219132/T2PQJnXOJXXXXXXXXX_!!90219132.jpg)
### 5.4 flatMap
和map类似，不同的是其每个元素转换得到的是Stream对象，会把子Stream中的元素压缩到父集合中；
![flatMap](http://img01.taobaocdn.com/imgextra/i1/90219132/T2mBXnXQhXXXXXXXXX_!!90219132.jpg)
### 5.5 peak
生成一个包含原Stream的所有元素的新Stream，同时会提供一个消费函数（Consumer实例），新Stream每个元素被消费的时候都会执行给定的消费函数；
![peak](http://img03.taobaocdn.com/imgextra/i3/90219132/T2DrFmXHtaXXXXXXXX_!!90219132.jpg)
### 5.6 limit
对一个Stream进行截断操作，获取其前N个元素，如果原Stream中包含的元素个数小于N，那就获取其所有的元素；
![limit](http://img02.taobaocdn.com/imgextra/i2/90219132/T2QAXlXJBaXXXXXXXX_!!90219132.jpg)
### 5.7 skip
返回一个丢弃原Stream的前N个元素后剩下元素组成的新Stream，如果原Stream中包含的元素个数小于N，那么返回空Stream；
![skip](http://img04.taobaocdn.com/imgextra/i4/90219132/T24A8mXUJXXXXXXXXX_!!90219132.jpg)
### 5.8　组合一下
```Java
List<Integer> nums = Lists.newArrayList(1,1,null,2,3,4,null,5,6,7,8,9,10);
System.out.println(“sum is:”+nums.stream().filter(num -> num != null).
            distinct().mapToInt(num -> num * 2).
            peek(System.out::println).skip(2).limit(4).sum());
```
这段代码演示了上面介绍的所有转换方法（除了flatMap），简单解释一下这段代码的含义：给定一个Integer类型的List，获取其对应的Stream对象，然后进行过滤掉null，再去重，再每个元素乘以2，再每个元素被消费的时候打印自身，在跳过前两个元素，最后去前四个元素进行加和运算(解释一大堆，很像废话，因为基本看了方法名就知道要做什么了。这个就是声明式编程的一大好处！)。
### 5.9 性能问题
有些细心的同学可能会有这样的疑问：在对于一个Stream进行多次转换操作，每次都对Stream的每个元素进行转换，而且是执行多次，这样时间复杂度就是一个for循环里把所有操作都做掉的N（转换的次数）倍啊。其实不是这样的，转换操作都是lazy的，多个转换操作只会在汇聚操作（见下节）的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在汇聚操作的时候循环Stream对应的集合，然后对每个元素执行所有的函数。

## 6. 汇聚（Reduce）Stream
汇聚操作（也称为折叠）接受一个元素序列为输入，反复使用某个合并操作，把序列中的元素合并成一个汇总的结果。比如查找一个数字列表的总和或者最大值，或者把这些数字累积成一个List对象。Stream接口有一些通用的汇聚操作，比如reduce()和collect()；也有一些特定用途的汇聚操作，比如sum(),max()和count()。
> 注意：sum方法不是所有的Stream对象都有的，只有IntStream、LongStream和DoubleStream是实例才有。

下面会分两部分来介绍汇聚操作：

- 可变汇聚：把输入的元素们累积到一个可变的容器中，比如Collection或者StringBuilder；
- 其他汇聚：除去可变汇聚剩下的，一般都不是通过反复修改某个可变对象，而是通过把前一次的汇聚结果当成下一次的入参，反复如此。比如reduce，count，allMatch；

### 6.1 可变汇聚
可变汇聚对应的只有一个方法：**collect**，正如其名字显示的，它可以把Stream中的要有元素收集到一个结果容器中（比如Collection）。先看一下最通用的collect方法的定义（还有其他override方法）：
```Java
<R> R collect(Supplier<R> supplier,
        BiConsumer<R, ? super T> accumulator,
        BiConsumer<R, R> combiner);
```
先来看看这三个参数的含义：
Supplier supplier是一个工厂函数，用来生成一个新的容器；
BiConsumer accumulator也是一个函数，用来把Stream中的元素添加到结果容器中；
BiConsumer combiner还是一个函数，用来把中间状态的多个结果容器合并成为一个（并发的时候会用到）。
```Java
List<Integer> nums = Lists.newArrayList(1,1,null,2,3,4,null,5,6,7,8,9,10);
List<Integer> numsWithoutNull = nums.stream().filter(num -> num != null).
        collect(() -> new ArrayList<Integer>(),
                (list, item) -> list.add(item),
                (list1, list2) -> list1.addAll(list2));
```
上面这段代码就是对一个元素是Integer类型的List，先过滤掉全部的null，然后把剩下的元素收集到一个新的List中。进一步看一下collect方法的三个参数，都是lambda形式的函数（上面的代码可以使用方法引用来简化，留给读者自己去思考）。
第一个函数生成一个新的ArrayList实例；
第二个函数接受两个参数，第一个是前面生成的ArrayList对象，二个是stream中包含的元素，函数体就是把stream中的元素加入ArrayList对象中。第二个函数被反复调用直到原stream的元素被消费完毕；
第三个函数也是接受两个参数，这两个都是ArrayList类型的，函数体就是把第二个ArrayList全部加入到第一个中；
但是上面的collect方法调用也有点太复杂了，没关系！我们来看一下collect方法另外一个override的版本，其依赖Collector。
<R, A> R collect(Collector<? super T, A, R> collector);
这样清爽多了！少年，还有好消息，Java8还给我们提供了Collector的工具类–Collectors，其中已经定义了一些静态工厂方法，比如：Collectors.toCollection()收集到Collection中, Collectors.toList()收集到List中和Collectors.toSet()收集到Set中。
```Java
List<Integer> numsWithoutNull = nums.stream()
        .filter(num -> num != null)
        .collect(Collectors.toList());
```

### 6.2 其他汇聚
#### 6.2.1 reduce方法
reduce方法非常的通用，后面介绍的count，sum等都可以使用其实现。reduce方法有三个override的方法，本文只介绍两种最常用的。
**第一种形式**：
```Java
Optional<T> reduce(BinaryOperator<T> accumulator);
```
接受一个BinaryOperator类型的参数，在使用的时候我们可以用lambda表达式来。
```Java
List<Integer> ints = Lists.newArrayList(1,2,3,4,5,6,7,8,9,10);
System.out.println("ints sum is:" + ints.stream().reduce((sum, item) -> sum + item).get());
```
可以看到reduce方法接受一个函数，这个函数有两个参数，第一个参数是上次函数执行的返回值（也称为中间结果），第二个参数是stream中的元素，这个函数把这两个值相加，得到的和会被赋值给下次执行这个函数的第一个参数。
**第二种形式**：
```Java
T reduce(T identity, BinaryOperator<T> accumulator);
```
这个定义上上面已经介绍过的基本一致，不同的是：它允许用户提供一个循环计算的初始值，如果Stream为空，就直接返回该值。而且这个方法不会返回Optional，因为其不会出现null值。
```Java
List<Integer> ints = Lists.newArrayList(1,2,3,4,5,6,7,8,9,10);
System.out.println("ints sum is:" + ints.stream().reduce(0, (sum, item) -> sum + item));
```
#### 6.2.2 count方法
获取Stream中元素的个数。比较简单，这里就直接给出例子，不做解释了。
```Java
List<Integer> ints = Lists.newArrayList(1,2,3,4,5,6,7,8,9,10);
System.out.println("ints sum is:" + ints.stream().count());
```
#### 6.2.3 allMatch
是不是Stream中的所有元素都满足给定的匹配条件
```Java
List<Integer&gt; ints = Lists.newArrayList(1,2,3,4,5,6,7,8,9,10);
System.out.println(ints.stream().allMatch(item -> item < 100));
```
#### 6.2.4 anyMatch
Stream中是否存在任何一个元素满足匹配条件
#### 6.2.5 findFirst
返回Stream中的第一个元素，如果Stream为空，返回空Optional
#### 6.2.6 noneMatch
是不是Stream中的所有元素都不满足给定的匹配条件
#### 6.2.7 max和min
使用给定的比较器（Operator），返回Stream中的最大|最小值
```Java
List<Integer&gt; ints = Lists.newArrayList(1,2,3,4,5,6,7,8,9,10);
ints.stream().max((o1, o2) -> o1.compareTo(o2)).ifPresent(System.out::println);
```


https://www.jianshu.com/p/3d34fbb8080e