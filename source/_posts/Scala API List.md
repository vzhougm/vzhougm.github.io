---
title: Scala API List
date: 2020-09-18 13:03:34
categories:
 - Scala
 - 集合框架
tags: 
 - scala
 - scala-api
 - collection
---
### Introduction
>  sealed abstract class List[+A] extends AbstractSeq[A] with LinearSeq[A] with LinearSeqOps[A, List, List[A]] with StrictOptimizedLinearSeqOps[A, List, List[A]] with StrictOptimizedSeqOps[A, List, List[A]] with IterableFactoryDefaults[A, List] with DefaultSerializable

不可变链表的类，表示A型元素的有序集合。

此类带有两个实现案例的类scala.Nil和scala。::，它们实现抽象成员isEmpty，head和tail。

此类对于后进先出（LIFO），类似堆栈的访问模式是最佳的。 
如果您需要其他访问模式，例如随机访问或FIFO，请考虑使用一个比List更适合此访问的集合。

#### 性能
**时间：**列表具有O（1）前置和头/尾访问。 列表中元素的数量大多数为O（n）。 
这包括基于索引的元素查找，长度，附加和反向。

**空间：**列表实现尾部列表的结构共享。 这意味着许多操作要么是零内存成本，要么是恒定内存成本。

```scala
val mainList = List(3, 2, 1)
val with4 =    4 :: mainList  // re-uses mainList, costs one :: instance
val with42 =   42 :: mainList // also re-uses mainList, cost one :: instance
val shorter =  mainList.tail  // costs nothing as it uses the same 2::1::Nil instances as mainList
```

##### Annotations
`@SerialVersionUID()`

##### Source
`List.scala`

##### Example

```scala
// Make a list via the companion object factory
val days = List("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")

// Make a list element-by-element
val when = "AM" :: "PM" :: Nil

// Pattern match
days match {
  case firstDay :: otherDays =>
    println("The first day of the week is: " + firstDay)
  case Nil =>
    println("There don't seem to be any week days.")
}
```

##### Note
功能列表的特点是持久性和结构共享，因此，如果使用正确，则在某些情况下会提供可观的性能和空间消耗优势。 
但是，请注意，对同一功能列表具有多个引用的对象（即，依赖于结构共享的对象）将使用多个列表进行序列化和反序列化，每个列表对应一个引用。 即 序列化/反序列化后，结构共享丢失。

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
用转换函数组成此部分函数，​​该转换函数将应用于该部分函数的结果。

#### def appended[B >: A](elem: B): List[B]
此序列的副本，其中附加了元素。

#### def appendedAll[B >: A](suffix: IterableOnce[B]): List[B]
返回一个新列表，该列表包含左侧操作数中的元素，然后是右侧操作数中的元素。

#### def apply(n: Int): A
获取指定索引处的元素。

#### def applyOrElse[A1 <: Int, B1 >: A](x: A1, default: (A1) => B1): B1
当部分函数包含在函数域中时，将其应用于给定参数。

#### def canEqual(that: Any): Boolean
从相等方法调用的方法，以便用户定义的子类可以拒绝与其他相同类型的集合相等。

#### final def collect[B](pf: PartialFunction[A, B]): List[B]
通过将部分函数应用于此列表中定义该函数的所有元素来构建新列表。

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
选择除前n个元素外的所有元素。

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
将f应用于每个元素的副作用注意：[U]参数需要帮助scalac的类型推断。

#### def groupBy[K](f: (A) => K): Map[K, List[A]]
根据某些区分函数，将此可迭代集合划分为可迭代集合的映射。

#### def groupMap[K, B](key: (A) => K)(f: (A) => B): Map[K, List[B]]
根据标识符功能键将此可迭代集合划分为可迭代集合的映射。

#### def groupMapReduce[K, B](key: (A) => K)(f: (A) => B)(reduce: (B, B) => B): Map[K, B]
根据标识符功能键将此可迭代集合划分为映射。

#### def grouped(size: Int): Iterator[List[A]]
在固定大小的可迭代集合中对元素进行分区。

#### def hashCode(): Int
引用类型的hashCode方法。

#### def head: A
选择此可迭代集合的第一个元素。

#### def headOption: Option[A]
（可选）选择第一个元素。

#### def indexOf[B >: A](elem: B): Int
查找此序列中某个值首次出现的索引。

#### def indexOf[B >: A](elem: B, from: Int): Int
在此序列中某个起始索引之后或起始索引处查找该值首次出现的索引。

#### def indexOfSlice[B >: A](that: collection.Seq[B]): Int
查找该序列包含给定序列作为切片的第一个索引。

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

#### def lazyZip[B](that: collection.Iterable[B]): LazyZip2[A, B, List.this.type]
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
使用分隔符字符串以字符串形式显示此集合的所有元素。

#### final def mkString(start: String, sep: String, end: String): String
使用开始，结束和分隔符字符串以字符串形式显示此集合的所有元素。

#### def nonEmpty: Boolean
测试集合是否不为空。

#### def orElse[A1 <: Int, B1 >: A](that: PartialFunction[A1, B1]): PartialFunction[A1, B1]
将此部分功能与后备部分功能组成，该功能将在未定义此部分功能的情况下应用。

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
通过在固定大小的块上传递元素“滑动窗口”来对它们进行分组（这与对它们进行分区相反，就像分组一样）。

#### def sliding(size: Int): Iterator[List[A]]
通过在固定大小的块上传递“滑动窗口”来对元素进行分组（与分组方式不同，这与对它们进行分区相反）。

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

#### final def toIterable: List.this.type

#### final def toList: List[A]

#### def toMap[K, V](implicit ev: <:<[A, (K, V)]): Map[K, V]

#### final def toSeq: List.this.type

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

#### def withFilter(p: (A) => Boolean): WithFilter[A, [_]List[_]]
创建此可迭代集合的非严格过滤器。

#### def zip[B](that: IterableOnce[B]): List[(A, B)]
通过将成对的相应元素组合在一起，返回从此可迭代集合和另一个可迭代集合形成的可迭代集合。

#### def zipAll[A1 >: A, B](that: collection.Iterable[B], thisElem: A1, thatElem: B): List[(A1, B)]
通过将成对的相应元素组合在一起，返回从此可迭代集合和另一个可迭代集合形成的可迭代集合。

#### def zipWithIndex: List[(A, Int)]
用索引压缩此可迭代的集合。
