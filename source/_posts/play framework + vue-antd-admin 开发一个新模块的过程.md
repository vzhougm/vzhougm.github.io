---
title: play framework + vue-antd-admin 开发一个新模块的过程
date: 2020-09-18 19:49:35
comments: true
categories:
 - Scala
 - Play framework
tags: 
 - play framework
 - ant-design-vue
 - scala
---

## 需求
做一个简单的增删改查功能
## 添加页面
简单的话一个list就搞定了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918142731653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
### 使用的组件

[Table 表格 - 可编辑行](https://www.antdv.com/components/table-cn/#%E5%8F%AF%E7%BC%96%E8%BE%91%E8%A1%8C)
[Table 表格 - 可编辑单元格](https://www.antdv.com/components/table-cn/#%E5%8F%AF%E7%BC%96%E8%BE%91%E5%8D%95%E5%85%83%E6%A0%BC)

### 代码
```javascript
<template>
  <a-card :bordered="false">
    <a-button type="primary" style="margin-bottom: 10px" @click="handleAdd">Add</a-button>
    <a-table :columns="columns" :data-source="protocols" bordered>
      <template
          v-for="col in ['name', 'code']"
          :slot="col"
          slot-scope="text, record"
      >
        <div :key="col">
          <a-input
              v-if="record.editable"
              style="margin: -5px 0"
              :value="text"
              @change="e => handleChange(e.target.value, record.key, col)"
          />
          <template v-else>
            {{ text }}
          </template>
        </div>
      </template>
      <template slot="operation" slot-scope="text, record">
        <div class="editable-row-operations">
              <span v-if="record.editable">
                <a @click="() => save(record.key)">保存</a>
                <a-popconfirm title="确定取消修改？" @confirm="() => cancel(record.key)" style="margin-left: 5px">
                  <a>取消</a>
                </a-popconfirm>
              </span>
          <span v-else><a :disabled="editingKey !== ''" @click="() => edit(record.key)">编辑</a></span>

          <a-popconfirm style="margin-left: 10px"
                        v-if="protocols.length && editingKey === ''"
                        title="确定要删除？"
                        @confirm="() => onDelete(record.key)">
            <a href="javascript:">删除</a>
          </a-popconfirm>
        </div>
      </template>
    </a-table>
  </a-card>
</template>

<script>

export default {
  data() {
    return {
      count: 1,
      protocols: [],
      editingKey: '',
      columns: [
        {
          title: '#',
          dataIndex: 'key',
          fixed: 'left',
          width: 70,
          scopedSlots: {customRender: 'key'}
        },
        {
          title: 'name',
          dataIndex: 'name',
          scopedSlots: {customRender: 'name'},
        },
        {
          title: 'code',
          dataIndex: 'code',
          scopedSlots: {customRender: 'code'},
        },
        {
          title: 'operation',
          dataIndex: 'operation',
          scopedSlots: {customRender: 'operation'},
        },
      ],
    };
  },
  methods: {
    // ###################### Table start ######################
    handleAdd() {
      const {count, protocols} = this;
      const newData = {
        key: count,
        name: '协议名',
        code: 'Code'
      };
      this.protocols = [...protocols, newData];
      this.cacheData = this.protocols.map(item => ({...item}))
      this.count = count + 1;
    },
    handleChange(value, key, column) {
      const newData = [...this.protocols];
      const target = newData.filter(item => key === item.key)[0];
      if (target) {
        target[column] = value;
        this.protocols = newData;
      }
    },
    edit(key) {
      const newData = [...this.protocols];
      const target = newData.filter(item => key === item.key)[0];
      this.editingKey = key;
      if (target) {
        target.editable = true;
        this.protocols = newData;
      }
    },
    save(key) {
      const newData = [...this.protocols];
      const newCacheData = [...this.cacheData];
      const target = newData.filter(item => key === item.key)[0];
      const targetCache = newCacheData.filter(item => key === item.key)[0];
      if (target && targetCache) {
        delete target.editable;
        this.protocols = newData;
        Object.assign(targetCache, target);
        this.cacheData = newCacheData;
      }
      this.editingKey = '';
    },
    cancel(key) {
      const newData = [...this.protocols];
      const target = newData.filter(item => key === item.key)[0];
      this.editingKey = '';
      if (target) {
        Object.assign(target, this.cacheData.filter(item => key === item.key)[0]);
        delete target.editable;
        this.protocols = newData;
      }
    },
    onDelete(key) {
      const dataSource = [...this.protocols];
      this.protocols = dataSource.filter(item => item.key !== key);
    },
    // ###################### Table end ######################
  }
};
</script>
<style>
.editable-cell {
  position: relative;
}

.editable-cell-input-wrapper,
.editable-cell-text-wrapper {
  padding-right: 24px;
}

.editable-cell-text-wrapper {
  padding: 5px 24px 5px 5px;
}

.editable-cell-icon,
.editable-cell-icon-check {
  position: absolute;
  right: 0;
  width: 20px;
  cursor: pointer;
}

.editable-cell-icon {
  line-height: 18px;
  display: none;
}

.editable-cell-icon-check {
  line-height: 28px;
}

.editable-cell:hover .editable-cell-icon {
  display: inline-block;
}

.editable-cell-icon:hover,
.editable-cell-icon-check:hover {
  color: #108ee9;
}

.editable-add-btn {
  margin-bottom: 8px;
}
</style>

```

## 添加路由
用的是异步路由

在`src/router/async/router.map.js`中添加新页面的路由 `protocol`

```javascript
device: {
        name: '设备管理',
        path: 'device',
        icon: 'api',
        component: view.blank,
    },
    protocol: {
        name: '协议管理',
        path: 'protocol',
        component: () => import('@/pages/protocol/list'),
    },
    devices: {
        name: '所有设备',
        path: 'devices',
        component: () => import('@/pages/devices/list'),
    }
```

添加新路由到数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918143850341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

### 效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918143216765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

## 添加请求
在 `services`目录下添加新模块的请求接口文件`protocol.js`

在`api.js`文件中追加新模块的请求路径

```javascript
PROTOCOL_ADD: `${BASE_URL}/protocol/add`,
PROTOCOL_DEL: `${BASE_URL}/protocol/del`,
PROTOCOL_UPDATE: `${BASE_URL}/protocol/update`,
PROTOCOL_LIST: `${BASE_URL}/protocol/list`,
```
给`protocol.js`添加内容

```javascript
import {METHOD, request} from "@/utils/request";
import {PROTOCOL_ADD, PROTOCOL_DEL, PROTOCOL_UPDATE, PROTOCOL_LIST} from "@/services/api";

export async function add(data) {
    return request(PROTOCOL_ADD, METHOD.POST, data)
}

export async function update(data) {
    return request(PROTOCOL_UPDATE, METHOD.PUT, data)
}

export async function del(fdId) {
    return request(PROTOCOL_DEL, METHOD.POST, {
        fdId: fdId
    })
}

export async function list(page, limit) {
    return request(PROTOCOL_LIST, METHOD.GET, {
        page: page,
        limit: limit
    })
}
```
### 页面中使用
#### list

```javascript
created() {
    list().then(response => {
      const {data} = response.data
      for (let i = 0; i < data.length; i++) {
        this.protocols.push({key: this.count++, fdId: data[i].fdId, fdName: data[i].fdName, fdCode: data[i].fdCode})
      }
    })
  },
```
#### add

```javascript
 save(key) {
      ...
      // 后台更新
        add({fdId: target.fdId, fdName: target.fdName, fdCode: target.fdCode}).then(response => {
              const {meta} = response.data
              if (meta.code === 50000) {
                // todo 失败！
              }
            })
	  ...
    },
```

#### del

```javascript
 onDelete(key, type) {
      if (type) {
        const target = this.protocols.filter(item => key === item.key)[0];
        del(target.fdId).then(response => {
          const {meta} = response.data
          if (meta.code === 50000) {
            // todo 失败！
          } else {
            this.count--
            const dataSource = [...this.protocols];
            this.protocols = dataSource.filter(item => item.key !== key);
          }
        })
      }
    },
```
---

## 后台服务（Play framework）
前面的表设计和 slick 代码生成就略过了，这里使用了通用的CRUD工具，感兴趣的同学可以看看我之前的博客 [slick 通用查询工具（带分页）](https://blog.oweight.com/2020/08/30/slick%20%E9%80%9A%E7%94%A8%E6%9F%A5%E8%AF%A2%E5%B7%A5%E5%85%B7%EF%BC%88%E5%B8%A6%E5%88%86%E9%A1%B5%EF%BC%89/)

### 创建一个Controller

```scala
package controllers

import dao.ProtocolDAO
import javax.inject.Inject
import play.api.mvc.{AbstractController, Action, AnyContent, ControllerComponents}

import scala.concurrent.{ExecutionContext, Future}

/**
 * 支持的协议
 *
 * @author Rubin
 * @version v1 2020/9/17 15:35
 */
class ProtocolController @Inject()(components: ControllerComponents,
                                   protocolDAO: ProtocolDAO)
                                  (implicit executionContext: ExecutionContext) extends AbstractController(components) {


  def list: Action[AnyContent] = Action async { implicit request =>

    Future {
      Ok("")
    }

  }

  def update: Action[AnyContent] = Action async { implicit request =>

    Future {
      Ok("")
    }
  }

  def del: Action[AnyContent] = Action async { implicit request =>

    Future {
      Ok("")
    }
  }
}

```

### 创建一个DAO

```scala
package dao

import javax.inject.Inject
import play.api.db.slick.{DatabaseConfigProvider, HasDatabaseConfigProvider}
import repos.CRUD
import repos.table.Tables._
import slick.jdbc.JdbcProfile

import scala.concurrent.ExecutionContext

/**
 * @author Rubin
 * @version v1 2020/9/18 15:13
 */
class ProtocolDAO @Inject()(protected val dbConfigProvider: DatabaseConfigProvider)
                           (implicit executionContext: ExecutionContext)
  extends HasDatabaseConfigProvider[JdbcProfile] with CRUD[DeviceProtocol, DeviceProtocolRow] {

  override val database = db
  override val table: AnyRef = DeviceProtocol

}

```

### 创建一个Service
e，简单CRUD不想写这一层了，这里处理除了CRUD外的业务逻辑。没有就不写咯~ 摊手🤪

### 添加路由

```scala
# =========== 设备模块 ===========
#获取协议列表
GET         /protocol/list          controllers.ProtocolController.list
#添加或修改协议
POST        /protocol/add           controllers.ProtocolController.add
#移除协议
POST        /protocol/del           controllers.ProtocolController.del
```
### 完善Controller

```scala
package controllers

import dao.ProtocolDAO
import dao.ProtocolDAO.DeviceProtocol
import javax.inject.Inject
import play.api.libs.json._
import play.api.mvc._
import repos.table.Tables.DeviceProtocolRow
import utils.Message

import scala.concurrent.{ExecutionContext, Promise}
import scala.util.Success

/**
 * 支持的协议
 *
 * @author Rubin
 * @version v1 2020/9/17 15:35
 */
class ProtocolController @Inject()(components: ControllerComponents,
                                   protocolDAO: ProtocolDAO)
                                  (implicit executionContext: ExecutionContext) extends AbstractController(components) {


  def list: Action[AnyContent] = Action async { implicit request =>

    val p = Promise[Result]
    val message = Message.build
    protocolDAO.list onComplete {
      case Success(value) =>
        val list1 = message.getJavaList[DeviceProtocolRow]

        value.foreach(list1.add)
        message.data = list1
        p.success(Ok(message.toJson))
    }

    p.future

  }

  def add: Action[JsValue] = Action(parse.json) async { implicit request =>

    val p = Promise[Result]
    val message = Message.build
    val jsonValue = request.body

    implicit val protocolRowFormat: OFormat[DeviceProtocol] = Json.format[DeviceProtocol]
    Json.fromJson[DeviceProtocol](jsonValue) match {
      case protocol: JsSuccess[DeviceProtocol] =>
        protocolDAO.update(protocol.value.toTableRow)
        p.success(Ok(message.toJson))
      case e: JsError => println(e.errors)
    }
    p.future

  }

  def del: Action[AnyContent] = Action async { implicit request =>
    val json = request.body.asJson.get
    val fdId = json("fdId").as[String]
    val p = Promise[Result]
    protocolDAO.delete(fdId) onComplete {
      case Success(value) => p.success(Ok(Message.build.ok(value.toString).toJson))
    }
    p.future
  }
}
```
add方法中，Scala JSON操作还不是很熟悉，目前觉得有点麻烦，也是需要注意的地方。

我在DAO里定义了个伴生类

```scala
object ProtocolDAO {

  case class DeviceProtocol(fdId: String, fdName: String, fdCode: String) {
    def toTableRow: DeviceProtocolRow = fdId match {
      case "" => DeviceProtocolRow(fdName = fdName, fdCode = fdCode)
      case _ => DeviceProtocolRow(fdId = fdId, fdName = fdName, fdCode = fdCode)
    }
  }

}
```
以上代码后期都可以用代码工具生成，有空了写个代码工具。

## 最终效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918174847462.gif#pic_center)
额... 新增 -> 取消 有个🐞
