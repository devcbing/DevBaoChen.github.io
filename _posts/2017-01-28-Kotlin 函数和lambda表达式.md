---
layout: post
title: Kotlin 函数 和 lambda 表达式
categories: [kotlin, ]
description: Kotlin 函数 和 lambda 表达式
keywords: Kotlin
---

Kotlin 函数 和 lambda 表达式学习

![](/images/posts/kotlin/Kotlin_logo.png)

### 函数

#### 函数声明 

在 kotlin 中 使用 `fun` 关键字声明函数：

```
  fun double(x: Int): Int {
      return 2 * x
  }
```
#### 函数的用法：

```
  val result = double(2)
```
调用成员函数。

```
  Sample().foo() // 创建类 Sample 实例并调⽤ foo
```

#### 参数：

函数参数使⽤ Pascal 表⽰法定义，即 name: type。参数⽤逗号隔开。每个参数必须有显式类型：

```
  fun powerOf(number: Int, exponent: Int) { …… }
```
##### 默认参数：

函数参数可以有默认值，当省略相应的参数时使⽤默认值。与其他语⾔相⽐，这可以减少重载数量：

```
  fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) { …… }
```
默认值通过类型后⾯的 = 及给出的值来定义。
覆盖⽅法总是使⽤与基类型⽅法相同的默认参数值。当覆盖⼀个带有默认参数值的⽅法时，必须从签名中省略默认参数值：

```
  open class A {
      open fun foo(i: Int = 10) { …… }
  } 
  class B : A() {
      override fun foo(i: Int) { …… } // 不能有默认值
  }
```
如果⼀个默认参数在⼀个⽆默认值的参数之前，那么该默认值只能通过使⽤命名参数调⽤该函数来使⽤：

```
  fun foo(bar: Int = 0, baz: Int) { …… }
  foo(baz = 1) // 使⽤默认值 bar = 0
```

如果在默认参数之后的最后⼀个参数是 lambda 表达式，那么它既可以作为命名参数在括号内传⼊，也可以在括号外传⼊：

```
  fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { …… }
  foo(1) { println("hello") } // 使⽤默认值 baz = 1
  foo(qux = { println("hello") }) // 使⽤两个默认值 bar = 0 与 baz = 1
  foo { println("hello") } // 使⽤两个默认值 bar = 0 与 baz = 1
```

##### 命名参数

可以在调⽤函数时使⽤命名的函数参数。当⼀个函数有⼤量的参数或默认参数时这会⾮常⽅便。

```
  fun reformat(str: String,
              normalizeCase: Boolean = true,
              upperCaseFirstLetter: Boolean = true,
              divideByCamelHumps: Boolean = false,
              wordSeparator: Char = ' ') {
      ……
  }
```
们可以使⽤默认参数来调⽤它：

```
reformat(str)
```
当使⽤⾮默认参数调⽤它时，该调⽤看起来就像：

```
reformat(str, true, true, false, '_')
```
使⽤命名参数我们可以使代码更具有可读性：

```
  reformat(str,
      normalizeCase = true,
      upperCaseFirstLetter = true,
      divideByCamelHumps = false,
      wordSeparator = '_'
  )
```

并且如果我们不需要所有的参数：

```
reformat(str, wordSeparator = '_')
```
当⼀个函数调⽤混⽤位置参数与命名参数时，所有位置参数都要放在第⼀个命名参数之前。例如，允许调⽤ f(1, y = 2) 但不允许 f(x = 1, 2) 。

可以通过使⽤星号操作符将可变数量参数（vararg）以命名形式传⼊：

```
  fun foo(vararg strings: String) { …… }
  foo(strings = *arrayOf("a", "b", "c"))
```

在调⽤ Java 函数时不能使⽤命名参数语法，因为 Java 字节码并不总是保留函数参数的名称。

##### 返回 Unit 的函数

