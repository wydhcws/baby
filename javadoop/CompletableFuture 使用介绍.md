---
name: completable-future
title: CompletableFuture 使用介绍
date: 2019-04-18 16:29:57
tags: 
categories: 
---
本文安利一个 Java8 的工具 `CompletableFuture`，这是 Java8 带来的一个非常好用的用于异步编程的类。还没使用过的小伙伴，赶紧用起来吧。

本文不介绍它的实现源码，仅介绍它的接口使用，本文也不做它和 RxJava 等其他异步编程框架的对比。

## 实例化

首先，不管我们要做什么，我们第一步是需要构造出 CompletableFuture 实例。

最简单的，我们可以通过构造函数来进行实例化：

```java
CompletableFuture<String> cf = new CompletableFuture<String>();
```

这个实例此时还没有什么用，因为它没有实际的任务，我们选择结束这个任务：

```java
// 可以选择在当前线程结束，也可以在其他线程结束
cf.complete("coding...");
```

因为 CompletableFuture 是一个 Future，我们用 `String result = cf.get()` 就能获取到结果了。

> CompletableFuture 提供了 `join()` 方法，它的功能和 get() 方法是一样的，都是阻塞获取值，它们的区别在于 join() 抛出的是 unchecked Exception。

上面的代码确实没什么用，下面介绍几个 **static** 方法，它们使用任务来实例化一个 CompletableFuture 实例。

```java
CompletableFuture.runAsync(Runnable runnable);
CompletableFuture.runAsync(Runnable runnable, Executor executor);

CompletableFuture.supplyAsync(Supplier<U> supplier);
CompletableFuture.supplyAsync(Supplier<U> supplier, Executor executor)
```

- runAsync 方法接收的是 Runnable 的实例，意味着它没有返回值
- supplyAsync 方法对应的是有返回值的情况
- 这两个方法的带 **executor** 的变种，表示让任务在指定的线程池中执行，不指定的话，通常任务是在 `ForkJoinPool.commonPool()` 线程池中执行的。

好的，现在我们已经有了第一个 CompletableFuture 实例了，我们来看接下来的内容。

## 任务之间的顺序执行

我们先来看执行两个任务的情况，首先执行任务 A，然后将任务 A 的结果传递给任务 B。

其实这里有很多种情况，任务 A 是否有返回值，任务 B 是否需要任务 A 的返回值，任务 B 是否有返回值，等等。有个明确的就是，肯定是任务 A 执行完后再执行任务 B。

我们用下面的 6 行代码来说：

```java
CompletableFuture.runAsync(() -> {}).thenRun(() -> {}); 
CompletableFuture.runAsync(() -> {}).thenAccept(resultA -> {}); 
CompletableFuture.runAsync(() -> {}).thenApply(resultA -> "resultB");

CompletableFuture.supplyAsync(() -> "resultA").thenRun(() -> {});
CompletableFuture.supplyAsync(() -> "resultA").thenAccept(resultA -> {});
CompletableFuture.supplyAsync(() -> "resultA").thenApply(resultA -> resultA + " resultB");
```

前面 3 行代码演示的是，任务 A 无返回值，所以对应的，第 2 行和第 3 行代码中，resultA 其实是 `null`。

第 4 行用的是 `thenRun(Runnable runnable)`，任务 A 执行完执行 B，并且 B 不需要 A 的结果。

第 5 行用的是 `thenAccept(Consumer action)`，任务 A 执行完执行 B，B 需要 A 的结果，但是任务 B 不返回值。

第 6 行用的是 `thenApply(Function fn)`，任务 A 执行完执行 B，B 需要 A 的结果，同时任务 B 有返回值。

这一小节说完了，如果任务 B 后面还有任务 C，往下继续调用 `.thenXxx()` 即可。

### 异常处理

说到这里，我们顺便来说下 CompletableFuture 的异常处理。这里我们要介绍两个方法：

```java
public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn);
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
```

看下面的代码：

```java
CompletableFuture.supplyAsync(() -> "resultA")
    .thenApply(resultA -> resultA + " resultB")
    .thenApply(resultB -> resultB + " resultC")
    .thenApply(resultC -> resultC + " resultD");
```

上面的代码中，任务 A、B、C、D 依次执行，如果任务 A 抛出异常（当然上面的代码不会抛出异常），那么后面的任务都得不到执行。如果任务 C 抛出异常，那么任务 D 得不到执行。

那么我们怎么处理异常呢？看下面的代码，我们在任务 A 中抛出异常，并对其进行处理：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException();
})
        .exceptionally(ex -> "errorResultA")
        .thenApply(resultA -> resultA + " resultB")
        .thenApply(resultB -> resultB + " resultC")
        .thenApply(resultC -> resultC + " resultD");

System.out.println(future.join());
```

上面的代码中，任务 A 抛出异常，然后通过 `.exceptionally()` 方法处理了异常，并返回新的结果，这个新的结果将传递给任务 B。所以最终的输出结果是：

```java
errorResultA resultB resultC resultD
```

再看下面的代码，我们来看下另一种处理方式，使用 `handle(BiFunction fn)` 来处理异常：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "resultA")
        .thenApply(resultA -> resultA + " resultB")
        // 任务 C 抛出异常
        .thenApply(resultB -> {throw new RuntimeException();})
        // 处理任务 C 的返回值或异常
        .handle(new BiFunction<Object, Throwable, Object>() {
            @Override
            public Object apply(Object re, Throwable throwable) {
                if (throwable != null) {
                    return "errorResultC";
                }
                return re;
            }
        })
        .thenApply(resultC -> resultC + " resultD");

System.out.println(future.join());
```

