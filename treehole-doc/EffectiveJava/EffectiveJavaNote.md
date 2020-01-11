# Effective Java读书笔记

## 第二章
### 第1条：考虑用静态工厂方法代替构造器
静态工厂方法（static factory method），返回类的实例的静态方法。具有以下优点：

#### 1.1 相对于构造器，有名称
当一个类需要多个带有相同签名的构造器时，用静态工厂方法代替构造器，并且慎重地选择名称以便突出它们之间的区别。
#### 1.2 不必在每次调用它们时都创建一个新对象
避免创建不必要的重复对象。如果程序经常请求创建相同的对象，并且创建对象的代价很好，那么应该这么用。如果类总能严格控制在某个时刻哪些实例应该存在，那么可以被称作**实例受控的类（instance-controlled）**。这种类可以确保是一个Singleton或者是不可实例化的，使得不可变的类确保不会存在两个相等的实例。
#### 1.3 可以返回原返回类型的任何子类型的对象
选择返回对象的时候有更大的灵活性。
通过接口来引用被返回的对象，而不是其实现类。

## 第十章
### 第71条：慎用延迟初始化
延迟初始化（lazy initialization）是延迟到需要域的值时才将它初始化的行为。如果不需要这个值，这个域就永远不会被初始化。
虽然可以降低初始化类或者创建实例的开销，却增加了访问被延迟初始化的域的开销，是一把双刃剑。最好的建议是“**除非绝对需要，否则就不要这么做**”。
如果域只在类的实例部分被访问，并且初始化这个域的开销很高，可能就**值得**进行延迟初始化。
大多数情况下，正常的初始化要优先于延迟初始化。
正常初始化：
```Java
private final FieldType field = computeFieldValue();
```
延迟初始化，使用同步访问方法：
```Java
private FieldType field;
synchronized FieldType getField(){
    if(field == null){
        field = computeFieldValue();
    }
    return field;
}
```
对**静态域**使用延迟初始化，用**lazy initialization holder class**模式比较好：
```Java
private static class FieldHolder{
    static final FieldType field = computeFieldValue();
}
static FieldType getField(){return FieldHolder.field;}
```
当getField方法第一次被调用时，第一次读取FieldHolder.field，导致FieldHolder类得到初始化。

- 优点是：getField方法没有被同步，并且只执行一个域访问，延迟初始化并没有增加任何访问成本。

如果出于性能的考虑，需要对**实例域**使用延迟初始化，使用双重检查模式（double-check idiom）。避免了在域被初始化之后访问这个域时的锁定开销。
```Java
private volatile FieldType field;
FieldType getField(){
    FieldType result = field;
    if(result == null){    //First check(no locking)
        synchronized(this){
            result = field;
            if(result == null){//Second check(with locking)
                field = result  = computeFieldValue();
            }
        }
    }
    return result;
}
```
两次检查域的值

- 第一次检查时没有锁定，看看这个域是否被初始化了；
- 第二次检查时有锁定。只有此次检查时表明这个域没有被初始化，才会调用computeFieldValue方法对域进行初始化。所以域被声明为**volatile很重要**。

局部变量result的作用：确保field只在已经被初始化的情况下读取一次，可以提升性能。
可以接受重复初始化的实例域，可以使用单重检查模式（single-check idiom）
```Java
private volatile FieldType field;

private FieldType getField(){
    FieldType result = field;
    if(result == null){
        field = result = computeFieldValue();
    }
    return result;
}
```

#### 总结
实例域：双重检查模式(double-check idiom)
静态域：lazy initialization holder模式
可以接受重复初始化的实例域：单重检查模式（single-check idiom）