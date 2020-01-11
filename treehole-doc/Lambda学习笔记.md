# Lambda学习笔记

## 1. 背景
面向对象编程语言和函数式编程语言中的基本元素都可以动态封装程序：面向对象编程语言使用带有方法的对象封装行为，函数式编程语言使用函数封装对象。

随着回调模式和函数式编程风格的日益流行，我们需要在Java中提供一种尽可能轻量级的将代码封装为数据（Model code as data）的方法。匿名内部类并不是一个好的 选择，因为：

1. 语法过于冗余
2. 匿名类中的 this 和变量名容易使人产生误解
3. 类型载入和实例创建语义不够灵活
4. 无法捕获非 final 的局部变量
5. 无法对控制流进行抽象

上面的多数问题均在Java SE 8中得以解决：

- 通过提供更简洁的语法和局部作用域规则，Java SE 8 彻底解决了问题 1 和问题 2
- 通过提供更加灵活而且便于优化的表达式语义，Java SE 8 绕开了问题 3
- 通过允许编译器推断变量的“常量性”（finality），Java SE 8 减轻了问题 4 带来的困扰

不过，Java SE 8 的目标并非解决所有上述问题。因此捕获可变变量（问题 4）和非局部控制流（问题 5）并不在 Java SE 8的范畴之内。（尽管我们可能会在未来提供对这些特性的支持）
## 2. 函数式接口（Functional interfaces）
把只拥有一个方法的接口称为*函数式接口*，如`Runnable`接口和`Comparator`接口。编译器会根据接口的结构自行判断，可以通过`@FunctionalInterface`注解来显式指定一个接口是函数式接口。

现有的类库大量使用了函数式接口，通过沿用这种模式，我们使得现有类库能够直接使用 lambda 表达式。例如下面是 Java SE 7 中已经存在的函数式接口：
- `java.lang.Runnable`
- `java.util.concurrent.Callable`
- `java.security.PrivilegedAction`
- `java.util.Comparator`
- `java.io.FileFilter`
- `java.beans.PropertyChangeListener`

除此之外，Java SE 8中增加了一个新的包：java.util.function，它里面包含了常用的函数式接口，例如：

- `Predicate<T>`——接收 T 并返回 boolean
- `Consumer<T>`——接收 T，不返回值
- `Function<T, R>`——接收 T，返回 R
- `Supplier<T>`——提供 T 对象（例如工厂），不接收值
- `UnaryOperator<T>`——接收 T 对象，返回 T
- `BinaryOperator<T>`——接收两个 T，返回 T

## 3. lambda表达式（lambda expressions）
匿名类型最大的问题就在于其冗余的语法。有人戏称匿名类型导致了“高度问题”（height problem）：比如前面 ActionListener 的例子里的五行代码中仅有一行在做实际工作。

lambda 表达式是匿名方法，它提供了轻量级的语法，从而解决了匿名内部类带来的“高度问题”。

```Java
(int x, int y) -> x + y
() -> 42
(String s) -> { System.out.println(s); }
```

第一个 lambda 表达式接收`x`和`y`这两个整形参数并返回它们的和；第二个 lambda 表达式不接收参数，返回整数 ‘42’；第三个lambda表达式接收一个字符串并把它打印到控制台，不返回值。

lambda 表达式的语法由参数列表、箭头符号 -> 和函数体组成。函数体既可以是一个表达式，也可以是一个语句块：
- 表达式：表达式会被执行然后返回执行结果。
- 语句块：语句块中的语句会被依次执行，就像方法中的语句一样——`return`语句会把控制权交给匿名方法的调用者；`break`和 `continue`只能在循环中使用；如果函数体有返回值，那么函数体内部的每一条路径都必须返回值。

表达式函数体适合小型 lambda 表达式，它消除了`return`关键字，使得语法更加简洁。

lambda 表达式也会经常出现在嵌套环境中，比如说作为方法的参数。为了使 lambda 表达式在这些场景下尽可能简洁，我们去除了不必要的分隔符。不过在某些情况下我们也可以把它分为多行，然后用括号包起来，就像其它普通表达式一样。

下面是一些出现在语句中的 lambda 表达式：

```Java
FileFilter java = (File f) -> f.getName().endsWith("*.java");
String user = doPrivileged(() -> System.getProperty("user.name"));
new Thread(() -> {
  connectToService();
  sendNotification();
}).start();
```