如果⼀个函数不返回任何有⽤的值，它的返回类型是 Unit 。Unit 是⼀种只有⼀个值—— Unit 的类型。这个值不需要显式返回：

```
  fun printHello(name: String?): Unit {
      if (name != null)
          println("Hello ${name}")
      else
          println("Hi there!")
  // `return Unit` 或者 `return` 是可选的
  }
```
Unit 返回类型声明也是可选的。上⾯的代码等同于：

```
  fun printHello(name: String?) { …… }
```

##### 单表达式函数

当函数返回单个表达式时，可以省略花括号并且在 = 符号之后指定代码体即可：

```
  fun double(x: Int): Int = x * 2
```
当返回值类型可由编译器推断时，显式声明返回类型是可选的：

```
  fun double(x: Int) = x * 2
```

##### 显式返回类型

具有块代码体的函数必须始终显式指定返回类型，除⾮他们旨在返回 Unit ，在这种情况下它是可选的。Kotlin 不推断具有块代码体的函数的返回类型，因为这样的函数在代码体中可能有复杂的控制流，并且返回类型对于读者（有时甚⾄对于编译器）是不明显的


##### 可变数量的参数（Varargs）

函数的参数（通常是最后⼀个）可以⽤ vararg 修饰符标记：

```
  fun <T> asList(vararg ts: T): List<T> {
      val result = ArrayList<T>()
      for (t in ts) // ts is an Array
          result.add(t)
      return result
  }
```
允许将可变数量的参数传递给函数：

```
  val list = asList(1, 2, 3)
```
在函数内部，类型 T 的 vararg 参数的可⻅⽅式是作为 T 数组，即上例中的 ts 变量具有类型 Array <out T> 。

只有⼀个参数可以标注为 vararg 。如果 vararg 参数不是列表中的最后⼀个参数，可以使⽤命名参数语法传递其后的参数的值，或者，如果参数具有函数类型，则通过在括号外部传⼀个 lambda

当我们调⽤ vararg -函数时，我们可以⼀个接⼀个地传参，例如 asList(1, 2, 3) ，或者，如果我们已经有⼀个数组并希望将其内容传给该函数，我们使⽤伸展（spread）操作符（在数组前⾯加 * ）：

```
  val a = arrayOf(1, 2, 3)
  val list = asList(-1, 0, *a, 4)
```

##### 中缀表⽰法
标有 `infix` 关键字的函数也可以使⽤中缀表⽰法（忽略该调⽤的点与圆括号）调⽤。中缀函数必须满⾜
以下要求：

- 它们必须是成员函数或扩展函数；
- 它们必须只有⼀个参数；
- 其参数不得接受可变数量的参数且不能有默认值。

```
  infix fun Int.shl(x: Int): Int { …… }
  // ⽤中缀表⽰法调⽤该函数
  1 shl 2
  // 等同于这样
  1.shl(2)
```
中缀函数调⽤的优先级低于算术操作符、类型转换以及 rangeTo 操作符。

中缀函数总是要求指定接收者与参数。当使⽤中缀表⽰法在当前接收者上调⽤⽅法时，需要显式使⽤ this ；不能像常规⽅法调⽤那样省略。这是确保⾮模糊解析所必需的。

```
  class MyStringCollection {
      infix fun add(s: String) { …… }
      fun build() {
          this add "abc" // 正确
          add("abc") // 正确
          add "abc" // 错误：必须指定接收者
      }
  }
```

#### 函数作用域

在Kotlin 中函数可以在⽂件顶层声明，这意味着你不需要像⼀些语⾔如 Java、C# 或 Scala 那样需要创建⼀个类来保存⼀个函数。此外除了顶层函数，Kotlin 中函数也可以声明在局部作⽤域、作为成员函数以及扩展函数。


##### 局部函数

Kotlin ⽀持局部函数，即⼀个函数在另⼀个函数内部：

