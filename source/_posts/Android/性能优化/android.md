---
title: 那些年，我做过的性能优化 Android 篇
date: 2023-02-26 15:54:07
tags:
- Android
- Java
- 性能优化
---

这篇文章记录我工作以来，所做过的优化。

<!--more-->
# Android
## 1. 使用 if , 减少刷新
如：当业务中车速发生变化则刷新 UI ，那么在车速相等的时则不需要刷新 UI 。
```
@Override
public void onSpeedChange(CarPropertyValue speedValue){
    Log.tag(TAG).v("onSpeedChange " + speedValue);
    if (xxx) updateUI();
}
```

## 2. Collections.sort
在 `Android` 中调用 `Collections.sort` 实际上会判断 `SDK` 版本

```
public static <T> void sort(List<T> list, Comparator<? super T> c) {
    // BEGIN Android-changed: List.sort() vs. Collections.sort() app compat.
    // list.sort(c);
    int targetSdkVersion = VMRuntime.getRuntime().getTargetSdkVersion();
    if (targetSdkVersion > 25) {
        list.sort(c);
    } else {
        // Compatibility behavior for API <= 25. 
        ...
    }
    // END Android-changed: List.sort() vs. Collections.sort() app compat.
}
```
优化方案：
1. 如果 `SKD > 25` 我们使用 `list.sort()` 接口。

## 3. Bitmap 优化
Android 中图片是占用内存的大头

优化加载内存：
- 使用 `inJustDecodeBounds、inSampleSize ` 让图片按照 `View` 的大小加载内存
- 如果图片经常更新 `inBitmap` 让图片完成复用
- `BitmapRegionDecoder` 区域加载（大图加载方案）
- `inPreferredConfig` 使用合适的大小加载图片
- 使用 `svg` 矢量图，方便日后业务变化
- `LRU` 管理Bitmap

优化图片大小
- 考试使用合适的图片压缩算法 （PNG、JPG）


## 4. 优化日志库实战

**前置条件：**

-   创建一个 **Demo** 工程，在 **MainActivity** 中只进行打印日志的工作
-   两个线程，每秒打印一次日志
-   统计时间 **1** 分钟

通过 `Android Profile` 模式下观察，这段时间内一共创建了 **0.86MB** 的对象

优化前：
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40d31ec65992438181ac8951925b43b1~tplv-k3u1fbpfcp-watermark.image?)

优化后：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9874f4a5b4b34540b5950ff311381c42~tplv-k3u1fbpfcp-watermark.image?)

结论：

1.  `Char[]` 对象减少创建 `85.6%`

2.  `String` 对象减少创建 `75.4%`

3.  `String[]` 对象减少创建 `100%`

4.  `StringBuilder` 对象减少 `67%`

5.  `ThreadLocalMap` 减少 `100%`

6.  `ArrayList&ltr` 对象减少 `100%`

7.  `byte[]` 数组减少 `100%`

8.  `BaseOneTimeContrl` 减少 `100%`
-   优化前一共创建 `0.86MB （Shallow SIze）`大小的对象，优化后创建了 `0.046MB （Shallow SIze）`对象

-   因此得出结论：优化了 **95%** 的内存占用



优化点：
1. `BasicOnetimeControl` 对象由每次打印创建一次，优化为一个线程拥有一个 `BasicOnetimeControl` ，通过与 `ThreadLocal` 配合保证线程安全。

2. `InnerLog.log()` 接口优化,虽然 `InnerLog` 没有打印的效果，构建参数时使用了 `Java` 中的 `+` 关键字 , 额外创建了 `StringBuffer` 对象，通过将 `if` 判断提前减少对象的创建。

3. 调用 `d(String message, Object... args)` 接口实际上会创建一个 `ArrayList&Itr` 对象,通过添加重载方法避免额外对象的创建。

4. `forEach` 改 `fori` ：
- 使用 `forEach` 时也会创建一个 `ArrayList&Itr` 对象，相比起来 `fori` 中创建一个 `int` 更节约内存，其次不会再堆空间分配内存。
- `fori` 之前判断一个长度，如果长度是 `1` ，没必要进行 `for` 循环

5. 删除 `message.getBytes()` 操作根据源码分析，会根据字符串长度与编码规则创建一个新的 bytes 数组造成空间浪费。
-  在 `PrettyFormatStrategy.log 111` 行的 `message.getBytes()` , 使用字符串本身长度去判断.
-  同时修改了日志最大长度限制由 4000 改为了 2048，因为 Android 日志打印的长度限制是 4000，使用字符串本身长度判断可能是不准确的，因为中文是按照 2-4 字节编码的，为了保证大部分日志不会过长修改了长度大小。
-  对于超过 2038 长度的字符保持原来的打印逻辑不变.

6. 优化 `createMessage()` 方法，如果没有可变参数，提前 `return` 防止代码执行到 `try-catch` 中


7. 每次执行 `System.getProperty("line.separator")` 都会创建 `StringBuffer` ,然而每次得到的结果都一样的，所以改为静态变量。

8. 添加 autoNewLine 是否支持自动换行，每次打印日志都要判断换行是不合理的，应该由用户来控制是否换行。



# Java

## 1. StringBuilder
如果拼接较长的字符串，考虑给 `StringBuilder` 设置一个长度，减少扩容次数。

## 2. 可选参数

Java 中的可选择参数看上去很方便。

实际上可选参数会可能创建一个 `ArrayList$ltr` 对象，如果该接口调用次数频繁则可能产生大量无用的对象。
```
void foo(Integer a, Integer... b) {
}

# 这样会创建一个 `ArrayList$ltr` 对象
foo(1)

# 传入 null 则不会创建对象
foo(1, null)
```
优化方案：
1. 提供重载方法 `void foo(Integer a)`。

## 3. + 操作符
注意下面日志打印语句，我们经常使用 `+` 号完成字符串的拼接。

通过观察成字节码：
实际上是通过创建 `StringBuilder()` 然后通过 `append()` 方法完成

如果这个日志比较频繁打印的话，就要考虑加一些判断减少改操作，
```
 Log.tag(TAG).v("onSpeedChange " + speedValue);
```
优化方案：
- 通过加 `if` 判断减少日志的打印
- 减少无用日志的打印
- 调整日志等级
- 不要使用第三方库日志优化功能
