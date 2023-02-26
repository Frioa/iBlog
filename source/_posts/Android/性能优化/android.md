---
title: 那些年，我做过的性能优化 Android 篇
date: 2023-02-26 15:17:56
tags:
- Android
- Java
- 性能优化
---


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

## 4. 日志
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

