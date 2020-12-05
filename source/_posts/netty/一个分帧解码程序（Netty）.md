---
title: 一个分帧解码程序（Netty）
date: 2020-09-03 21:24:17
comments: true
categories:
 - Netty
 - 解码
tags: 
 - netty
 - 网络
---

```java
package sdk.protocol.m200x.frame.coder;

import sdk.protocol.m200x.frame.FrameConstant;
import sdk.util.HexadecimalUtil;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufUtil;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToMessageDecoder;
import lombok.extern.slf4j.Slf4j;

import java.util.Arrays;
import java.util.List;

/**
 * @author Rubin
 * @version v1 2020/7/16 15:42
 */
@Slf4j
public class FrameDecoder extends MessageToMessageDecoder<ByteBuf> {

    /**
     * 协议头标识
     */
    private int head = 0;

    /**
     * 帧buff
     */
    private byte[] frame = new byte[1024];

    /**
     * 当前帧buff下标
     */
    private int currentFrameIndex = 0;

    /**
     * 数据长度
     */
    private int dataLen = 0;

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext,
                          ByteBuf msg, List<Object> out) throws Exception {
                          
        while (msg.isReadable()) {
            byte readByte = msg.readByte();

            // 如果head==1，开始读数据
            if (head == 1) {
                frame[currentFrameIndex++] = readByte;

                // 如果frameIndex==4，获取dataLen
                if (currentFrameIndex == 5) {
                    dataLen = HexadecimalUtil.get10HexNum(HexadecimalUtil.bytes2HexString(new byte[]{frame[3], frame[4]}));
                    // todo 读取到数据长度，取消读数据长度超时
                    // todo 需要判断长度是否正确
                    // todo 开始读数据长度，设置取数据长度的超时
                }
                // 判断是否读完一帧数据
                else if (dataLen == (currentFrameIndex - FrameConstant.LENGTH)) {

                    // 分好帧后存入队列
                    // 去除多余的空间
                    byte[] frameBytes = Arrays.copyOfRange(frame, 0, dataLen + FrameConstant.LENGTH);// 截取索引2（包括）到索引6（不包括）的元素
                    //System.arraycopy(frame, 0, frameBytes, 0, frameBytes.length);

                    //入队
                    out.add(frameBytes);

                    // 重新读取，初始化下标
                    head = 0;
                    currentFrameIndex = 0;
                    dataLen = 0;
                    frame = new byte[1024];

                    // todo 读取完指定的数据长度，取消取数据长度的超时

                }
            }
            // 如果不是开头，则判断7F
            else {
                if (readByte == 127) {
                    // 是7F时
                    frame[currentFrameIndex++] = readByte;
                    // 标记头已经找到
                    head = 1;
                    // todo 设置读数据长度超时
                }
            }
        }

    }
}
```
