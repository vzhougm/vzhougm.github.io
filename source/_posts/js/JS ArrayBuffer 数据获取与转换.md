---
title: JS ArrayBuffer 数据获取与转换
date: 2020-09-11 12:03:06
comments: true
categories:
 - JS
 - 数据结构
tags: 
 - js
 - 数据结构
---
# JS ArrayBuffer 数据获取与转换

## 简介
> ArrayBuffer 对象用来表示通用的、固定长度的原始二进制数据缓冲区。
> 
> 它是一个字节数组，通常在其他语言中称为“byte array”。
> 
> 你不能直接操作 ArrayBuffer 的内容，而是要通过类型数组对象或 DataView 对象来操作，它们会将缓冲区中的数据表示为特定的格式，并通过这些格式来读写缓冲区的内容。
> 
> [ArrayBuffer - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer "ArrayBuffer - JavaScript | MDN")

## 查看ArrayBuffer内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911114942396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

内部有4种存储方式，一般 Uint8Array 使用的多
```javascript
[Int8Array]]: Int8Array(20) [104, 116, 116, 112, 58, 47, 47, 119, 119, 119, 46, 99, 109, 115, 111, 102, 116, 46, 99, 110]
[[Int16Array]]: Int16Array(10) [29800, 28788, 12090, 30511, 30583, 25390, 29549, 26223, 11892, 28259]
[[Int32Array]]: Int32Array(5) [1886680168, 1999580986, 1663989623, 1718580077, 1851993716]
[[Uint8Array]]: Uint8Array(20) [104, 116, 116, 112, 58, 47, 47, 119, 119, 119, 46, 99, 109, 115, 111, 102, 116, 46, 99, 110]
byteLength: 20
```

## 转16进制字符
```javascript
function buf2hex(buffer) { // buffer is an ArrayBuffer
      return Array.prototype.map.call(new Uint8Array(buffer), x => ('00' + x.toString(16)).slice(-2)).join('');
  }
  
// 也可这样定义 
const buf2hex = _ => Array.prototype.map.call(new Uint8Array(_), _ => ('00' + _.toString(16)).slice(-2)).join('')
```

![效果图](https://img-blog.csdnimg.cn/20200911122109869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)


👌 介绍完毕！请开始你的表演~