```
  fun dfs(graph: Graph) {
      fun dfs(current: Vertex, visited: Set<Vertex>) {
          if (!visited.add(current)) return
          for (v in current.neighbors)
              dfs(v, visited)
      } 
      dfs(graph.vertices[0], HashSet())
  }
```
局部函数可以访问外部函数（即闭包）的局部变量，所以在上例中，visited 可以是局部变量：

```
  fun dfs(graph: Graph) {
      val visited = HashSet<Vertex>()
      fun dfs(current: Vertex) {
          if (!visited.add(current)) return
          for (v in current.neighbors)
              dfs(v)
      } 
      dfs(graph.vertices[0])
  }
```

##### 成员函数

成员函数是在类或对象内部定义的函数：

```
  class Sample() {
    fun foo() { print("Foo") }
  }
```
成员函数以点表⽰法调⽤：

```
Sample().foo() // 创建类 Sample 实例并调⽤ foo
```

### 高级函数

```
  fun <T, R> Collection<T>.fold(
      initial: R,
      combine: (acc: R, nextElement: T) -> R
  ): R {
      var accumulator: R = initial
      for (element: T in this) {
          accumulator = combine(accumulator, element)
      }
      return accumulator
  }
```

在上述代码中，参数 combine 具有函数类型 (R, T) -> R ，因此 fold 接受⼀个函数作为参数，该函数接受类型分别为 R 与 T 的两个参数并返回⼀个 R 类型的值。在 for-循环内部调⽤该函数，然后将其返回值赋值给 accumulator 。

为了调⽤ fold ，需要传给它⼀个函数类型的实例作为参数，⽽在⾼阶函数调⽤处，（下⽂详述的）lambda 表达 式⼴泛⽤于此⽬的。


```
  val items = listOf(1, 2, 3, 4, 5)
  // Lambdas 表达式是花括号括起来的代码块。
  items.fold(0, {
  // 如果⼀个 lambda 表达式有参数，前⾯是参数，后跟“->”
      acc: Int, i: Int ->
      print("acc = $acc, i = $i, ")
      val result = acc + i
      println("result = $result")
  // lambda 表达式中的最后⼀个表达式是返回值：
      result
  })
  // lambda 表达式的参数类型是可选的，如果能够推断出来的话：
  val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })
  // 函数引⽤也可以⽤于⾼阶函数调⽤：
  val product = items.fold(1, Int::times)
```

#### 函数类型
Kotlin 使⽤类似 (Int) -> String 的⼀系列函数类型来处理函数的声明：val onClick: () -> Unit = …… 。

这些类型具有与函数签名相对应的特殊表⽰法，即它们的参数和返回值：
- 所有函数类型都有⼀个圆括号括起来的参数类型列表以及⼀个返回类型：(A, B) -> C 表⽰接受类型分别为 A 与 B 两个参数并返回⼀个 C 类型值的函数类型。参数类型列表可以为空，如 ()-> A 。Unit 返回类型不可省略。
- 函数类型可以有⼀个额外的接收者类型，它在表⽰法中的点之前指定：类型 A.(B) -> C 表⽰可以在 A 的接收者对象上以⼀个 B 类型参数来调⽤并返回⼀个 C 类型值的函数。带有接收者的函数字⾯值通常与这些类型⼀起使⽤
- 挂起函数属于特殊种类的函数类型，它的表⽰法中有⼀个 suspend 修饰符 ，例如 suspend () -> Unit 或者 suspend A.(B) -> C 。

挂起函数属于特殊种类的函数类型，它的表⽰法中有⼀个 suspend 修饰符 ，例如 suspend () -> Unit 或者 suspend A.(B) -> C 。

```
如需将函数类型指定为可空，请使⽤圆括号：((Int, Int) -> Int)?。
函数类型可以使⽤圆括号进⾏接合：(Int) -> ((Int) -> Unit)
箭头表⽰法是右结合的，(Int) -> (Int) -> Unit 与前述⽰例等价，但不等于 ((Int) ->
(Int)) -> Unit。
```
还可以通过使⽤类型别名给函数类型起⼀个别称：

