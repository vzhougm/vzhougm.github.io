---
title: Netty 多协议的动态切换
date: 2020-09-17 10:50:21
comments: true
categories:
 - Netty
tags: 
 - netty
 - 网络
---

## 使用场景

不同设备间不同协议的解析

## Netty常规使用

启动服务前向`Bootstrap`中添加`handler`，启动后数据就安装添加的Handler顺序流动，Handler各自处理自己相关业务。显然在客户端已知的情况下这样是没毛病的，但在客户端未知的情况下，这就有点尴尬了。不同的设备（客户端）又有不同的协议，不同协议就要有不同的解码程序，所以常规的方式此时就不可用了。

## 看看源码

对一个Handler往上走

```
public interface ChannelInboundHandler extends ChannelHandler
```

对ServerBootstrap的 .childHandler(...)往上走

```java
// io.netty.bootstrap.ServerBootstrap
public ServerBootstrap childHandler(ChannelHandler childHandler) {
    this.childHandler = ObjectUtil.checkNotNull(childHandler, "childHandler");
    return this;
}
```

喏，添加的Handler的代码在这，`child.pipeline().addLast(childHandler);`。

这是对应Server端来说这是新连接进来时第一个读到数据的Handler，然后它把配置的childHandler加载到调用链中，然后自己功成身退。

```
@Override
@SuppressWarnings("unchecked")
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    final Channel child = (Channel) msg;

    child.pipeline().addLast(childHandler);

    setChannelOptions(child, childOptions, logger);
    setAttributes(child, childAttrs);

    try {
        childGroup.register(child).addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                if (!future.isSuccess()) {
                    forceClose(child, future.cause());
                }
            }
        });
    } catch (Throwable t) {
        forceClose(child, t);
    }
}
```

