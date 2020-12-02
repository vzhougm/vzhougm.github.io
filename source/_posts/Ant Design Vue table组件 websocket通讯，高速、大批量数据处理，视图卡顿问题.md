---
title: Ant Design Vue table组件 websocket通讯，高速、大批量数据处理，视图卡顿问题
date: 2020-12-02 19:51:46
comments: true
categories:
 - UI框架
 - ant-design-vue
tags: 
 - vue
 - ant-design-vue
 - websocket
---
# 问题
Vue 常用的UI框架，table组件在 websocket通讯时，大量数据过来时页面数据加载不过来的情况，三个UI框架对比结果 ant-design-vue 和 iview 比较差 element-ui 相对好点，但也卡顿。

下图是博主开发的读卡器SDK测试工具，在做测速时就遇到的这问题，找来找去，最终发现**是UI框架的问题**！

# 效果对比
**测试机器：[NRP-D915I4 超高频分体式读写器](http://www.enruipu.com/?c=msg&id=2130)**
单天线模式，2500~3600ms读500张卡，测试环境非严苛，严苛环境下读卡性能更强悍。
## 卡顿
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201202193659585.gif)

需要鼠标移动视图才会刷新！体验感极差

## 流畅
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201202193718628.gif)
效果很明显

页面实际过来的数据远远大于500，这里对数据进行了去重。

# 解决问题
之前也找了各种方案，如`this.$forceUpdate()`、`this.$nextTick()`、 `this.$set()`等，均与失败告终。后面发现只要数据不加载到table里，它就不会卡。于是就**自己写了个table组件！** 🤣

效果还不错，其实也简单样式就拷贝UI框架本身的，把数据部分替换掉就行了。

# 代码
Table.vue
```html
<template>
  <div class="ant-table-wrapper">
    <div class="ant-spin-nested-loading">
      <div class="ant-spin-container">
        <div
            class="ant-table ant-table-fixed-header ant-table-scroll-position-left ant-table-layout-fixed ant-table-small ant-table-bordered">
          <div class="ant-table-content">
            <div class="ant-table-scroll">
              <div class="ant-table-header ant-table-hide-scrollbar"
                   style="margin-bottom: -17px; padding-bottom: 0px; min-width: 17px; overflow: scroll;">
                <table class="">
                  <colgroup>
                    <col style="width: 70px; min-width: 70px;">
                    <col>
                    <col style="width: 40%; min-width: 40%;">
                    <col style="width: 85px; min-width: 85px;">
                    <col>
                    <col>
                  </colgroup>
                  <thead class="ant-table-thead">
                  <tr>
                    <th key="key" class="ant-table-row-cell-break-word"><span
                        class="ant-table-header-column"><div><span
                        class="ant-table-column-title">#</span><span
                        class="ant-table-column-sorter"></span></div></span></th>
                    <th key="pc" class=""><span class="ant-table-header-column"><div><span
                        class="ant-table-column-title">PC</span><span
                        class="ant-table-column-sorter"></span></div></span></th>
                    <th key="epc" class="ant-table-row-cell-break-word"><span
                        class="ant-table-header-column"><div><span class="ant-table-column-title">EPC</span><span
                        class="ant-table-column-sorter"></span></div></span></th>
                    <th key="rssi" class="ant-table-row-cell-break-word"><span
                        class="ant-table-header-column"><div><span class="ant-table-column-title">RSSI(dBm)</span><span
                        class="ant-table-column-sorter"></span></div></span></th>
                    <th key="ant" class=""><span class="ant-table-header-column"><div><span
                        class="ant-table-column-title">ANT</span><span
                        class="ant-table-column-sorter"></span></div></span></th>
                    <th key="times" class="ant-table-row-cell-last"><span
                        class="ant-table-header-column"><div><span class="ant-table-column-title">Times</span><span
                        class="ant-table-column-sorter"></span></div></span></th>
                  </tr>
                  </thead>
                </table>
              </div>
              <div tabindex="-1" class="ant-table-body" v-bind:style="{ height: height + 'px' }"
                   style="overflow-y: scroll;">
                <table class="">
                  <colgroup>
                    <col style="width: 70px; min-width: 70px;">
                    <col>
                    <col style="width: 40%; min-width: 40%;">
                    <col style="width: 85px; min-width: 85px;">
                    <col>
                    <col>
                  </colgroup>
                  <tbody class="ant-table-tbody">

                  <tr v-for="(data, index) in dataSource"
                      v-bind:class="{ even: index % 2 === 0}">
                    <td class="ant-table-row-cell-break-word">{{ index + 1 }}</td>
                    <td class="">{{ data.pc }}</td>
                    <td class="ant-table-row-cell-break-word">{{ data.epc }}</td>
                    <td class="ant-table-row-cell-break-word">{{ data.rssi }}</td>
                    <td class="">{{ data.ant + 1 }}</td>
                    <td class="">{{ data.times }}</td>
                  </tr>

                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Table2',
  props: {
    dataSource: {
      type: Array
    },
    height: {
      type: String
    }
  }
}
</script>

<style scoped>
.even {
  background-color: #F5F7FA;
}
</style>

```

使用
```html
<!--        <a-table :columns="columns"-->
<!--                 size="small"-->
<!--                 :data-source="tags"-->
<!--                 :pagination="false"-->
<!--                 :scroll="{ y: screenHeight - 200 }"-->
<!--                 bordered>-->
<!--        </a-table>-->


                <table2 :data-source="tags" :height="screenHeight - 200"/>

```
OK！搞定~ 👌
