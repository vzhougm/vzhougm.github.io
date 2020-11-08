---
title: Netty 执行命令带 Future返回
date: 2020-09-23 13:39:13
comments: true
categories:
 - Netty
tags: 
 - netty
 - 网络
---
## Netty常规操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092313002714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
## Netty 带`Future<T>`返回的操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923130647321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
Netty 带`Future<T>`返回的操作的使用场景主要就是需要在同一个方法内获取执行结果的操作，如设备连机。当然第一种操作也可以，但是因为不在同一个线程内，操作起来比较麻烦。一起来看看怎么做吧！

我的Netty执行方法写在 BusHandler 类中

```java
package rfid.sdk.base;

import rfid.sdk.IReaderDrive;
import io.netty.channel.Channel;
import io.netty.channel.ChannelId;
import io.netty.util.concurrent.DefaultPromise;
import io.netty.util.concurrent.Future;

import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;


/**
 * @author Rubin
 * @version v1 2020/7/8 11:31
 */
public class BusHandler {

    public final static ConcurrentMap<ChannelId, DefaultPromise<?>> PROMISE_CACHE = new ConcurrentHashMap<>();

    /**
     * 执行命令
     *
     * @param cqr 原始命令
     */
    public static void execute(CommandQueryRoot cqr) {
        IReaderDrive.getDevice(cqr.getDeviceKey()).writeAndFlush(cqr.getFrameArray());
    }

    /**
     * 执行命令
     *
     * @param cqr 原始命令
     */
    public static <T> Future<T> submit(CommandQueryRoot cqr) {
        Channel device = IReaderDrive.getDevice(cqr.getDeviceKey());
        device.writeAndFlush(cqr.getFrameArray());

        DefaultPromise<T> promise = new DefaultPromise<>(device.eventLoop());
        PROMISE_CACHE.put(device.id(), promise);

        return promise;
    }
}
```
就是这个方法了`public static <T> Future<T> submit(CommandQueryRoot cqr)`，使用Netty 的 DefaultPromise，我们走上去看看。

```java
public class DefaultPromise<V> extends AbstractFuture<V> implements Promise<V> {
public interface Promise<V> extends Future<V> {
public interface Future<V> extends java.util.concurrent.Future<V> {
```
### 创建并缓存 Promise
每次submit执行都会创建一个 DefaultPromise 与其通道ID绑定，然后缓存起来

```java
DefaultPromise<T> promise = new DefaultPromise<>(device.eventLoop());
PROMISE_CACHE.put(device.id(), promise);
```
**其实这里不严谨，如果同一设备，用此方法同时执行了多条命令，之前的命令没回来的话就会被后面的覆盖掉！所以缓存key最好还是加个命令ID去，即通道ID+命令ID。**

### Promise 塞值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923132359900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

如果加了命令ID就这样

```java
DefaultPromise promise = BusHandler.PROMISE_CACHE.get(channel.id().asLongText() + cmdType.name());
```
对于Promise 来说严格意义上应该只使用一次，不可重复使用！上面的`case INVENTORY:` 是可能多次触发的，所以注释掉了这个的回调。

### 使用（Scala）

```scala
def evenMachine(dType: String, ipKey: String) = {

    dType match {
      case "r100" =>

        cache.get[String](ipKey) onComplete {
          case Success(driveKey) =>
            drive.setDriveKey(driveKey.get)
            val future = drive.submit[GetSoftwareInfoEvent](new DeviceInfoQuery)
            val event = future.get(5, TimeUnit.SECONDS)
            println(event.getData.mkString("Array(", ", ", ")"))
        }


      case "r2000" =>
        drive.execute(new ReadDeviceId)

      case "handset" => None
      case _ => None
    }

  }
```
look

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923133339605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

## 总结
以上仅提供一种问题决绝方式，上面的代码只能做为小规模的方法使用，如果想所有的方法都这样做的话，缓存需要设计的好，特别是多设备，多命令的情况下！