```
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

##### 函数类型实例化
有⼏种⽅法可以获得函数类型的实例：

1. 使⽤函数字⾯值的代码块，采⽤以下形式之⼀：
 - lambda 表达式: { a, b -> a + b } ,
 - 匿名函数: fun(s: String): Int { return s.toIntOrNull() ?: 0 }
带有接收者的函数字⾯值可⽤作带有接收者的函数类型的值。

2. 使⽤已有声明的可调⽤引⽤：
 - 顶层、局部、成员、扩展函数：::isOdd 、String::toInt ，
 - 顶层、成员、扩展属性：List<Int>::size ，
 - 构造函数：::Regex
 这包括指向特定实例成员的绑定的可调⽤引⽤：foo::toString 。

使⽤实现函数类型接⼝的⾃定义类的实例：

```
  class IntTransformer: (Int) -> Int {
      override operator fun invoke(x: Int): Int = TODO()
  } 
  val intFunction: (Int) -> Int = IntTransformer()
```
如果有⾜够信息，编译器可以推断变量的函数类型：

```
  val a = { i: Int -> i + 1 } // 推断出的类型是 (Int) -> Int
```

带与不带接收者的函数类型⾮字⾯值可以互换，其中接收者可以替代第⼀个参数，反之亦然。例如，(A,B) -> C 类型的值可以传给或赋值给期待 A.(B) -> C 的地⽅，反之亦然：

```
  val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
  val twoParameters: (String, Int) -> String = repeatFun // OK
  fun runTransformation(f: (String, Int) -> String): String {
      return f("hello", 3)
  }
  val result = runTransformation(repeatFun) // OK
```

##### 函数类型实例调⽤

函数类型的值可以通过其 invoke(……) 操作符调⽤：f.invoke(x) 或者直接 f(x) 。

如果该值具有接收者类型，那么应该将接收者对象作为第⼀个参数传递。调⽤带有接收者的函数类型值的另⼀个⽅式是在其前⾯加上接收者对象，就好⽐该值是⼀个扩展函数：1.foo(2) ，

```
  val stringPlus: (String, String) -> String = String::plus
  val intPlus: Int.(Int) -> Int = Int::plus
  println(stringPlus.invoke("<-", "->"))
  println(stringPlus("Hello, ", "world!"))
  println(intPlus.invoke(1, 1))
  println(intPlus(1, 2))
  println(2.intPlus(3)) // 类扩展调⽤
```


#### Lambda 表达式与匿名函数
lambda 表达式与匿名函数是“函数字⾯值”，即未声明的函数，但⽴即做为表达式传递。考虑下⾯的例⼦：

```
  max(strings, { a, b -> a.length < b.length })
```
函数 max 是⼀个⾼阶函数，它接受⼀个函数作为第⼆个参数。其第⼆个参数是⼀个表达式，它本⾝是⼀个函数，即函数字⾯值，它等价于以下命名函数：

```
fun compare(a: String, b: String): Boolean = a.length < b.length
```

##### Lambda 表达式语法

完整语法形式如下：

```
  val sum = { x: Int, y: Int -> x + y }
```
lambda 表达式总是括在花括号中，完整语法形式的参数声明放在花括号内，并有可选的类型标注，函数体跟在⼀个 -> 符号之后。如果推断出的该 lambda 的返回类型不是 Unit ，那么该 lambda 主体中的最后⼀个（或可能是单个）表达式会视为返回值。

```
  val sum: (Int, Int) -> Int = { x, y -> x + y }
```

将 lambda 表达式传给最后⼀个参数

在 Kotlin 中有⼀个约定：如果函数的最后⼀个参数接受函数，那么作为相应参数传⼊的 lambda 表达式可以放在圆括号之外：

```
  val product = items.fold(1) { acc, e -> acc * e }
```
如果该 lambda 表达式是调⽤时唯⼀的参数，那么圆括号可以完全省略：

```
  run { println("...") }
