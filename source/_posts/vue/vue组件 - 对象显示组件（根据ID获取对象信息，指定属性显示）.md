---
title: vue组件 - 对象显示组件（根据ID获取对象信息，指定属性显示）
date: 2020-11-03 14:43:41
comments: true
categories:
 - Vue
 - 组件
tags: 
 - vue
 - vue组件
---
## 需求
很多时候因对象关联，前端只收到一个关联对象的ID，或者枚举值。这个如果后台不处理的话前端往往需要根据ID再去后台查询获得对象数据显示对象信息（一般显示name）。因此这是一个很常见的操作，所以我们可以封装一个组件，让组件自动帮我们去查询。

## 开始做
新建 GetModelInfo.vue 文件

### 定义参数
包含 fdId（要查询的ID）、url（后台接口路径）、show（显示的字段）、attributes（要查询的属性）
```js
props: {
    fdId: {
      type: String,
      required: true
    },
    url: {
      type: String,
      required: true
    },
    show: {
      type: String,
      required: true
    },
    attributes: {
      type: Array,
      required: false
    }
  },
```
### 显示

```html
<template>
  <span>{{ modelInfo }}</span>
</template>
```
### 查询

```javascript
activated() {
   // 进入页面的时候查询数据
  show(this.url, this.fdId, this.attributes).then(_ => {
    const { data } = _.data
    this.modelInfo = data[this.show] // 显示字段
    this.$emit('model-info', data) // 返回调用者完整的数据
  })
}
```

### 完整代码

```html
<template>
  <span>{{ modelInfo }}</span>
</template>

<script>
import { show } from '@/services/baseService'

export default {
  name: 'GetModelInfo',
  props: {
    fdId: {
      type: String,
      required: true
    },
    url: {
      type: String,
      required: true
    },
    show: {
      type: String,
      required: true
    },
    attributes: {
      type: Array,
      required: false
    }
  },
  data() {
    return {
      modelInfo: ''
    }
  },
  activated() {
    show(this.url, this.fdId, this.attributes).then(_ => {
      const { data } = _.data
      this.modelInfo = data[this.show]
      this.$emit('model-info', data)
    })
  }
}
</script>

<style scoped>

</style>

```

这里的 show  是个通用查询，根据ID和要查询的字段动态返回。

## 使用
在需要调用的地方引入即可

```html
...
<!-- test -->
<get-model-info fd-id="rrr" :url="url0" show="fdType" :attributes="attributes" @model-info="modelInfo" />
...

<script>
import GetModelInfo from '@/components/GetModelInfo'
...

export default {
  components: { GetModelInfo },
  data() {
    return {
      url0: 'eam_device_main',
      attributes: ['fdId', 'fdType'],
      ...
    }
  },
  methods: {
  	modelInfo(data) {
      console.log(data)
    },
  }
  ...
</script>
```

## 效果
请求
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103144024552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

结果显示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103143731773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

