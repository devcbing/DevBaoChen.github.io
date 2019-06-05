# 协程

<a name="sFdaN"></a>
### 什么是协程
协程是比线程更轻量的，有状态，可暂停可恢复的任务单元。

对于操作系统而言，进程是运行每个程序的任务单元。每个应用程序都在自己的进程中运行，状态和数据相互隔离，稳定运行；一个程序崩溃了不会影响其他程序运行。这些程序是并发运行的。<br />对于进程而言，为了提高程序的运行速度，我们会将一些耗时的任务分离为更小的任务单元，就是线程。多个线程并发工作，能大大加快整体任务的执行速度。

既然进程和线程都能通过并发执行提高运行效率，那协程有什么优势呢？

- 更小的内存开销。进程和线程的内存开销比较大；
- 没有上CPU下文切换带来的性能开销。线程和进程由CPU来调度执行，每个CPU会执行多个线程，每当切换新线程时，需要先存储当前线程的状态，再加载新线程的状态；在频繁的调度下，切换线程消耗CPU的很多性能。而协程由应用程序控制状态的切换，性能开销要小很多。

虽然多线程也能很好进行并发编程，但协程的并发会消耗更少的资源，有更高的调度性能。这对于服务器处理高并发的场景会带来很大的优势

<a name="tZjx9"></a>
### 协程是如何实现的

目前我所知道的支持协程的语言有Python, NodeJs，Go和Kotlin。简单讲，原理大都是**OS Thread Pool**配合状态机来实现的。协程底层仍然是靠线程池调度，靠状态机来维护状态。

