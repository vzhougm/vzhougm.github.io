---
title: Scala API LazyList
date: 2020-09-15 19:00:00
comments: true
categories:
 - Scala
 - API
 - 集合框架
tags: 
 - scala
 - scala-api
 - collection
---

### Introduction
> final class LazyList[+A] extends AbstractSeq[A] with LinearSeq[A] with LinearSeqOps[A, LazyList, LazyList[A]] with IterableFactoryDefaults[A, LazyList] with Serializable

这个类实现了一个不可变链表。我们称它为“懒”，因为仅在需要时，它才会计算它的元素。

元素是记忆化的；也就是说，每个元素的值最多计算一次。元素是按顺序计算的，不会被跳过。换句话说，**访问尾部会先计算头部。**

一个`LazyList`有多懒？当你有一个类型为`LazyList`的值时，你还不知道这个列表是否为空。如果你知道它是非空的，那么你也知道头已经被计算过了。但如果最后一个元素是一个`LazyList`，它是否为空是未确定的。

一个`LazyList`是无限的。例如，`LazyList.from(0)`包含所有自然数0、1、2等等。对于无穷序列，某些方法(如'count'、'sum'、'max'或'min')将不会终止。

#### example

```scala
import scala.math.BigInt
object Main extends App {
  val fibs: LazyList[BigInt] =
    BigInt(0) #:: BigInt(1) #:: fibs.zip(fibs.tail).map{ n => n._1 + n._2 }
  fibs.take(5).foreach(println)
}

// prints
//
// 0
// 1
// 1
// 2
// 3
```

加点额外输出信息，让我们更直观的了解其工作过程。

```scala
import scala.math.BigInt
object Main extends App {
  val fibs: LazyList[BigInt] =
    BigInt(0) #:: BigInt(1) #::
      fibs.zip(fibs.tail).map{ n =>
        println(s"Adding ${n._1} and ${n._2}")
        n._1 + n._2
      }
  fibs.take(5).foreach(println)
  fibs.take(6).foreach(println)
}

// prints
//
// 0
// 1
// Adding 0 and 1
// 1
// Adding 1 and 1
// 2
// Adding 1 and 2
// 3

// And then prints
//
// 0
// 1
// 1
// 2
// 3 前面5个元素在第一次计算中使用并保存了下来
// Adding 2 and 3  元素6是新的所以才触发了 n._1 + n._2 操作
// 5
```

<span style=color:red>注意，fibs的定义使用的是`val`而不是`def`。`LazyList`的记忆需要我们有地方存储，而`val`可以。</span>

关于LazyList的语义进一步的说明：

- 虽然LazyList改变，因为它是访问，这并不违反其不变性。一旦值被记忆化它们就不会改变。这还没有值仍有待被记忆化的“存在”，它们根本就没有被计算呢。

- 我们必须谨慎值的记忆化；如果你不小心，它会消耗内存。这是因为LazyList的记忆化创建了一个类似[[scala.collection.immutable.List]]的结构。只要将其赋值给了对象，那它就从头部到尾部的数据一直保持。如果没有赋值到对象（例如，我们用def来定义LazyList），那么一旦它不再被使用，它就会消失。

- 请注意，一些操作，包括[[drop]]、[[dropWhile]]、[[flatMap]]或[[collect]]可以在返回前处理大量中间元素。

这里是另外一个例子。让我们开始的自然数，并在它们之间迭代。