测试一下
![dbug](https://img-blog.csdnimg.cn/20200916105527894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

果然不出我所料

dbug了一下，干活的在这里 `AbstractChannelHandlerContext`

```java
private void invokeChannelActive() {
    if (invokeHandler()) {
        try {
            ((ChannelInboundHandler) handler()).channelActive(this);
        } catch (Throwable t) {
            invokeExceptionCaught(t);
        }
    } else {
        fireChannelActive();
    }
}
```

入口在这
![入口](https://img-blog.csdnimg.cn/20200916112217579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

```
/**
 * Register the {@link Channel} of the {@link ChannelPromise} and notify
 * the {@link ChannelFuture} once the registration was complete.
 */
void register(EventLoop eventLoop, ChannelPromise promise);
```

### Netty Server 启动流程（大概）

![启动流程](https://img-blog.csdnimg.cn/20200917110208753.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

### Netty Server 客户端连接流程（大概）

![客户端连接流程](https://img-blog.csdnimg.cn/20200917111920298.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
好吧，以上就单纯看了个源码，了解了下Netty大致的启动过程。接下来就进入主题吧

## 创建一个协议匹配Handler

Netty的Handler是个链表结构的，数据从头到尾依次流过，所以我们只要在头部给它个匹配Handler，后面的数据处理就可以动态切换了。

```java
package rfid.sdk.base;

import rfid.sdk.IReaderDrive;
import rfid.sdk.protocol.r100.frame.coder.R100FrameDecoder;
import rfid.sdk.protocol.r100.frame.coder.R100FrameEncoder;
import rfid.sdk.protocol.r100.frame.coder.R100MonitorFrameDecoder;
import rfid.sdk.protocol.r100.frame.coder.R100MonitorFrameEncoder;
import rfid.sdk.protocol.r100.handler.R100ProtocolHandler;
import rfid.sdk.protocol.r2000.frame.coder.R2000FrameDecoder;
import rfid.sdk.protocol.r2000.frame.coder.R2000FrameEncoder;
import rfid.sdk.protocol.r2000.frame.coder.R2000MonitorFrameDecoder;
import rfid.sdk.protocol.r2000.frame.coder.R2000MonitorFrameEncoder;
import rfid.sdk.protocol.r2000.handler.R2000ProtocolHandler;
import io.netty.channel.*;
import lombok.extern.slf4j.Slf4j;

import java.util.Set;
import java.util.concurrent.Executors;

/**
 * 动态切换Netty Handler
 *
 * @author Rubin
 * @version v1 2020/9/16 10:05
 */
@Slf4j
public class DispatcherHandler extends ChannelInboundHandlerAdapter {

    private final boolean monitor;
    private final Set<MessageCallBack> messageCallBacks;

    public DispatcherHandler(Set<MessageCallBack> messageCallBacks, boolean monitor) {
        this.messageCallBacks = messageCallBacks;
        this.monitor = monitor;
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
        new SelectHandler(ctx).invoke();

        IReaderDrive.addDevice(ctx.channel());
        // 设备连接通知
        MessageCallBack.DeviceConnectionEvent connectionEvent = new MessageCallBack.DeviceConnectionEvent(new Drive(ctx.channel()));
        for (MessageCallBack messageCallBack : messageCallBacks) {
            Executors.newSingleThreadExecutor().execute(() -> MessageCallBack.sendEvent(messageCallBack, connectionEvent));
        }
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        super.channelInactive(ctx);
        IReaderDrive.removeDevice(ctx.channel());
        // 断开连接通知   todo 意外断开需要有重连机制
        MessageCallBack.DeviceDisconnectedEvent disconnectedEvent = new MessageCallBack.DeviceDisconnectedEvent(new Drive(ctx.channel()));
        for (MessageCallBack messageCallBack : messageCallBacks) {
            Executors.newSingleThreadExecutor().execute(() -> MessageCallBack.sendEvent(messageCallBack, disconnectedEvent));
        }
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        super.channelRead(ctx, msg);
        new SelectHandler(ctx).invoke();
    }

    private class SelectHandler {
        private ChannelHandlerContext ctx;

        public SelectHandler(ChannelHandlerContext ctx) {
            this.ctx = ctx;
        }

        public void invoke() {
            ChannelPipeline pipeline = ctx.pipeline();
            String driveKey = IReaderDrive.getDriveKey(ctx.channel());
	    // todo 测试用，先写几个。
            switch (driveKey.substring(0, driveKey.indexOf("#"))) {
                // 手持机
                case "192.168.3.18":
                    log.info("手持机 => " + driveKey);
                    removeR2000Handler(pipeline);
                    addR100Handler(pipeline);
                    break;
                // R100
                case "192.168.3.15":
                    log.info("R100设备 => " + driveKey);
                    removeR2000Handler(pipeline);
                    addR100Handler(pipeline);
                    break;
                // R2000
                case "192.168.3.233":
                    log.info("R2000设备 => " + driveKey);
                    removeR100Handler(pipeline);
                    addR2000Handler(pipeline);
                    break;

                default:
                    System.out.println("未注册的设备 => " + driveKey);
            }
        }

        private void removeR2000Handler(ChannelPipeline pipeline) {
            if (pipeline.context("r2000Decoder") != null) {
                pipeline.remove("r2000Decoder");
                pipeline.remove("r2000Encoder");
                pipeline.remove("r2000Protocol");
            }
        }

        private void addR100Handler(ChannelPipeline pipeline) {
            if (pipeline.context("r100Decoder") == null) {
                pipeline.addLast("r100Decoder", monitor ? new R100MonitorFrameDecoder() : new R100FrameDecoder())
                        .addLast("r100Encoder", monitor ? new R100MonitorFrameEncoder() : new R100FrameEncoder())
                        .addLast("r100Protocol", new R100ProtocolHandler(messageCallBacks));
            }
        }

        private void removeR100Handler(ChannelPipeline pipeline) {
            if (pipeline.context("r100Decoder") != null) {
                pipeline.remove("r100Decoder");
                pipeline.remove("r100Encoder");
                pipeline.remove("r100Protocol");
            }
        }

        private void addR2000Handler(ChannelPipeline pipeline) {
            if (pipeline.context("r2000Decoder") == null) {
                pipeline.addLast("r2000Decoder", monitor ? new R2000MonitorFrameDecoder() : new R2000FrameDecoder())
                        .addLast("r2000Encoder", monitor ? new R2000MonitorFrameEncoder() : new R2000FrameEncoder())
                        .addLast("r2000Protocol", new R2000ProtocolHandler(messageCallBacks));
            }
        }
    }
}
```

## ServerBootstrap

```java
ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    public void initChannel(SocketChannel ch) {
                        ch.pipeline().addLast(
                                new DispatcherHandler(drive.getMessageCallBacks(), config.isMonitor()));
                    }
                });
```

## 在Scala中使用

```scala
package utils

import rfid.sdk.base.{ConnectOption, MessageCallBack}
import rfid.sdk.connect.tcp.server.Client
import rfid.sdk.protocol.r100.CmdType
import rfid.sdk.protocol.r100.event._
import rfid.sdk.protocol.r100.handler.R100MessageCallBack
import rfid.sdk.protocol.r100.query.{Inventory => R100Inventory}
import rfid.sdk.protocol.r2000.event
import rfid.sdk.protocol.r2000.event.{InventoryReplyEvent => _, _}
import rfid.sdk.protocol.r2000.handler.R2000MessageCallBack
import rfid.sdk.protocol.r2000.query.{Inventory => R2000Inventory}
import rfid.sdk.util.HexadecimalUtil
import rfid.sdk.{ReaderDrive => SdkReaderDrive}
import controllers.UserController.{port, port2, stop}

/**
 * @author Rubin
 * @version v1 2020/9/11 15:03
 */
class ReaderDrive {

}

object ReaderDrive extends Client {

  var key = ""

  // 连机（查询设备信息）
  //  def test(): Unit = {
  //    drive.setDriveKey(key)
  //    drive.query(new DeviceInfoQuery)
  //  }
  val drive = new SdkReaderDrive

  def connect: String = {
    drive.closeAll()
// 开启服务    drive.connect(ConnectOption.builder.mode(ConnectOption.Mode.SOCKS_SERVER).port(8300).monitor(false).build())
  }
  // R100设备数据监听
  drive.addMessageCallBack(new R100MessageCallBack {

    override def deviceConnection(event: MessageCallBack.DeviceConnectionEvent): Unit = {
      println("设备连接 ===> " + event.getData.getKey)
      //      key = event.getData.getKey
      //      key.substring(0, key.indexOf("#")) match {
      //        case "192.168.3.15" =>
      //          println("设备连接 ===> R100")
      //          ReaderDrive.showDevices(key)
      //          ReaderDrive.open(key, "COM3")
      //
      //        case "192.168.3.18" =>
      //          println("设备连接 ===> 手持机")
      //
      //        case "192.168.3.233" =>
      //          println("设备连接 ===> R20100")
      //          drive.query(new R2000Inventory(0, 0))
      //      }


    }

    override def deviceDisconnected(event: MessageCallBack.DeviceDisconnectedEvent): Unit = {
      println("设备断开 ===> " + event.getData.getKey)
    }

    override def onGetSoftwareInfo(event: GetSoftwareInfoEvent): Unit = {
      println("获取设备信息 ===> " + HexadecimalUtil.bytes2HexString(event.getData))
    }

    override def onGetTransmitPower(event: GetTransmitPowerEvent): Unit = {
      println("获取发射功率 ===> " + event.getPower)
    }

    override def onInventoryReply(event: InventoryReplyEvent): Unit = {

      event.getDrive.getKey match {
        case key if key.contains("192.168.3.15") =>
          println("++++++R100盘点 ===> " + event.toString)
          ReaderDrive.drive.setDriveKey(s"192.168.3.15#$port2")
          ReaderDrive.drive.query(new R100Inventory())
        case _ => println("-----------手持机盘点 ===> " + event.toString)
      }

    }

    override def onReadTagData(event: ReadTagDataEvent): Unit = {
      println("读取标签数据 ===> " + HexadecimalUtil.bytes2HexString(event.getData))
    }

    override def onWriteTagData(event: WriteTagDataEvent): Unit = {
      println("写标签返回 ===> " + HexadecimalUtil.bytes2HexString(event.getData))
    }

    override def onGetReceiverDemodulatorInfo(event: GetReceiverDemodulatorInfoEvent): Unit = {
      println("获取接收解调器参数 ===> " + event.getDistance)
    }

    override def response(event: ResponseEvent): Unit = {
      println("ResponseEvent ===> " + event.toString)
      event.getType match {
        case CmdType.CLIENT_SHOW_DEVICES =>
          val data = new String(event.getData)
          println(data)
        case CmdType.CLIENT_OPEN_DEVICE =>
          println("串口打开")
          // test() // 连机测试

//          while (true) {
//            TimeUnit.MILLISECONDS.sleep(100)
//            drive.setDriveKey(key)
//            drive.query(new R100Inventory)
//          }

        case CmdType.CLIENT_CLOSE_DEVICE =>
          println("串口关闭")

        case CmdType.ERROR =>
          ReaderDrive.drive.setDriveKey(s"192.168.3.15#$port2")
          ReaderDrive.drive.query(new R100Inventory())

        case _ => None
      }
    }
  })
  // R2000设备数据监听	
  drive.addMessageCallBack(new R2000MessageCallBack {

    override def deviceConnection(deviceConnectionEvent: MessageCallBack.DeviceConnectionEvent): Unit = None

    override def deviceDisconnected(deviceDisconnectedEvent: MessageCallBack.DeviceDisconnectedEvent): Unit = None

    override def onInventoryStart(event: InventoryStartEvent): Unit = {
      println("R2000盘点开始")
    }

    override def onInventoryEnd(event: InventoryEndEvent): Unit = {
      println("R2000盘点结束")
      //val port = 49177
      if (!stop) {
        ReaderDrive.drive.setDriveKey(s"192.168.3.233#$port")
        ReaderDrive.drive.query(new R2000Inventory(0, 0))
      }
    }

    override def onInventoryReply(event2: event.InventoryReplyEvent): Unit = {
      println("*******R2000数据 => " + HexadecimalUtil.bytes2HexString(event2.getEpc))
    }

    override def onConfigReadReply(event: ConfigReadReplyEvent): Unit = {

    }

    override def onCardOperateReply(event: CardOperateReplyEvent): Unit = {

    }

    override def onReadEnd(event: ReadEndEvent): Unit = {

    }

    override def onWriteEnd(event: WriteEndEvent): Unit = {

    }

    override def onEnd(event: BaseEndEvent[_]): Unit = {

    }

  })
}
```

## 效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917141207430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

设备

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917143831183.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
