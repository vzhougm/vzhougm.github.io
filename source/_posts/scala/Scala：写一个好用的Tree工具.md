---
title: Scala：写一个好用的Tree工具
date: 2020-11-07 14:55:26
comments: true
categories:
 - 工具
 - Scala
tags: 
 - scala
 - tool
---

## 一、定义IdName特质
包含Id和Name

```scala
trait Id {
  val fdId: String
}

trait Name {
  val name: String
}

trait IdName extends Id with Name
```

## 二、定义引用类（父类）
```scala
/**
 * 关联对象
 *
 * @param fdId ID
 * @param name 名字
 */
case class Relevance(fdId: String = "-", name: String = "-") extends IdName
```

## 三、定义Tree特质
包含parent和children，children使用的java版本的，你懂的scala的集合类转Json是比较痛苦的。

```scala
trait Tree extends IdName {
  val parent: Relevance
  val children: util.List[AnyRef]
}
```

## 四、创建Tree工具类
这里用伴生对象

```scala
object Tree {

  /**
   * 创建树
   *
   * @param data 数据
   * @param root 根 (fdId, parent)
   * @tparam T 类型
   * @return
   */
  def create[T <: Tree](data: Seq[T], root: (String, String) = ("root", "-1")): Seq[T] = {
    for (d <- data; if d.parent.fdId == root._2 || d.fdId == root._1) yield {
      recursionFn(data, d); d
    }
  }

  /**
   * 递归列表
   *
   * @param data   所有数据
   * @param parent 父节点
   */
  private def recursionFn[T <: Tree2](data: Seq[T], parent: T): Unit = {
    val children = data.filter(_.parent.fdId == parent.fdId)
    parent.children.addAll(children.asJava)
    children.foreach(nextChild => recursionFn(data, nextChild))
  }

}
```
## 五、使用

以组织架构树为例子

### 1. 定义一个 OrgTree case calss 集成 Tree特质

```scala
case class OrgTree(fdId: String, name: String, parent: Relevance, children: util.List[AnyRef]) extends Tree
```

### 2. 从数据库查询数据并转换为Tree

```scala
/**
   * 组织架构树
   *
   * @return
   */
  def tree: Future[Seq[OrgTree]] = {
    val p = Promise[Seq[OrgTree]]()
    val all = db.run(table.sortBy(_.sequence).map(d => (d.fdId, d.name, d.parent)).result)
    all onComplete {
      case Success(data) =>
        p.success(Tree.create(data map t2TreeDTO))
      case Failure(exception) =>
        p.failure(exception)
    }
    p.future
  }

  /**
   * 特质转TreeDTO
   *
   * @param t3 (fdId, name, parent)
   * @return
   */
  private def t2TreeDTO(t3: (String, String, String)): OrgTree = {
    implicit val dbRun = (id: String) => db.run(findParentCompiled(id).result.head)
    OrgTree(t3._1, t3._2, findParentByCache(t3._3), new util.ArrayList[AnyRef]())
  }
```
findParentByCache 是根据父类ID查询父类信息的函数

## 六、效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201107144514452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

## 七、Tree 完整代码

```scala

import java.util

import scala.jdk.CollectionConverters.SeqHasAsJava

/**
 * @author Rubin
 * @version v1 2020/11/7 11:28
 */
trait Tree extends IdName {
  val parent: Relevance
  val children: util.List[AnyRef]
}

object Tree {

  /**
   * 创建树
   *
   * @param data 数据
   * @param root 根
   * @tparam T 类型
   * @return
   */
  def create[T <: Tree](data: Seq[T], root: (String, String) = ("root", "-1")): Seq[T] = {
    for (d <- data; if d.parent.fdId == root._2 || d.fdId == root._1) yield {
      recursionFn(data, d); d
    }
  }

  /**
   * 递归列表
   *
   * @param data   所有数据
   * @param parent 父节点
   */
  private def recursionFn[T <: Tree](data: Seq[T], parent: T): Unit = {
    val children = data.filter(_.parent.fdId == parent.fdId)
    parent.children.addAll(children.asJava)
    children.foreach(nextChild => recursionFn(data, nextChild))
  }

}
```