```scala
// We'll start with a silly iteration
def loop(s: String, i: Int, iter: Iterator[Int]): Unit = {
  // Stop after 200,000
  if (i < 200001) {
    if (i % 50000 == 0) println(s + i)
    loop(s, iter.next(), iter)
  }
}

// Our first LazyList definition will be a val definition
val lazylist1: LazyList[Int] = {
  def loop(v: Int): LazyList[Int] = v #:: loop(v + 1)
  loop(0)
}

// Because lazylist1 is a val, everything that the iterator produces is held
// by virtue of the fact that the head of the LazyList is held in lazylist1
val it1 = lazylist1.iterator
loop("Iterator1: ", it1.next(), it1)

// We can redefine this LazyList such that all we have is the Iterator left
// and allow the LazyList to be garbage collected as required.  Using a def
// to provide the LazyList ensures that no val is holding onto the head as
// is the case with lazylist1
def lazylist2: LazyList[Int] = {
  def loop(v: Int): LazyList[Int] = v #:: loop(v + 1)
  loop(0)
}
val it2 = lazylist2.iterator
loop("Iterator2: ", it2.next(), it2)

// And, of course, we don't actually need a LazyList at all for such a simple
// problem.  There's no reason to use a LazyList if you don't actually need
// one.
val it3 = new Iterator[Int] {
  var i = -1
  def hasNext = true
  def next(): Int = { i += 1; i }
}
loop("Iterator3: ", it3.next(), it3)
```

在前面的示例示例中，有趣的是尾巴完全起作用。  fibs的首字母为（0，1，LazyList（...）），因此tail是确定性的。 如果我们将fibs定义为仅具体知道0，则确定尾部的动作将需要对尾部进行评估，**因此计算将无法进行**，如以下代码所示：

```scala
// The first time we try to access the tail we're going to need more
// information which will require us to recurse, which will require us to
// recurse, which...
lazy val sov: LazyList[Vector[Int]] = Vector(0) #:: sov.zip(sov.tail).map { n => n._1 ++ n._2 }
```

上面的fib定义会创建比所需数量更多的对象，具体取决于您可能希望如何实现。 由于以下事实可以更直接地路由到数字本身，因此以下实现提供了一种更具“成本效益”的实现：

```scala
lazy val fib: LazyList[Int] = {
  def loop(h: Int, n: Int): LazyList[Int] = h #:: loop(n, h + n)
  loop(1, 1)
}
```

头，尾以及列表是否为空最初可能是未知的。 一旦对其中任何一个进行了评估，它们都是众所周知的，尽管如果尾巴是使用＃::或＃:::构建的，则其内容仍然不会被评估。 取而代之的是，将评估尾部内容的时间推迟到评估尾部的空状态，头或尾部。

延迟对LazyList是否为空的评估直到需要时才允许，LazyList不急于评估过滤器调用中的任何元素。

仅当进一步评估（可能永远不会！）时，任何元素才会被强制执行。

```scala
def tailWithSideEffect: LazyList[Nothing] = {
  println("getting empty LazyList")
  LazyList.empty
}

val emptyTail = tailWithSideEffect // prints "getting empty LazyList"

val suspended = 1 #:: tailWithSideEffect // doesn't print anything
val tail = suspended.tail // although the tail is evaluated, *still* nothing is yet printed
val filtered = tail.filter(_ => false) // still nothing is printed
filtered.isEmpty // prints "getting empty LazyList"
```

### Value Members

#### final def ++[B >: A](suffix: IterableOnce[B]): List[B]
concat 的别名

#### final def ++:[B >: A](prefix: IterableOnce[B]): List[B]
prependedAll 的别名

#### final def +:[B >: A](elem: B): List[B]
prepended 的别名

#### final def :+[B >: A](elem: B): List[B]
appended 的别名

#### final def :++[B >: A](suffix: IterableOnce[B]): List[B]
appendedAll 的别名

#### def ::[B >: A](elem: B): List[B]
在此列表的开头添加一个元素。

#### def :::[B >: A](prefix: List[B]): List[B]
将给定列表的元素添加到此列表的前面。

#### final def addString(b: mutable.StringBuilder): mutable.StringBuilder
将此集合的所有元素追加到字符串生成器。

#### final def addString(b: mutable.StringBuilder, sep: String): mutable.StringBuilder
使用分隔符字符串将此集合的所有元素追加到字符串生成器。

#### def addString(b: mutable.StringBuilder, start: String, sep: String, end: String): mutable.StringBuilder
使用开始，结束和分隔符字符串将此集合的所有元素追加到字符串生成器。