## 4. 目标类型（Target typing）
对于给定的 lambda 表达式，它的类型是什么？答案是：它的类型是由其上下文推导而来。
同样的 lambda 表达式在不同上下文里可以拥有不同的类型：

```Java
Callable<String> c = () -> "done";
PrivilegedAction<String> a = () -> "done";
```

第一个 lambda 表达式 `() -> "done"` 是 `Callable` 的实例，而第二个 lambda 表达式则是 `PrivilegedAction` 的实例。

编译器负责推导 lambda 表达式类型。它利用 lambda 表达式所在上下文 所期待的类型 进行推导，这个 被期待的类型 被称为 目标类型。lambda 表达式只能出现在目标类型为函数式接口的上下文中。

当且仅当下面所有条件均满足时，lambda 表达式才可以被赋给目标类型 `T`：

- `T` 是一个函数式接口
- lambda 表达式的参数和 `T` 的方法参数在数量和类型上一一对应
- lambda 表达式的返回值和 `T` 的方法返回值相兼容（Compatible）
- lambda 表达式内所抛出的异常和 `T` 的方法 `throws` 类型相兼容

由于目标类型（函数式接口）已经“知道” lambda 表达式的形式参数（Formal parameter）类型，所以我们没有必要把已知类型再重复一遍。也就是说，lambda 表达式的参数类型可以从目标类型中得出：

```Java
Comparator<String> c = (s1, s2) -> s1.compareToIgnoreCase(s2);
```

在上面的例子里，编译器可以推导出 s1 和 s2 的类型是 String。此外，当 lambda 的参数只有一个而且它的类型可以被推导得知时，该参数列表外面的括号可以被省略：

```Java
FileFilter java = f -> f.getName().endsWith(".java");
button.addActionListener(e -> ui.dazzle(e.getModifiers()));
```

## 5. 目标类型的上下文（Contexts for target typing）
带有目标类型的上下文：

- 变量声明
- 赋值
- 返回语句
- 数组初始化器
- 方法和构造方法的参数
- lambda 表达式函数体
- 条件表达式（? :）
- 转型（Cast）表达式

在前三个上下文（变量声明、赋值和返回语句）里，目标类型即是被赋值或被返回的类型：

```Java
Comparator<String> c;
c = (String s1, String s2) -> s1.compareToIgnoreCase(s2);
public Runnable toDoLater() {
  return () -> {
    System.out.println("later");
  }
}
```

数组初始化器和赋值类似，只是这里的“变量”变成了数组元素，而类型是从数组类型中推导得知：

```Java
filterFiles(
  new FileFilter[] {
    f -> f.exists(), f -> f.canRead(), f -> f.getName().startsWith("q")
  });
```

方法参数的类型推导要相对复杂些：目标类型的确认会涉及到其它两个语言特性：重载解析（Overload resolution）和参数类型推导（Type argument inference）。

重载解析会为一个给定的方法调用（method invocation）寻找最合适的方法声明（method declaration）。由于不同的声明具有不同的签名，当 lambda 表达式作为方法参数时，重载解析就会影响到 lambda 表达式的目标类型。编译器会通过它所得之的信息来做出决定。如果 lambda 表达式具有 **显式类型**（参数类型被显式指定），编译器就可以直接 使用lambda 表达式的返回类型；如果lambda表达式具有 **隐式类型**（参数类型被推导而知），重载解析则会忽略 lambda 表达式函数体而只依赖 lambda 表达式参数的数量。

如果在解析方法声明时存在二义性（ambiguous），我们就需要利用转型（cast）或显式 lambda 表达式来提供更多的类型信息。如果 lambda 表达式的返回类型依赖于其参数的类型，那么 lambda 表达式函数体有可能可以给编译器提供额外的信息，以便其推导参数类型。

```Java
List<Person> ps = ...
Stream<String> names = ps.stream().map(p -> p.getName());
```

在上面的代码中，`ps` 的类型是 `List<Person>`，所以 `ps.stream()` 的返回类型是 `Stream<Person>`。`map()` 方法接收一个类型为 `Function<T, R>` 的函数式接口，这里 `T` 的类型即是 `Stream` 元素的类型，也就是 `Person`，而 `R` 的类型未知。由于在重载解析之后 lambda 表达式的目标类型仍然未知，我们就需要推导 `R` 的类型：通过对 lambda 表达式函数体进行类型检查，我们发现函数体返回 `String`，因此 `R` 的类型是 `String`，因而 `map()` 返回 `Stream<String>`。绝大多数情况下编译器都能解析出正确的类型，但如果碰到无法解析的情况，我们则需要：

