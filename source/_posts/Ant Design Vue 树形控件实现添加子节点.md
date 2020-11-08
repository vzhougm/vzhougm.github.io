---
title: Ant Design Vue 树形控件实现添加子节点
date: 2020-08-30 11:57:00
comments: true
categories:
 - UI框架
 - ant-design-vue
tags: 
 - vue
 - ant-design-vue
---


# 先上个效果图😎
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828131635956.gif#pic_center)
# 前奏
上面的树是基于 [Ant Design Vue 可搜索树](https://1x.antdv.com/components/tree-cn/) 修改而来的。
# #要点# JS对象赋值是基于地址引用的
所以找到了要添加子节点的对象把它赋值到自己定义的对象中，就可操作此对象了，如添加子节点。

# 添加到根目录下
这个比较简单，因为树就是从root开始的，所以直接拿到root节点的子节点数组，添加新节点对象即可。

**定义的字段**
`currentSelectTreeData` Tree 当前选择的节点Key
`findMenuObj` 用于存数组中到到 currentSelectTreeData对应对象的地址

```javascript
addMenu() {
  const newChild = {
    title: '编辑菜单',
    key: (new Date()).getTime(),
    scopedSlots: {title: 'title'},
    children: []
  }
  const root = this.treeData[0]
  // 添加到根目录
  if (this.currentSelectTreeData === 'root') {
    root.children.push(newChild)
  } else {
    // 添加到子目录
  }
  generateList(this.treeData)
}
```

# 添加到其他子节点下
非 root 下节点需要遍历 Tree 匹配 key，这个 key 就是当前选中的key。

**查找选择的子节点，并将其绑定到 findMenuObj 对象**
```javascript
// 查找菜单在树的节点位置
findMenu(data, key) {
  data.children.filter(item => this.searchMenu(item, key))
},
searchMenu(item, key) {
  if (item.key === key) {
    this.findMenuObj = item
    return true
  } else {
    if (item.children.length > 0) {
      this.findMenu(item, key)
    } else {
      return false
    }
  }
}
```
**添加到子目录**
```javascript
addMenu() {
  const newChild = {
    title: '编辑菜单',
    key: (new Date()).getTime(),
    scopedSlots: {title: 'title'},
    children: []
  }
  const root = this.treeData[0]
  // 添加到根目录
  if (this.currentSelectTreeData === 'root') {
    root.children.push(newChild)
  } else {
    // 添加到子目录
    this.findMenu(root, this.currentSelectTreeData)
    this.findMenuObj.children.push(newChild)
  }
  generateList(this.treeData)
}
```

# 完整代码

```javascript
// 添加菜单
addMenu() {
  const newChild = {
    title: '编辑菜单',
    key: (new Date()).getTime(),
    scopedSlots: {title: 'title'},
    children: []
  }
  const root = this.treeData[0]
  // 添加到根目录
  if (this.currentSelectTreeData === 'root') {
    root.children.push(newChild)
  } else {
    // 添加到子目录
    this.findMenu(root, this.currentSelectTreeData)
    this.findMenuObj.children.push(newChild)
  }
  generateList(this.treeData)
},
// 查找菜单在树的节点位置
findMenu(data, key) {
  data.children.filter(item => this.searchMenu(item, key))
},
searchMenu(item, key) {
  if (item.key === key) {
    this.findMenuObj = item
    return true
  } else {
    if (item.children.length > 0) {
      this.findMenu(item, key)
    } else {
      return false
    }
  }
}
```
👌 增搞定了，删和改就小k斯了😎

至此大功告成~ ✌ 