#### def andThen[C](k: PartialFunction[A, C]): PartialFunction[Int, C]
用另一个应用于该部分函数结果的部分函数组成该部分函数。

#### def andThen[C](k: (A) => C): PartialFunction[Int, C]
用转换函数组成此部分函数，该转换函数将应用于该部分函数的结果。

#### def appended[B >: A](elem: B): List[B]
此序列的副本，其中附加了元素。

#### def appendedAll[B >: A](suffix: IterableOnce[B]): List[B]
返回一个新列表，其中包含左侧操作数中的元素，然后是右侧操作数中的元素。

#### def apply(n: Int): A
获取指定索引处的元素。

#### def applyOrElse[A1 <: Int, B1 >: A](x: A1, default: (A1) => B1): B1
当部分函数包含在函数域中时，将其应用于给定参数。

#### def canEqual(that: Any): Boolean
从相等方法调用的方法，以便用户定义的子类可以拒绝与其他相同类型的集合相等。

#### final def collect[B](pf: PartialFunction[A, B]): List[B]
通过将部分函数应用于此列表中定义了该函数的所有元素来构建新列表。

#### def collectFirst[B](pf: PartialFunction[A, B]): Option[B]
查找为其定义了给定局部函数的集合的第一个元素，并将局部函数应用于该元素。

#### def combinations(n: Int): Iterator[List[A]]
遍历组合。

#### def compose[R](k: PartialFunction[R, Int]): PartialFunction[R, A]
用此部分函数组成另一个部分函数k，以便将该部分函数应用于k的结果。

#### def compose[A](g: (A) => Int): (A) => A
在新的Function1中组成Function1的两个实例，最后应用此函数。

#### final def concat[B >: A](suffix: IterableOnce[B]): List[B]
返回一个新序列，该序列包含左侧操作数中的元素，然后是右侧操作数中的元素。

#### final def contains[A1 >: A](elem: A1): Boolean
测试此列表是否包含给定值作为元素。

#### def containsSlice[B](that: collection.Seq[B]): Boolean
测试此序列是否包含给定序列作为切片。

#### def copyToArray[B >: A](xs: Array[B], start: Int, len: Int): Int
将元素复制到数组，返回写入的元素数。

#### def copyToArray[B >: A](xs: Array[B], start: Int): Int
将元素复制到数组，返回写入的元素数。

#### def copyToArray[B >: A](xs: Array[B]): Int
将元素复制到数组，返回写入的元素数。

#### def corresponds[B](that: collection.Seq[B])(p: (A, B) => Boolean): Boolean
通过满足测试谓词来测试此列表中的每个元素是否与另一个序列的相应元素相关。

#### def corresponds[B](that: IterableOnce[B])(p: (A, B) => Boolean): Boolean
通过满足测试谓词，测试此集合的迭代器的每个元素是否与另一个集合的相应元素相关。

#### def count(p: (A) => Boolean): Int
计算集合中满足谓词的元素的数量。

#### def diff[B >: A](that: collection.Seq[B]): List[A]
计算此序列与另一个序列之间的多集差异。

#### def distinct: List[A]
选择此序列的所有元素，而忽略重复项。

#### def distinctBy[B](f: (A) => B): List[A]
在应用转换函数f之后，忽略此不变序列的所有元素，而忽略由==确定的重复项。

#### def drop(n: Int): List[A]
选择除前n个元素以外的所有元素。

#### def dropRight(n: Int): List[A]
集合的其余部分不包含最后n个元素。

#### def dropWhile(p: (A) => Boolean): List[A]
删除满足谓词的元素的最长前缀。

#### def elementWise: ElementWiseExtractor[Int, A]
返回带有unapplySeq方法的提取器对象，该方法提取序列数据的每个元素。

#### def empty: List[A]
与该迭代器相同类型的空迭代器

#### def endsWith[B >: A](that: collection.Iterable[B]): Boolean
测试此序列是否以给定序列结尾。

#### def equals(o: Any): Boolean
AnyRef中定义的通用相等方法。

#### final def exists(p: (A) => Boolean): Boolean
测试谓词是否对该列表的至少一个元素成立。