- 使用显式 lambda 表达式（为参数 `p` 提供显式类型）以提供额外的类型信息
- 把 lambda 表达式转型为 `Function<Person, String>`
- 为泛型参数 `R` 提供一个实际类型。（`.<String>map(p -> p.getName())`）

## 6. 词法作用域（Lexical scoping）
lambda 表达式的语义就十分简单：它不会从超类（supertype）中继承任何变量名，也不会引入一个新的作用域。lambda 表达式基于词法作用域，也就是说 lambda 表达式函数体里面的变量和它外部环境的变量具有相同的语义（也包括 lambda 表达式的形式参数）。此外，’this’ 关键字及其引用在 lambda 表达式内部和外部也拥有相同的语义。

基于词法作用域的理念，lambda 表达式不可以掩盖任何其所在上下文中的局部变量，它的行为和那些拥有参数的控制流结构（例如 `for` 循环和 `catch` 从句）一致。

##7. 变量捕获（Variable capture）
在 Java SE 7 中，编译器对内部类中引用的外部变量（即捕获的变量）要求非常严格：如果捕获的变量没有被声明为 `final` 就会产生一个编译错误。我们现在放宽了这个限制——对于 lambda 表达式和内部类，我们允许在其中捕获那些符合 有效只读（Effectively final）的局部变量。

简单的说，如果一个局部变量在初始化后从未被修改过，那么它就符合有效只读的要求，换句话说，加上 final 后也不会导致编译错误的局部变量就是有效只读变量。

```Java
Callable<String> helloCallable(String name) {
  String hello = "Hello";
  return () -> (hello + ", " + name);
}
```

试图修改捕获变量的行为仍然会被禁止，比如下面这个例子就是非法的：

```Java
int sum = 0;
list.forEach(e -> { sum += e.size(); });
```

因为这样的 lambda 表达式很容易引起 race condition。除非我们能够强制（最好是在编译时）这样的函数不能离开其当前线程，但如果这么做了可能会导致更多的问题。简而言之，lambda 表达式对 值 封闭，对 变量 开放。

## 8. 方法引用（Method references）
lambda 表达式允许我们定义一个匿名方法，并允许我们以函数式接口的方式使用它。我们也希望能够在 已有的 方法上实现同样的特性。

方法引用和 lambda 表达式拥有相同的特性（例如，它们都需要一个目标类型，并需要被转化为函数式接口的实例），不过我们并不需要为方法引用提供方法体，我们可以直接通过方法名称引用已有方法。

以下面的代码为例，假设我们要按照 `name` 或 `age` 为 `Person` 数组进行排序：

```Java
class Person {
  private final String name;
  private final int age;
  public int getAge() { return age; }
  public String getName() {return name; }
  ...
}
Person[] people = ...
Comparator<Person> byName = Comparator.comparing(p -> p.getName());
Arrays.sort(people, byName);
```

在这里我们可以用方法引用代替lambda表达式：

```Java
Comparator<Person> byName = Comparator.comparing(Person::getName);
```

这里的 `Person::getName` 可以被看作为 lambda 表达式的简写形式。尽管方法引用不一定（比如在这个例子里）会把语法变的更紧凑，但它拥有更明确的语义——如果我们想要调用的方法拥有一个名字，我们就可以通过它的名字直接调用它。

## 9. 方法引用的种类（Kinds of method references）
方法引用有很多种，它们的语法如下：

- 静态方法引用：`ClassName::methodName`
- 实例上的实例方法引用：`instanceReference::methodName`
- 超类上的实例方法引用：`super::methodName`
- 类型上的实例方法引用：`ClassName::methodName`
- 构造方法引用：`Class::new`
- 数组构造方法引用：`TypeName[]::new`

## 10. 默认方法和静态接口方法（Default and static interface methods）

## 11. 继承默认方法（Inheritance of default methods）

## 12. 融会贯通（Putting it together）

TODO


http://zh.lucida.me/blog/java-8-lambdas-insideout-language-features/