Kotlin官网的动态图，足够说明一切：<br />![](https://cdn.nlark.com/yuque/0/2019/gif/354817/1559632648223-1d19b219-6d89-4ab9-9a89-28eea36e2b92.gif#align=left&display=inline&height=336&originHeight=336&originWidth=997&size=0&status=done&width=997)

由于协程可暂停和可恢复的特性，能直接消除异步回调，让我们用同步写法编写异步执行代码。很多编程语言在处理异步任务结果时都采用Callback的方式，比如早期的JavaScript。当逻辑复杂的时候，很容易陷入**回调地域**，导致代码可读性差，可维护性低。

```kotlin
fun main() {
    GlobalScope.launch { 
        var url = "https://www.baidu.com"
        //等待异步请求返回，无需Callback
        var result = request(url).await()
        println("请求结果为：$result")
    }
}
```
协程有以下几个有点：

- 更少的资源消耗和更高的调度性能
- 用同步的方式写异步代码，可读性更好
- 协程比线程更容易使用，不需要关心过多的状态，直接编写逻辑即可

<a name="ubRjW"></a>
### 协程的实现
在 kotlin 中，协程不是其标准库，所以需要使用下面的方式引入

```kotlin
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.2.0'
}
```

编写一个协程程序，并在协程中延时1秒：

```kotlin
fun main() {
    // 在后台启动一个新的协程
    GlobalScope.launch {
        delay(1000L) // 挂起当前协程，但并不会阻塞程序，1秒后恢复执行
        println("World!") //延迟1秒后打印
    }
    println("Hello,") // 会立即打印，协程的delay并不会阻塞程序
    Thread.sleep(2000L) // 阻塞主线程 2 秒钟来保证 JVM 存活，否则的话协程还未恢复执行，进程就退出了
}

//输出
Hello,
World!
```

可以看到开启协程很简单，我们不用关心哪个线程在调度协程，也不用关心协程的状态，只需要专心编写我们的异步逻辑即可。<br />`delay`是一个`suspend`关键字修饰的挂起函数，会暂停当前协程的执行，但并不阻塞主线程往下进行；等时间到，便恢复执行。

<a name="TEoft"></a>
### 主协程
由于上面的协程无法阻塞住当前线程，我们使用`Thread.sleep()`来阻塞线程，使得协程有机会得到执行。Kotlin提供了一个特殊的**主协程**可以阻塞主线程：

```kotlin
fun main() = runBlocking { //开启主协程
    GlobalScope.launch { //开启子协程
        delay(1000L) // 挂起当前协程，但并不会阻塞程序，1秒后恢复执行
        println("World!") //延迟1秒后打印
    }
    println("Hello,") // 会立即打印，协程的delay并不会阻塞程序
}
```

`runBlocking`开启的为主协程，由于`GlobalScope.launch`是在一个协程中开启协程，因此我们叫它子协程。<br />但是上面的`World`仍然不会得到执行，因为**主协程**瞬间就执行完毕，并不会等待`GlobalScope`开启的子协程执行完成才结束。**主协程**一旦结束，主线程就执行结束，整个程序就结束。<br />有两种方式可以让**主协程**等待子线程执行完成才结束：一种是使用`delay`函数挂起**主协程**，另一种是让子协程`join`到**主协程**中。<br />先看第一种，使用`delay`函数挂起**主协程**，挂起的时间要大于子协程挂起的时间：

```kotlin
fun main() = runBlocking { 
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    delay(2000) //挂起主协程，等待子协程执行完毕
}
//输出
Hello,
World!
```

另外一种，使用一个变量记住`GlobalScope.launch`开启的协程的引用：

```kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() //等待子协程执行完才结束
}
//输出
Hello,
World!
```

看起来，使用`join`方法更加优雅。

<a name="zetSO"></a>
### 协程存活期
继续上面的例子，我们刚才得出`GlobalScope.launch`开启的子协程并不能阻塞主它的父协程。但仔细想想这不合理。<br />假设逻辑再复杂一些，在刚才的主协程中，我们开启5个子协程。那就必须手动持有5个子协程的引用，否则无法保证让每个协程得到执行。如果我们忘记持有某个协程的引用，那么这个协程的代码就报废了，因为无法得到执行。如果真的是这样的话，那出错的概率还是很大的。难道父协程不能自动的等所有子协程执行完毕才结束吗？其实是可以的。<br />为什么上面的例子不行呢？每个协程是有存活期的，在一个协程中开启的子协程的存活期一般不会超过其父协程的存活期。但是`GlobalScope`比较特殊，它开启的是顶级协程。顶级协程的存活期由整个应用程序管理，并不受主协程限制，相当于直辖市。顶级协程虽然在主协程内部开启，但是在存活期和作用域上和主协程平级，因此它无法阻塞主协程，需要我们手动的`join`或者`delay`主协程。<br />每个协程对象都是`CoroutineScope`实例，`CoroutineScope`有个`launch`方法用来在自己的作用域内开启一个受自己管辖的子协程。而且会自动的等所有子协程执行完毕才结束。将上面的例子稍做改动就可以：

```kotlin
fun main() = runBlocking {
    //去掉了GlobalScope
    val job = launch { //在自己的作用域内开启子协程
        delay(1000L)
        println("World!")
    }
    println("Hello,")
//    job.join() 无需join了
}
//输出
Hello,
World!
```

Kotlin不建议我们直接使用`GlobalScope`开启顶级协程，通常应该直接使用`launch`方法在自己的作用域内开启子协程，这样不容易出错。

<a name="z8aiF"></a>
### 取消与超时

协程通常用来执行耗时操作。 在Android开发中，我们在一个界面开启协程进行耗时请求。假如此时用户关闭了界面，那么协程的执行结果已经不需要了，因此协程应该是可以被取消的。<br />协程提供了`cancel()`方法来取消：

```kotlin
fun main() = runBlocking {
    val job = launch {
        println("i am working...")
        delay(2000L)
        println("work done!") //将不会输出
    }
    delay(1000)
    job.cancel() //取消协程
}
```

有时候耗时操作的时间是不确定的，比如在Android发起一个网络请求，我们并不确定它什么时候会返回，因此超时的处理是必要的。我们假设如果请求超过10秒钟未返回结果，用户已经没有耐心等待了，此时应该结束这个协程了。<br />使用`withTimeout`来开启带超时限制的协程：

```kotlin
withTimeout(5000){
    println("start request")
    delay(120000) //延时12秒，模拟巨慢的弱网环境
    println("get result!")
}
```
协程的超时会抛出`TimeoutCancellationException`异常。如果你不喜欢抛出异常的方式，可以使用`withTimeoutOrNull`的方式开启协程，如果协程超时则返回null，这样就不再有异常了。

```kotlin
val result = withTimeoutOrNull(5000){
    println("start request")
    delay(120000) //延时12秒，模拟巨慢的弱网环境
    println("get result!")
}
println(result) //null
```

<a name="Esqc0"></a>
### suspend函数

使用`suspend`修饰的函数叫做挂起函数，`delay`就是一个挂起函数。由于我们不可能将所有异步逻辑都写到协程中，必然要重构和抽取。比如：

```kotlin
val job = launch { 
    //执行网络请求
    var result = doRequest() 
    println(result)
}
fun doRequest(): String{
    return "请求的结果"
}
```

假设所有的耗时请求都抽取到`doRequest`方法中，但是普通的方法并不能挂起协程，所以`doRequest()`无法阻塞住`println()`。给函数添加`suspend`修饰符即可：

```kotlin
suspend fun doRequest(): String{
    delay(2000) //模拟请求耗时2秒
    return "请求的结果"
}
```

<a name="ci13T"></a>
### 协程的并发执行

```kotlin
suspend fun doRequest1(): Int{
    delay(2000)
    return 1
}
suspend fun doRequest2(): Int{
    delay(2000)
    return 2
}
val totalTime = measureTimeMillis {
    doRequest1()
    doRequest2()
}
println("totalTime: $totalTime") // totalTime: 4009
```

为了提高执行效率，我们希望两个耗时操作是并发执行的。使用`async`就可以做到：

```kotlin
val totalTime = measureTimeMillis {
    val result1 = async { doRequest1() }
    val result2 = async { doRequest2() }
    println("result: ${result1.await() + result2.await()}") //result: 3
}
println("totalTime: $totalTime") //totalTime: 2032
```

`async`开启一个特殊的协程，能够与其他协程并发工作。它返回一个`Deferred`对象，该对象可以通过`await()`来等待异步执行的结果；同时`Deferred`对象也是一个`Job`对象，可以`cancel()`掉。<br />上面的`async`代码块一旦执行，协程就开始工作了。有时候我们希望满足某些条件下，协程在开始工作。那么可以这样使用`懒惰的async`：

```kotlin
val totalTime = measureTimeMillis {
    val result1 = async(start = CoroutineStart.LAZY) { doRequest1() } //只是创建协程对象，并未开始工作
    val result2 = async(start = CoroutineStart.LAZY) { doRequest2() } //只是创建协程对象，并未开始工作

    //满足条件了才执行
    result1.start() //协程开始执行
    result2.start() //协程开始执行
    println("result: ${result1.await() + result2.await()}")
}
println("totalTime: $totalTime")
```

<a name="7nNVe"></a>
### 异常处理

协程中的逻辑有可能遇到异常，如果我们不处理，他们则默认向上传播给调度线程，从而导致程序崩溃：

```kotlin
fun main() = runBlocking {
    launch {
        throw ArrayIndexOutOfBoundsException()
    }
    launch {
        throw IllegalArgumentException()
    }
    println("start...")
}
```

上面的程序在遇到第一个协程抛出的`ArrayIndexOutOfBoundsException`时就会终止执行。我们除了在每个协程代码块中进行`try/catch`之外，也可以设置一个全局的异常处理器。<br />由于协程最终由线程调度，所有未处理的异常最终都会抛给线程，因此给线程设置全局的异常处理器即可：

```kotlin
fun main() = runBlocking {
    Thread.setDefaultUncaughtExceptionHandler { t, e ->
        println("catch exception: $e")
    }
    GlobalScope.launch {
        throw ArrayIndexOutOfBoundsException()
    }.join()
    launch {
        throw IllegalArgumentException()
    }
    println("start...")
}
//输出
catch exception: java.lang.ArrayIndexOutOfBoundsException
start...
catch exception: java.lang.IllegalArgumentException
```
<br />
<a name="FHb7l"></a>
### 协程并发安全问题
当我们使用多线程对同一个共享数据进行修改时，很可能遇到线程安全问题。协程本质上仍然由线程调度执行，所以协程并发执行时，也有和线程类似的安全问题。来看一段代码：

```kotlin
fun main() = runBlocking {
    var n = 0
    val list = mutableListOf<Job>()
    repeat(100) {
        list.add(GlobalScope.launch {
            repeat(100) { n++ }
        })
    }
    list.forEach {
        it.join()
    }
    println("n: $n")
}
```

这段代码重复添加100个协程对象，每个协程执行100次++，共执行10000次++操作。运行结果很可能不是10000，可以多次运行看看：

```kotlin
n: 9495
```

> 如果你的机器只有不超过2个CPU，你将总是看到10000。因为此时线程池只有一个线程来调度协程，不会出现并发安全问题。


在线程遇到安全问题时我们一般有2种处理方案：一种是加锁，另外一种是使用线程安全的数据结构。<br />加锁往往会降低效率，因此我们推荐采用第二种方案。JDK提供了大量线程安全的数据结构，我们使用`AtomicInteger` 来改写代码：

```kotlin
var n = AtomicInteger()
val list = mutableListOf<Job>()
repeat(100) {
    list.add(GlobalScope.launch {
        repeat(100) { n.incrementAndGet() }
    })
}
list.forEach {
    it.join()
}
println("n: $n")
```
无论运行多少次，你将总是得到10000。<br />Kotlin官方文档为协程并发安全提供了多种解决方案，其中使用线程安全的数据结构是效率最高的方案，这些数据结构由JDK常年迭代进行超细粒度的优化，直接使用即可。<br />在Android开发中，协程一般用来代替线程执行耗时任务，更有用的是它可以用同步的方式编写异步代码，能够将复杂的异步逻辑变的极具可读性。具体使用时协程配合强大的高阶函数，已经成为事实上的线程调度的最佳方案，RxJava已经没有存在的必要。
