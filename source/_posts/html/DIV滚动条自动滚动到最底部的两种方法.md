---
title: DIV滚动条自动滚动到最底部的两种方法
date: 2020-12-04 12:49:56
comments: true
categories:
 - HTML
tags: 
 - html
---
# 方法1
```javascript
function updateScroll(){
    var element = document.getElementById("divId");
    element.scrollTop = element.scrollHeight;
}
```

# [方法2](https://codepen.io/jimbol/pen/YVJzBg)
```html
<div class="container">
  <div class="inner">Bottom</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Hi</div>
  <div class="inner">Top</div>
</div>

<style>
.container {
  height: 100px;
  overflow: auto;
  display: flex;
  flex-direction: column-reverse;
}
</style>
```