#### def filter(p: (A) => Boolean): List[A]
选择列表中满足谓词的所有元素。

#### def filterNot(p: (A) => Boolean): List[A]
选择此列表中不满足谓词的所有元素。

#### final def find(p: (A) => Boolean): Option[A]
查找满足谓词的列表的第一个元素（如果有）。

#### def findLast(p: (A) => Boolean): Option[A]
查找满足谓词的序列的最后一个元素（如果有）。

#### final def flatMap[B](f: (A) => IterableOnce[B]): List[B]
通过将函数应用于此列表的所有元素并使用结果集合的元素来构建新列表。

#### def flatten[B](implicit toIterableOnce: (A) => IterableOnce[B]): List[B]
将此可遍历集合的可迭代集合转换为由这些可遍历集合的元素形成的可迭代集合。

#### def fold[A1 >: A](z: A1)(op: (A1, A1) => A1): A1
使用指定的关联二进制运算符折叠此集合的元素。

#### def foldLeft[B](z: B)(op: (B, A) => B): B
将二进制运算符应用于起始值和该序列的所有元素（从左到右）。

#### final def foldRight[B](z: B)(op: (A, B) => B): B
将一个二元运算符应用于此列表的所有元素和一个从右到左的起始值。

#### final def forall(p: (A) => Boolean): Boolean
测试谓词是否对该列表的所有元素成立。

#### final def foreach[U](f: (A) => U): Unit
将f应用于每个元素，以产生副作用注意：需要[U]参数以帮助scalac进行类型推断。

#### def groupBy[K](f: (A) => K): Map[K, List[A]]
根据某些区分函数，将此可迭代集合划分为可迭代集合的映射。

#### def groupMap[K, B](key: (A) => K)(f: (A) => B): Map[K, List[B]]
根据区分函数功能键将此可迭代集合划分为可迭代集合的映射。

#### def groupMapReduce[K, B](key: (A) => K)(f: (A) => B)(reduce: (B, B) => B): Map[K, B]
根据区分函数功能键将此可迭代集合划分为映射。

#### def grouped(size: Int): Iterator[List[A]]
在固定大小的可迭代集合中对元素进行分区。

#### def hashCode(): Int
引用类型的hashCode方法。

#### val head: A
选择此列表的第一个元素。

#### def headOption: Some[A]
（可选）选择第一个元素。

#### def indexOf[B >: A](elem: B): Int
查找此序列中某个值首次出现的索引。

#### def indexOf[B >: A](elem: B, from: Int): Int
查找在此序列中某个起始索引之后或之后的某个值首次出现的索引。

#### def indexOfSlice[B >: A](that: collection.Seq[B]): Int
查找第一个索引，其中该序列包含给定序列作为切片。

#### def indexOfSlice[B >: A](that: collection.Seq[B], from: Int): Int
在此序列包含给定序列作为切片的起始索引之后或起始索引处查找第一个索引。

#### def indexWhere(p: (A) => Boolean, from: Int): Int
查找在起始索引之后或起始索引处满足某些谓词的第一个元素的索引。

#### def indexWhere(p: (A) => Boolean): Int
查找满足某些谓词的第一个元素的索引。

#### def indices: Range
产生此序列所有索引的范围。

#### def init: List[A]
集合的初始部分，没有最后一个元素。

#### def inits: Iterator[List[A]]
迭代此可迭代集合的init。

#### def intersect[B >: A](that: collection.Seq[B]): List[A]
计算此序列与另一个序列之间的多集交集。

#### def isdefinedAt(x: Int): Boolean
测试此序列是否包含给定索引。

#### final def isEmpty: Boolean
测试列表是否为空。

#### def isTraversableAgain: Boolean
测试此可迭代集合是否可以重复遍历。

#### def iterableFactory: SeqFactory[List]
此列表的伴随对象，提供了各种工厂方法。

#### def iterator: Iterator[A]
迭代器只能使用一次

#### def knownSize: Int

#### def last: A
选择最后一个元素。