```
##### it：单个参数的隐式名称

⼀个 lambda 表达式只有⼀个参数是很常⻅的。
如果编译器⾃⼰可以识别出签名，也可以不⽤声明唯⼀的参数并忽略 -> 。该参数会隐式声明为 it ：

```
  ints.filter { it > 0 } // 这个字⾯值是“(it: Int) -> Boolean”类型的
```

##### 从 lambda 表达式中返回⼀个值

我们可以使⽤限定的返回语法从 lambda 显式返回⼀个值。否则，将隐式返回最后⼀个表达式的值。

所以下面是等价的.

```
  ints.filter {
      val shouldFilter = it > 0
      shouldFilter
  } 
  ints.filter {
      val shouldFilter = it > 0
      return@filter shouldFilter
  }
```
⼀约定连同在圆括号外传递 lambda 表达式⼀起⽀持 LINQ-⻛格 的代码：

```
  strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```
下划线⽤于未使⽤的变量

如果 lambda 表达式的参数未使⽤，那么可以⽤下划线取代其名称：

```
  map.forEach { _, value -> println("$value!") }
```
##### 匿名函数

上⾯提供的 lambda 表达式语法缺少的⼀个东西是指定函数的返回类型的能⼒。在⼤多数情况下，这是不必要的。因为返回类型可以⾃动推断出来。然⽽，如果确实需要显式指定，可以使⽤另⼀种语法：匿名函数 。

```
  fun(x: Int, y: Int): Int = x + y
```
匿名函数看起来⾮常像⼀个常规函数声明，除了其名称省略了。其函数体可以是表达式（如上所⽰）或代码块：

```
  fun(x: Int, y: Int): Int {
    return x + y
  }
```
参数和返回类型的指定⽅式与常规函数相同，除了能够从上下⽂推断出的参数类型可以省略：

```
  ints.filter(fun(item) = item > 0)
```
匿名函数的返回类型推断机制与正常函数⼀样：对于具有表达式函数体的匿名函数将⾃动推断返回类型，⽽具有代码块函数体的返回类型必须显式指定（或者已假定为 Unit ）

Lambda表达式与匿名函数之间的另⼀个区别是⾮局部返回的⾏为。⼀个不带标签的 return 语句总是在⽤ fun 关键字声明的函数中返回。这意味着 lambda 表达式中的 return 将从包含它的函数返回，⽽匿名函数中的 return 将从匿名函数⾃⾝返回


##### 闭包

Lambda 表达式或者匿名函数（以及局部函数和对象表达式）可以访问其 闭包 ，即在外部作⽤域中声明的变量。与 Java 不同的是可以修改闭包中捕获的变量：

```
  var sum = 0
  ints.filter { it > 0 }.forEach {
      sum += it
  }
  print(sum)
```

##### 带有接收者的函数字⾯值

带有接收者的函数类型，例如 A.(B) -> C ，可以⽤特殊形式的函数字⾯值实例化—— 带有接收者的函数字⾯值。

Kotlin 提供了调⽤带有接收者（提供接收者对象）的函数类型实例的能⼒。

在这样的函数字⾯值内部，传给调⽤的接收者对象成为隐式的this，以便访问接收者对象的成员⽽⽆需任何额外的限定符，亦可使⽤ this 表达式 访问接收者对象。

这种⾏为与扩展函数类似，扩展函数也允许在函数体内部访问接收者对象的成员。

这⾥有⼀个带有接收者的函数字⾯值及其类型的⽰例，其中在接收者对象上调⽤了 plus ：

```
  val sum: Int.(Int) -> Int = { other -> plus(other) }
```
匿名函数语法允许你直接指定函数字⾯值的接收者类型。如果你需要使⽤带接收者的函数类型声明⼀个变量，并在之后使⽤它，这将⾮常有⽤。

```
  val sum = fun Int.(other: Int): Int = this + other
```

