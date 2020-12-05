---
title: vue组件 - select组件（动态数据，显示默认对象信息，返回选中对象ID）
date: 2020-11-03 21:06:40
comments: true
categories:
 - Vue
 - 组件
tags: 
 - vue
 - vue组件
---
## 需求
还是对象关联，如父类别。对象关联对象的更新时用到本组件。

## 开始做
新建 KaiSelect.vue 文件（项目名叫KaiAdmin，自己琢磨的通用后台，在为此项目写组件。🤭）

本组件是基于 [Ant Design Vue Select 选择器](https://antdv.com/components/select-cn/) 修改而来的。


### 定义参数
包含 open（当前组件的状态，是否打开）、model（当前值）、url（后台接口路径）、placeholder（默认提示文本）
```js
props: {
    open: {
      type: Object,
      required: true
    },
    model: {
      type: Object,
      required: true
    },
    url: {
      type: String,
      required: true
    },
    placeholder: {
      type: String,
      required: false
    }
  },
```
### 显示

```html
<template>
  <a-select
    v-model="defaultValue"
    show-search
    :placeholder="placeholder"
    :filter-option="filterOption"
    @change="handleChange"
  >
    <a-select-option v-for="(d, index) in data" :key="index" :value="d.fdId">
      {{ d.name }}
    </a-select-option>

  </a-select>
</template>
```
### 查询
查询主要是查询select列表数据，这里设置在 created 和 watch:open 两处。
```javascript
created() {
    // 初次加载、刷新时查询
    this.defaultValue = this.model.fdId // 默认选中的值
    get(this.url).then(_ => {
      const { data } = _.data
      this.data = data
    })
  },


watch: {
    open(data) {
      // 当列表数据为空时查询
      if (!this.data.length) {
        get(this.url).then(_ => {
          const { data } = _.data
          this.data = data
        })
      }
      this.defaultValue = this.model.fdId  // 默认选中的值
    }
  },
```
显示默认值是在`a-select` 中设置 `v-model="defaultValue"`，defaultValue的值就是id，也就是 `<a-select-option v-for="(d, index) in data" :key="index" :value="d.fdId">` 中的 `:value="d.fdId"`

设置 `@change="handleChange"` 事件，选中值时及时返回。

```javascript
handleChange(data) {
 this.$emit('chosen', data) // 在调用模块被监听
},
```

### 完整代码

```html
<template>
  <a-select
    v-model="defaultValue"
    show-search
    :placeholder="placeholder"
    :filter-option="filterOption"
    @change="handleChange"
  >
    <a-select-option v-for="(d, index) in data" :key="index" :value="d.fdId">
      {{ d.name }}
    </a-select-option>

  </a-select>
</template>
<script>
import { get } from '@/services/baseService'

export default {
  name: 'KaiSelect',
  props: {
    open: {
      type: Object,
      required: true
    },
    model: {
      type: Object,
      required: true
    },
    url: {
      type: String,
      required: true
    },
    // eslint-disable-next-line vue/require-default-prop
    placeholder: {
      type: String,
      required: false
    }
  },
  data() {
    return {
      data: [],
      defaultValue: ''
    }
  },

  watch: {
    open(data) {
      if (!this.data.length) {
        get(this.url).then(_ => {
          const { data } = _.data
          this.data = data
        })
      }
      this.defaultValue = this.model.fdId
    }
  },
  created() {
    // 初次加载、刷新时查询
    this.defaultValue = this.model.fdId
    get(this.url).then(_ => {
      const { data } = _.data
      this.data = data
    })
  },
  methods: {
    handleChange(data) {
      this.$emit('chosen', data)
    },
    filterOption(input, option) {
      return (
        option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
      )
    }
  }
}
</script>

```

## 使用

```html
<a-form-model-item label="父级" prop="parent">
  <kai-select
    v-if="pageType!=='show'"
    :model="form.parent"
    placeholder="父级"
    url="eam_device_category/selectItems"
    :open="selectOpen"
    @chosen="chosen"
  />


<script>
 methods: {
    chosen(id) {
      // 选中的父类ID
      this.form.parent = { fdId: id, name: '' }
    },
 }
</script>
```

## 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103212808946.gif#pic_center)