#### def lastIndexOf[B >: A](elem: B, end: Int = length - 1): Int
查找在给定结束索引之前或之后的此序列中某个值最后一次出现的索引。

#### def lastIndexOfSlice[B >: A](that: collection.Seq[B]): Int
查找该序列包含给定序列作为切片的最后一个索引。

#### def lastIndexOfSlice[B >: A](that: collection.Seq[B], end: Int): Int
查找给定结束索引之前或之后的最后一个索引，其中此序列包含给定序列作为切片。

#### def lastIndexWhere(p: (A) => Boolean, end: Int): Int
查找在给定的结束索引之前或之后满足某些谓词的最后一个元素的索引。

#### def lastIndexWhere(p: (A) => Boolean): Int
查找满足某些谓词的最后一个元素的索引。

#### def lastOption: Option[A]
（可选）选择最后一个元素。

#### def lazyZip[B](that: collection.Iterable[B]): LazyZip2[A, B, ::.this.type]
与zip类似，除了每个集合中的元素只有在对返回的LazyZip2装饰器调用严格的操作后才会使用。

#### final def length: Int
列表的长度（元素数）。

#### final def lengthCompare(len: Int): Int
将此列表的长度与测试值进行比较。

#### def lengthCompare(that: collection.Iterable[_]): Int
比较此序列的长度和另一个Iterable的大小。

#### final def lengthIs: SizeCompareOps
返回一个包含用于将该序列的长度与测试值进行比较的操作的值类。

#### def lift: (Int) => Option[A]
将此部分函数转换为简单函数，返回Option结果。

#### final def map[B](f: (A) => B): List[B]
通过将函数应用于此列表的所有元素来构建新列表。

#### final def mapConserve[B >: A <: AnyRef](f: (A) => B): List[B]
通过将函数应用于此列表的所有元素来构建新列表。

#### def max[B >: A](implicit ord: math.Ordering[B]): A
查找最大的元素。

#### def maxBy[B](f: (A) => B)(implicit cmp: math.Ordering[B]): A
查找第一个元素，该元素产生通过函数f测量的最大值。

#### def maxByOption[B](f: (A) => B)(implicit cmp: math.Ordering[B]): Option[A]
查找第一个元素，该元素产生通过函数f测量的最大值。

#### def maxOption[B >: A](implicit ord: math.Ordering[B]): Option[A]
查找最大的元素。

#### def min[B >: A](implicit ord: math.Ordering[B]): A
查找最小的元素。

#### def minBy[B](f: (A) => B)(implicit cmp: math.Ordering[B]): A
查找第一个元素，该元素的最小值由函数f测量。

#### def minByOption[B](f: (A) => B)(implicit cmp: math.Ordering[B]): Option[A]
查找第一个元素，该元素的最小值由函数f测量。

#### def minOption[B >: A](implicit ord: math.Ordering[B]): Option[A]
查找最小的元素。

#### final def mkString: String
以字符串显示此集合的所有元素。

#### final def mkString(sep: String): String
使用分隔符字符串在字符串中显示此集合的所有元素。

#### final def mkString(start: String, sep: String, end: String): String
使用开始，结束和分隔符字符串以字符串形式显示此集合的所有元素。

#### def nonEmpty: Boolean
测试集合是否不为空。

#### def orElse[A1 <: Int, B1 >: A](that: PartialFunction[A1, B1]): PartialFunction[A1, B1]
将该部分功能与后备部分功能组成，该功能将在未定义此部分功能的情况下应用。

#### def padTo[B >: A](len: Int, elem: B): List[B]
此序列的副本，其中附加了元素值，直到达到给定的目标长度为止。

#### def partition(p: (A) => Boolean): (List[A], List[A])
一对，第一，所有满足谓词p的元素，第二，所有不满足谓词的元素。

#### def partitionMap[A1, A2](f: (A) => Either[A1, A2]): (List[A1], List[A2])
将函数f应用于可迭代集合的每个元素，并返回一对可迭代集合：第一个由f返回的值组成，这些值包装在scala.util.Left中，第二个由那些返回的值包装在scala.util.Left中。实用权。

