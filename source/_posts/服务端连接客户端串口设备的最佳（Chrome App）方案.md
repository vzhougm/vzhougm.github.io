---
title: 服务端连接客户端串口设备的最佳（Chrome App）方案
date: 2020-09-11 21:57:25
comments: true
categories:
 - 网络
 - 物联网
tags: 
 - 网络
 - 串口
---
## 可选的方案
1. 写个Java程序读取本地串口并通过TCP连接到服务器，打包成jar,放到指定目录，web页面通过nodejs的child_process模块唤起服务。
2. [serialport.io](https://serialport.io/ "serialport.io") 一开始以为可以通过页面直接调用本地串口，呵呵想多了虽是JS控制串口，但人家是运行在服务端的JS，要用的话与上面的操作一样。
3. [chrome app](https://developer.chrome.com/apps) 这个其实本质上和前两个一样，但它直接使用的是Google Chrome的API，所以其实谷歌是可以实现的，只是它没有开放API，只对Chrome App开放。怎么说也更简单了吧，直接欠到浏览器上，相对方便。
4. IE，[MSCOMM32.OCX](https://answers.microsoft.com/en-us/windows/forum/all/missing-file-mscomm32ocx/996447c3-3c1d-4d81-ac58-aa8284c5ed82) 控件。

要是换以前，直接一个 Java applet，轻轻松松搞定。


## Chrome App 的本质
本质就是一系列 html页面

> What Are Chrome Apps?
> Chrome Apps let you use HTML5, CSS, and JavaScript to deliver an experience comparable to a native application.

给大家推荐一个 Chrome 骚操作：[一键把网页变 APP](https://zhuanlan.zhihu.com/p/59692068)

## 开发一个 Chrome App

## 用 Vue 开发 Chrome App

## 通知
```javascript
// 通知
    notify(title, message) {
      notifications.clear('notificationId', () => {
        notifications.create('notificationId', {
          type: "basic",
          title: title,
          iconUrl: "./icons/icon_128.png",
          message: message,
        })
      })
    }
```
## 缓存用户设置
```javascript
// 缓存
storage.sync.set({'ip': that.ip, 'port': parseInt(that.port)})
// 使用
storage.sync.get(['ip', 'port'], _ => {
      this.ip = _.ip
      this.port = parseInt(_.port)
    })
```
## 与服务器通讯
### 打开通道 & 创建连接
```javascript
 // 连接服务
    connect() {
      const that = this
      sockets.tcp.create({}, function (createInfo) {
        sockets.tcp.connect(createInfo.socketId, that.ip, parseInt(that.port), () => {
          const callback = (result) => {
            if (result.resultCode < 0) {
              that.notify("服务器通讯", "连接失败，请确定服务及网络正常！")
            } else {
              that.socketId = parseInt(createInfo.socketId)
              that.notify("服务器通讯", "连接成功")
              // 监听接收信息
              sockets.tcp.onReceive.addListener(that.tcpOnReceive)
            }
          }
          // 发送连接信息
          sockets.tcp.send(createInfo.socketId, new Buffer("connect", "utf-8"), callback)
          // setKeepAlive
          sockets.tcp.setKeepAlive(createInfo.socketId, true, 3, (result) => {
            if (result < 0) {
              that.notify("服务器通讯", "通讯失败，请确定服务及网络正常！")
            }
          })
          // 存储配置
          storage.sync.set({'ip': that.ip, 'port': parseInt(that.port)})
        })
      })
    }
```
### 发送数据
```javascript
// 发送数据到服务器
    sendData2Server(data) {
      const that = this
      that.dataUp = true
      sockets.tcp.send(that.socketId, new Uint8Array(data), _ => {
        if (_.resultCode < 0) {
          that.notify("服务器通讯", "通讯失败，请确定服务及网络正常！")
        }
      })
    }
```
### 监听数据
```javascript
 // TCP下行数据监听
    tcpOnReceive(result) {
      const that = this
      // 数据传输状态灯
      clearTimeout(that.dataTransmissionTimeout)
      that.dataTransmission = true // 通讯标识
      that.dataUp = false // 下行数据
      that.dataTransmissionTimeout = setTimeout(() => {
        that.dataTransmission = false // 通讯标识
      }, 300);
      // const buf2hex = _ => Array.prototype.map.call(new Uint8Array(_), _ => ('00' + _.toString(16)).slice(-2)).join('')
      // console.log('原始数据：' + buf2hex(result.data))
      new Uint8Array(result.data).forEach(_ => this.framing(_))
    }
```
### 关闭连接 & 关闭通道
```javascript
// 断开连接
    disconnect() {
      const that = this
      sockets.tcp.disconnect(that.socketId, () => {
        sockets.tcp.close(that.socketId, () => {
          that.notify("服务器通讯", "断开连接")
          that.socketId = undefined
        })
      })
    }
```

------------


## 定义串口操作指令
这个是Java写的驱动，在tcp server模式下加了个客户端接口。
```java
import java.util.List;

/**
 * 客户端操作
 *
 * @author Rubin
 * @version v1 2020/9/11 13:48
 */

public interface Client {

    default void showDevices(String clientId) {
        // 7F00
        // IReaderDrive.getDevice(clientId).writeAndFlush(new byte[]{127, 0});
    }

    default void open(String clientId, String path) {
        // 7F01
        // IReaderDrive.getDevice(clientId).writeAndFlush(new byte[]{127, 1});
    }

    default void close(String clientId) {
        // 7F02
        // IReaderDrive.getDevice(clientId).writeAndFlush(new byte[]{127, 2});
    }

}

```
Scala调用，效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911155606536.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

🍚真香！短短几行代码完成了调用，一个SDK就此沦为一个工具类哈哈。😂

~~因为基本操作不是频繁的，也是与实际设备通讯的前奏，所以命令越短越好，暂时不考虑丢包问题。如需非常严谨，基本操作指令可以考虑丢包、粘包问题。~~

**靠！实际指令需要考虑丢包、粘包问题。头疼~ 😵**

## 客户端分帧
写了一下，大概长这样。主要思想不变，逻辑根据你实际协议来写。
```javascript
    frame: {
        // 协议头标识
        head: 0,
        // 帧buffer
        buffer: [],
        // 当前帧buff下标
        currentFrameIndex: 0,
        // 数据长度
        dataLen: 0,
        // 初始化
        initialize: () => {
          this.head = 0
          this.buffer = []
          this.currentFrameIndex = 0
          this.dataLen = 0
        }
      }

    /**
     * 分帧
     * @param data byte
     */
    framing(data) {
      const frame = this.frame
      const buffer = frame.buffer
      // 如果head==1，开始读数据
      if (frame.head === 1) {
        buffer[frame.currentFrameIndex++] = data;
        // 如果frameIndex==5，获取dataLen
        if (frame.currentFrameIndex === 5) {
          frame.dataLen = parseInt(this.bytes2Str([buffer[3], buffer[4]]), 16)
          console.log('frame.dataLen => ' + frame.dataLen)
        }
        // 判断是否读完一帧数据
        else if (frame.dataLen === (frame.currentFrameIndex - 7)) {

          console.log('完整的一帧数据')
          console.log(buffer)
          // 重新读取，初始化下标
          frame.head = 0
          frame.buffer = []
          frame.currentFrameIndex = 0
          frame.dataLen = 0

          console.log(frame)
        }
      }
      // 如果不是开头，则判断7F
      else if (data === 127) {
        buffer[frame.currentFrameIndex++] = data;
        // 标记头已经找到
        frame.head = 1;
      }
    }
```
嗯，效果杠杠滴

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911204929316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

## 命令匹配
```javascript
// 匹配命令
    commandMatch(cmd) {
	  // 第3位是命令类别
      switch (cmd[2]) {
        case 12:
          // 获取串口列表
          this.getSerialDevices()
          break
        case 13:
          // 打开指定串口
		  // 第5位是数据长度（其实是4和5都是，但这个命令不会占用第一字节，所以就直接取最后一个字节了。）
          this.connectSerial(this.asciis2Str(cmd.slice(5, 5 + cmd[4])))
          break
        case 14:
          // 关闭串口
          this.disconnectSerial()
          break
        default:
          // 数据传输
          this.sendData2Serial(cmd)
      }
    }
```
## 与串口通信
### 获取串口列表
```javascript
// 获取串口列表
    getSerialDevices() {
      serial.getDevices((ports) => {
        for (let i = 0; i < ports.length; i++) {
          console.log(ports[i])
        }
      })
    }
```

look

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911180551632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
👌okk，完全没问题！

还要与服务器通讯

```javascript
// 获取串口列表
    getSerialDevices() {
      serial.getDevices((ports) => {
        for (let i = 0; i < ports.length; i++) {
          const path = ports[i].path.toUpperCase()
          const pathAscii = new Uint8Array(new Buffer(path))
          // 协议格式：7f 00 0c 00 path.length pathAscii 00 7e
          let data = []
          data[data.length++] = 0x7f
          data[data.length++] = 0
          data[data.length++] = 0x0c
          data[data.length++] = 0
          data[data.length++] = path.length
          pathAscii.forEach(_ => data[data.length++] = _)
          data[data.length++] = 0
          data[data.length++] = 0x7e
          // 发送到服务器
         this.sendData2Server(data)
        }
      })
    },
```
心塞，服务端我还要再加三个协议解析 ┭┮﹏┭┮

```txt
2020-09-11 19:31:02.367  INFO  --- [ntLoopGroup-3-3] c.n.r.s.connect.tcp.server.SocksServer   : Turn on monitoring... 
2020-09-11 19:31:02.367  INFO  --- [ntLoopGroup-3-3] com.nrp.rfid.sdk.IReaderDrive            : New device connection => 192.168.3.2#14766 
设备连接 ===> 192.168.3.2#14766
2020-09-11 19:31:02:368 [write] - 7F 00 0C 00 01 00 0D 7E 
2020-09-11 19:31:02:376 [read ] - 63 6F 6E 6E 65 63 74 
2020-09-11 19:31:02:416 [read ] - 7F 00 0C 00 04 43 4F 4D 33 00 7E 7F 00 0C 00 04 43 4F 4D 31 00 7E 
ResponseEvent ===> ResponseEvent{error=0, parameter=0, type=CLIENT_SHOW_DEVICES, data=[67, 79, 77, 49]}
ResponseEvent ===> ResponseEvent{error=0, parameter=0, type=CLIENT_SHOW_DEVICES, data=[67, 79, 77, 51]}

```
新的协议加到默认解析里去了，偷了个懒~ 🤣

```scala
override def response(event: ResponseEvent): Unit = {
      // println("ResponseEvent ===> " + event.toString)
      event.getType match {
        case CmdType.CLIENT_SHOW_DEVICES =>
          val data = new String(event.getData)
          println(data)
      }
    }
```
```scala
2020-09-11 19:53:47:576 [read ] - 63 6F 6E 6E 65 63 74 7F 00 0C 00 04 43 4F 4D 33 00 7E 7F 00 0C 00 04 43 4F 4D 31 00 7E 
COM3
COM1
```
😭😭😭


### 打开串口
```javascript
// 打开串口
    connectSerial(path) {
      const that = this
      const options = {
        bitrate: 115200
      }
      that.disconnectSerial() // 先断开连接
      serial.connect(path, options, _ => {
        that.serialId = _.connectionId
		// 监听串口上行数据
        serial.onReceive.addListener(that.serialOnReceive)
        // 通知服务器串口打开成功
        let data = [127, 0, 13, 0, 1, 1, 0, 126]
        this.sendData2Server(data)
      })
    },
```
服务端下发命令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911200652337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)


### 断开串口
```javascript
 // 断开串口
    disconnectSerial() {
      if (!this.serialId) {
        return
      }
      serial.disconnect(this.serialId, _ => {
        if (_) {
          // 通知服务器串口断开成功
          let data = [127, 0, 14, 0, 1, 1, 0, 126]
          this.sendData2Server(data)
        } else {
          // 通知服务器串口断开失败
          let data = [127, 0, 14, 0, 1, 0, 0, 126]
          this.sendData2Server(data)
        }
      })
    },
```
### 发送数据
```javascript
// 发送数据到串口
    sendData2Serial(data) {
      const that = this
      serial.send(that.serialId, new Uint8Array(data), _ => {
        console.log('send bytes：' + _.bytesSent)
        serial.flush(that.serialId, _ => {
          if (!_) {
            that.notify("串口通讯", "通讯失败，请确定串口设备连接正常！")
          }
        })
      })
    },
```
## 将数据返回给服务端

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911213731385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

👌，下班 2020-09-11 21:54:55 星期五