上面的代码使用了 `handle` 方法来处理**任务 C** 的执行结果，上面的代码中，`re` 和 `throwable` 必然有一个是 null，它们分别代表正常的执行结果和异常的情况。

> 当然，它们也可以都为 null，因为如果它作用的那个 CompletableFuture 实例没有返回值的时候，re 就是 null。

## 取两个任务的结果

上面一节，我们说的是，任务 A 执行完 -> 任务 B 执行完 -> 执行任务 C，它们之间有先后执行关系，因为后面的任务依赖于前面的任务的结果。

这节我们来看怎么让任务 A 和任务 B 同时执行，然后取它们的结果进行后续操作。这里强调的是任务之间的并行工作，没有先后执行顺序。

如果使用 Future 的话，我们通常是这么写的：

```java
ExecutorService executorService = Executors.newCachedThreadPool();

Future<String> futureA = executorService.submit(() -> "resultA");
Future<String> futureB = executorService.submit(() -> "resultB");

String resultA = futureA.get();
String resultB = futureB.get();
```

接下来，我们看看 CompletableFuture 中是怎么写的，看下面的几行代码：

```java
CompletableFuture<String> cfA = CompletableFuture.supplyAsync(() -> "resultA");
CompletableFuture<String> cfB = CompletableFuture.supplyAsync(() -> "resultB");

cfA.thenAcceptBoth(cfB, (resultA, resultB) -> {});
cfA.thenCombine(cfB, (resultA, resultB) -> "result A + B");
cfA.runAfterBoth(cfB, () -> {});
```

第 3 行代码和第 4 行代码演示了怎么使用两个任务的结果 resultA 和 resultB，它们的区别在于，`thenAcceptBoth` 表示后续的处理不需要返回值，而 `thenCombine` 表示需要返回值。

如果你不需要 resultA 和 resultB，那么还可以使用第 5 行描述的 `runAfterBoth` 方法。

> 注意，上面的写法和下面的写法是没有区别的：
>
> ```java
> CompletableFuture<String> cfA = CompletableFuture.supplyAsync(() -> "resultA");
> 
> cfA.thenAcceptBoth(CompletableFuture.supplyAsync(() -> "resultB"),
>         (resultA, resultB) -> {});
> ```
> 千万不要以为这种写法任务 A 执行完了以后再执行任务 B。

## 取多个任务的结果

接下来，我们将介绍两个非常简单的静态方法：allOf() 和 anyOf() 方法。

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs){...}
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) {...}
```

这两个方法都非常简单，简单介绍一下。

```java
CompletableFuture cfA = CompletableFuture.supplyAsync(() -> "resultA");
CompletableFuture cfB = CompletableFuture.supplyAsync(() -> 123);
CompletableFuture cfC = CompletableFuture.supplyAsync(() -> "resultC");

CompletableFuture<Void> future = CompletableFuture.allOf(cfA, cfB, cfC);
// 所以这里的 join() 将阻塞，直到所有的任务执行结束
future.join();
```

由于 allOf 聚合了多个 CompletableFuture 实例，所以它是没有返回值的。这也是它的一个缺点。

anyOf 也非常容易理解，就是只要有任意一个 CompletableFuture 实例执行完成就可以了，看下面的例子：

```java
CompletableFuture cfA = CompletableFuture.supplyAsync(() -> "resultA");
CompletableFuture cfB = CompletableFuture.supplyAsync(() -> 123);
CompletableFuture cfC = CompletableFuture.supplyAsync(() -> "resultC");

CompletableFuture<Object> future = CompletableFuture.anyOf(cfA, cfB, cfC);
Object result = future.join();
```

最后一行的 join() 方法会返回最先完成的任务的结果，所以它的泛型用的是 Object，因为每个任务可能返回的类型不同。

### either 方法

如果你的 anyOf(...) 只需要处理两个 CompletableFuture 实例，那么也可以使用 `xxxEither()` 来处理，

```java
cfA.acceptEither(cfB, result -> {});
cfA.acceptEitherAsync(cfB, result -> {});
cfA.acceptEitherAsync(cfB, result -> {}, executorService);

cfA.applyToEither(cfB, result -> {return result;});
cfA.applyToEitherAsync(cfB, result -> {return result;});
cfA.applyToEitherAsync(cfB, result -> {return result;}, executorService);

cfA.runAfterEither(cfA, () -> {});
cfA.runAfterEitherAsync(cfB, () -> {});
cfA.runAfterEitherAsync(cfB, () -> {}, executorService);
```

上面的各个带 **either** 的方法，表达的都是一个意思，指的是两个任务中的其中一个执行完成，就执行指定的操作。它们几组的区别也很明显，分别用于表达是否需要任务 A 和任务 B 的执行结果，是否需要返回值。

大家可能会对这里的几个变种有盲区，这里顺便说几句。

1、`cfA.acceptEither(cfB, result -> {});` 和 `cfB.acceptEither(cfA, result -> {});` 是一个意思；

2、第二个变种，加了 **Async** 后缀的方法，代表将需要执行的任务放到 ForkJoinPool.commonPool() 中执行(非完全严谨)；第三个变种很好理解，将任务放到指定线程池中执行；

3、难道第一个变种是同步的？不是的，而是说，它由任务 A 或任务 B 所在的执行线程来执行，取决于哪个任务先结束。

## 小结

关于 CompletableFuture，本文已经覆盖了绝大多数的接口。如果还有不了解的接口，欢迎各位在留言板中进行交流。

（全文完）