#### def patch[B >: A](from: Int, other: IterableOnce[B], replaced: Int): List[B]
产生一个新的不可变序列，其中该不可变序列中的元素切片被另一个序列替换。

#### def permutations: Iterator[List[A]]
遍历不同的排列。

#### def prepended[B >: A](elem: B): List[B]
带有元素的列表的副本。

#### def prependedAll[B >: A](prefix: IterableOnce[B]): List[B]
与：++一样，返回一个新集合，其中包含左侧操作数中的元素，然后是右侧操作数中的元素。

#### def product[B >: A](implicit num: math.Numeric[B]): B
将这个集合的元素相乘。

#### def productElementNames: scala.Iterator[String]
该产品所有元素名称的迭代器。

#### def reduce[B >: A](op: (B, B) => B): B
使用指定的关联二进制运算符减少此集合的元素。

#### def reduceLeft[B >: A](op: (B, A) => B): B
将二进制运算符应用于此集合的所有元素，从左到右。

#### def reduceLeftOption[B >: A](op: (B, A) => B): Option[B]
（可选）将二进制运算符应用于此集合的所有元素，从左到右。

#### def reduceOption[B >: A](op: (B, B) => B): Option[B]
使用指定的关联二进制运算符减少此collection的元素（如果有）。

#### def reduceRight[B >: A](op: (A, B) => B): B
将二进制运算符应用于此集合的所有元素，从右到左。

#### def reduceRightOption[B >: A](op: (A, B) => B): Option[B]
（可选）将二进制运算符应用于此集合的所有元素，从右到左。

#### final def reverse: List[A]
返回具有相反顺序元素的新列表。

#### def reverseIterator: Iterator[A]
迭代器以相反的顺序产生元素。

#### def reverse_:::[B >: A](prefix: List[B]): List[B]
以相反的顺序在此列表的前面添加给定列表的元素。

#### def runWith[U](action: (A) => U): (Int) => Boolean
用动作函数组成该部分函数，​​该动作函数将应用于该部分函数的结果。

#### def sameElements[B >: A](that: IterableOnce[B]): Boolean
此集合的元素是否与该元素相同（并且顺序相同）？

#### def scan[B >: A](z: B)(op: (B, B) => B): List[B]
计算集合元素的前缀扫描。

#### def scanLeft[B](z: B)(op: (B, A) => B): List[B]
产生一个可迭代的集合，其中包含应用运算符从左到右的累积结果，包括初始值。

#### def scanRight[B](z: B)(op: (A, B) => B): List[B]
产生一个包含从右到左应用运算符的累积结果的集合。

#### def search[B >: A](elem: B, from: Int, to: Int)(implicit ord: Ordering[B]): SearchResult
在此排序序列的间隔内搜索特定元素。

#### def search[B >: A](elem: B)(implicit ord: Ordering[B]): SearchResult
在此排序序列中搜索特定元素。

#### def segmentLength(p: (A) => Boolean, from: Int): Int
计算从某个索引开始并且其元素都满足某些谓词的最长段的长度。

#### final def segmentLength(p: (A) => Boolean): Int
计算从第一个元素开始且其元素都满足某些谓词的最长段的长度。

#### final def size: Int
此序列的大小。

#### final def sizeCompare(that: collection.Iterable[_]): Int
比较此序列的大小与另一个Iterable的大小。

#### final def sizeCompare(otherSize: Int): Int
比较此序列的大小与测试值。

#### final def sizeIs: SizeCompareOps
返回一个包含用于将该可迭代集合的大小与测试值进行比较的操作的值类。

#### def slice(from: Int, until: Int): List[A]

#### def sliding(size: Int, step: Int): Iterator[List[A]]
通过在固定大小的块上传递“滑动窗口”来对元素进行分组（与分组方式不同，这与分组方式不同）。

#### def sliding(size: Int): Iterator[List[A]]
通过在固定大小的块上传递“滑动窗口”来对元素进行分组（与分组方式不同，这与分组方式不同）。

