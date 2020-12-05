---
title: Android TCP通讯
date: 2020-09-14 18:52:42
comments: true
categories:
 - 网络
 - 物联网
tags: 
 - 网络
 - 通讯
---
# Android通过USB线与本地服务通讯

## 思路

USB线通讯的本质是tcp通讯，通过adb转发数据。

所以首先服务端软件需要配置adb。写一个脚本，在准备连接之前调用一下

## adb脚本

```shell
adb shell am broadcast -a NotifyServiceStop
adb forward tcp:5000 tcp:13000
adb shell am broadcast -a NotifyServiceStart
```

## 设备端开启Socket服务

## 服务端去连接设备开启的Socket服务

## 数据传输

---



# Android通过Wifi与本地服务通讯

## 服务端开启Socket服务

## 设备端去连接服务端开启的Socket服务


# 简单的代码

```java
package com.example.rederdemo;

import android.util.Log;

import com.gg.reader.api.protocol.gx.LogBaseEpcInfo;

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.Executors;

public class SocketServerThread extends Thread {

    private BufferedOutputStream out;
    private static Socket client;

    @Override
    public void run() {
        try {
            // 发送数据到本地服务
            Executors.newSingleThreadExecutor().execute(() -> {
                while (true) {
                    try {
                        if (out != null) {
                            LogBaseEpcInfo epcInfo = MainActivity.tagEpcQueue.take();
                            out.write(epcInfo.getbEpc());
                            out.flush();
                        }
                    } catch (InterruptedException | IOException e) {
                        e.printStackTrace();
                    }
                }
            });
    
            Log.e("ws", "等待连接");
            System.out.println("---------socket 通信线程----等待连接");
            ServerSocket serverSocket = new ServerSocket(13000);
            while (true) {
                client = serverSocket.accept();
                out = new BufferedOutputStream(client.getOutputStream());
                // 开启子线程去读去数据
                new Thread(new SocketReadThread(new BufferedInputStream(client.getInputStream()))).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    class SocketReadThread implements Runnable {

        private BufferedInputStream in;

        public SocketReadThread(BufferedInputStream inStream) throws IOException {
            this.in = inStream;
        }

        public void run() {
            try {
                String readMsg;
                while (true) {
                    try {
                        if (!client.isConnected()) {
                            break;
                        }
                        // 读到后台发送的消息
                        readMsg = readMsgFromSocket(in);
                        if (readMsg.length() == 0) {
                            break;
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
                in.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        //读取PC端发送过来的数据
        private String readMsgFromSocket(BufferedInputStream in) {
            String msg = "";
            byte[] temp = new byte[1024];
            try {
                int readedBytes = in.read(temp, 0, temp.length);
                msg = new String(temp, 0, readedBytes, StandardCharsets.UTF_8);
                System.out.println(msg);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return msg;
        }
    }

    public static Thread wifi() {
        return new Thread(() -> {
            // 发送数据到本地服务
            Executors.newSingleThreadExecutor().execute(() -> {
                while (true) {
                    try {
                        if (client != null) {
                            LogBaseEpcInfo epcInfo = MainActivity.tagEpcQueue.take();

                            DataOutputStream out = new DataOutputStream(client.getOutputStream());
                            out.write(epcInfo.getbEpc());
                            out.flush();
                        }
                    } catch (InterruptedException | IOException e) {
                        e.printStackTrace();
                    }
                }
            });

            try {
                // 创建客户端Socket，指定服务器地址和端口
                client = new Socket("192.168.3.2", 8200);
//                socket.shutdownOutput();//关闭输出流
//                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
    }
}
