---
title: slick 通用查询工具（带分页）
date: 2020-08-30 11:15:42
comments: true
categories:
 - Scala
 - Slick
tags: 
 - scala
 - slick
 - tool
---
# 1. 首先定义一个Model特质
在指定ID查询和删除时用，根据实际情况定义字段。

```scala
package slick.crud

import slick.lifted.Rep

/**
 * @author Rubin
 * @version v1 2020/8/26 12:50
 */
trait Model {
  val fdId: Rep[String]
}
```

# 2. 再建一个CRUD特质
```scala
package repos

import repos.table.Tables.profile.api._
import slick.jdbc.JdbcProfile
import slick.lifted.TableQuery

import scala.concurrent.duration.Duration
import scala.concurrent.{Await, Future, Promise}

/**
 * Slick CRUD util
 *
 * @author Rubin
 * @version v1 2020/8/26 11:52
 */
trait CRUD[M <: Model, R] {

  val database: JdbcProfile#Backend#Database
  val table: AnyRef

  private def getTable: TableQuery[Table[R]] = table.asInstanceOf[TableQuery[Table[R]]]

  def insert(entity: R): Future[Unit] = database.run(DBIO.seq(getTable += entity))

  def delete(fdId: String): Future[Int] = database.run(getTable.filter(_.asInstanceOf[M].fdId === fdId).delete)

  def update(entity: R): Future[Int] = database.run(getTable.insertOrUpdate(entity))

  def findById(fdId: String): Future[R] = database.run(getTable.filter(_.asInstanceOf[M].fdId === fdId).result.head)
  
  def page(page: Page[R]): Future[Page[R]] = {
    val p = Promise[Page[R]]
    try {
      val data = database.run(getTable.drop(page.page).take(page.limit).result)
      val size = database.run(getTable.size.result)
      page.data = Await.result(data, Duration.Inf)
      page.totalCount = Await.result(size, Duration.Inf)
      page.setTotalPage()
    } catch {
      case _: Throwable => p.failure _
    }
    p.success(page)
    p.future
  }
  
}
```

# 3. 分页用的Page

setTotalPage 在查询到总数的时候需要显示的调用一下，因为查询是异步的，直接引用的话拿不到实际值。
```scala
package slick.crud

/**
 * @author Rubin
 * @version v1 2020/8/25 11:20
 */
class Page[T](page_ : Int = 1, limit_ : Int = 10) {
  val page: Int = page_
  val limit: Int = limit_
  var totalCount: Int = _
  var totalPage: Int = _
  var data: Seq[T] = _

  def setTotalPage(): Unit = {
    val count = totalCount / limit_
    if (totalCount % limit_ == 0) totalPage = count else totalPage = count + 1
  }

}
```
# 4. 使用
**1. 修改 slick-codegen 生成的代码，将表对应的对象添加 Model特质。**

``
class TestUser(_tableTag: Tag) extends profile.api.Table[TestUserRow](_tableTag, Some("test"), "test_user") with Model {
``

```scala
package slick.table

import slick.crud.Model
// AUTO-GENERATED Slick data model
/** Stand-alone Slick data model for immediate use */
object Tables extends {
  val profile = slick.jdbc.MySQLProfile
} with Tables

/** Slick data model trait for extension, choice of backend or usage in the cake pattern. (Make sure to initialize this late.) */
trait Tables {
  val profile: slick.jdbc.JdbcProfile
  import profile.api._
  import slick.model.ForeignKeyAction
  // NOTE: GetResult mappers for plain SQL are only generated for tables where Slick knows how to map the types of all columns.
  import slick.jdbc.{GetResult => GR}

  /** DDL for all tables. Call .create to execute. */
  lazy val schema: profile.SchemaDescription = TestUser.schema
  @deprecated("Use .schema instead of .ddl", "3.0")
  def ddl = schema

  /** Entity class storing rows of table TestUser
   *  @param fdid Database column fdId SqlType(VARCHAR), PrimaryKey, Length(64,true)
   *  @param fdname Database column fdName SqlType(VARCHAR), Length(255,true), Default(None) */
  case class TestUserRow(fdid: String, fdname: Option[String] = None)
  /** GetResult implicit for fetching TestUserRow objects using plain SQL queries */
  implicit def GetResultTestUserRow(implicit e0: GR[String], e1: GR[Option[String]]): GR[TestUserRow] = GR{
    prs => import prs._
    TestUserRow.tupled((<<[String], <<?[String]))
  }
  /** Table description of table test_user. Objects of this class serve as prototypes for rows in queries. */
    class TestUser(_tableTag: Tag) extends profile.api.Table[TestUserRow](_tableTag, Some("test"), "test_user") with Model {
    def * = (fdId, fdName) <> (TestUserRow.tupled, TestUserRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = ((Rep.Some(fdId), fdName)).shaped.<>({r=>import r._; _1.map(_=> TestUserRow.tupled((_1.get, _2)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))

    /** Database column fdId SqlType(VARCHAR), PrimaryKey, Length(64,true) */
    override val fdId: Rep[String] = column[String]("fdId", O.PrimaryKey, O.Length(64,varying=true))
    /** Database column fdName SqlType(VARCHAR), Length(255,true), Default(None) */
    val fdName: Rep[Option[String]] = column[Option[String]]("fdName", O.Length(255,varying=true), O.Default(None))
  }
  /** Collection-like TableQuery object for table TestUser */
  lazy val TestUser = new TableQuery(tag => new TestUser(tag))
}
```

**2. DAO with CRUD[M <: Model, R]**

```scala
package dao

import javax.inject.Inject
import play.api.db.slick.{DatabaseConfigProvider, HasDatabaseConfigProvider}
import repos.CRUD
import slick.jdbc.JdbcProfile

import repos.table.Tables._

import scala.concurrent.ExecutionContext

/**
 * @author Rubin
 * @version v1 2020/8/26 17:38
 */
class TestUserDAO  @Inject()(protected val dbConfigProvider: DatabaseConfigProvider)
                            (implicit executionContext: ExecutionContext) extends HasDatabaseConfigProvider[JdbcProfile] with CRUD[TestUser, TestUserRow] {

  override val database = db
  override val table = TestUser

  // 其他业务查询...
  
}
```
**3. 代码调用**
```scala

    for (i <- 1 to 500) {
      dao1.insert(TestUserRow("id_" + i))
    }
  
    dao1.delete("id_1").onComplete {
      case Success(value) => println(s"delete => $value")
    }

    dao1.findById("id_107").onComplete {
      case Success(value) => println(value.toArray.mkString("Array(", ", ", ")"))
    }

    dao1.update(TestUserRow("id_107", Option("修改了"))).onComplete {
      case Success(value) => println(s"update => $value")
    }
```

# 5. 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200826173253903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200826174447356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200826175209269.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200826175239762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)
👌 分页就不演示了，如果不觉得麻烦的话完全可以封装一个 springboot-jap 那种自动匹配的效果。

项目地址：https://gitee.com/vzhougm/slick_crud

最后附上官网给出的**第三方通用查询工具** [active-slick](https://github.com/strongtyped/active-slick)，玩的愉快~ 😘