#### def sortBy[B](f: (A) => B)(implicit ord: Ordering[B]): List[A]
根据排序对此序列进行排序，排序是通过使用转换函数对隐式给定的Ordering进行转换而得出的。

#### def sortWith(lt: (A, A) => Boolean): List[A]
根据比较功能对此序列进行排序。

#### def sorted[B >: A](implicit ord: Ordering[B]): List[A]
根据顺序对该不可变序列进行排序。

#### final def span(p: (A) => Boolean): (List[A], List[A])
根据谓词将此列表拆分为前缀/后缀对。

#### def splitAt(n: Int): (List[A], List[A])
在给定位置将此列表拆分为一个前缀/后缀对。

#### def startsWith[B >: A](that: IterableOnce[B], offset: Int = 0): Boolean
测试此序列是否在给定索引处包含给定序列。

#### def stepper[S <: Stepper[_]](implicit shape: StepperShape[A, S]): S
返回此collection的元素的scala.collection.Stepper。

#### def sum[B >: A](implicit num: math.Numeric[B]): B
总结此集合的元素。

#### def tail: List[A]
集合的其余部分不包含第一个元素。

#### def tails: Iterator[List[A]]
遍历此序列的尾部。

#### def take(n: Int): List[A]
选择前n个元素。

#### def takeRight(n: Int): List[A]
包含此集合的最后n个元素的集合。

#### final def takeWhile(p: (A) => Boolean): List[A]
接受满足谓词的元素的最长前缀。

#### def tapEach[U](f: (A) => U): List[A]
将副作用函数应用于此集合中的每个元素。

#### def to[C1](factory: Factory[A, C1]): C1
给定一个集合工厂工厂，将此集合转换为当前元素类型A的适当表示形式。

#### def toArray[B >: A](implicit arg0: ClassTag[B]): Array[B]
将集合转换为数组。

#### final def toBuffer[B >: A]: Buffer[B]

#### def toIndexedSeq: IndexedSeq[A]

#### final def toIterable: ::.this.type

#### final def toList: List[A]

#### def toMap[K, V](implicit ev: <:<[A, (K, V)]): Map[K, V]

#### final def toSeq: ::.this.type

#### def toSet[B >: A]: Set[B]

#### def toString(): String
创建此对象的String表示形式。

#### def toVector: Vector[A]

#### def transpose[B](implicit asIterable: (A) => collection.Iterable[B]): List[List[B]]
将此可迭代集合的可迭代集合转换为可迭代集合的可迭代集合。

#### def unapply(a: Int): Option[A]
尝试从A提取模式匹配表达式中的B。

#### def unlift: PartialFunction[Int, B]
将可选函数转换为部分函数。

#### def unzip[A1, A2](implicit asPair: (A) => (A1, A2)): (List[A1], List[A2])
将此对的可迭代集合转换为每个对的前半部分和后半部分的两个集合。

#### def unzip3[A1, A2, A3](implicit asTriple: (A) => (A1, A2, A3)): (List[A1], List[A2], List[A3])
将此三元组的可迭代集合转换为每个三元组的第一个，第二个和第三个元素的三个集合。

#### def updated[B >: A](index: Int, elem: B): List[B]
此列表的副本，其中包含一个被替换的元素。

#### def view: SeqView[A]
对该集合的元素的看法。

#### def withFilter(p: (A) => Boolean): WithFilter[A, []List[]]
创建此可迭代集合的非严格过滤器。

#### def zip[B](that: IterableOnce[B]): List[(A, B)]
通过将成对的相应元素组合在一起，返回从此可迭代集合和另一个可迭代集合形成的可迭代集合。

#### def zipAll[A1 >: A, B](that: collection.Iterable[B], thisElem: A1, thatElem: B): List[(A1, B)]
通过将成对的相应元素组合在一起，返回从此可迭代集合和另一个可迭代集合形成的可迭代集合。

#### def zipWithIndex: List[(A, Int)]
用索引压缩此可迭代的集合。
