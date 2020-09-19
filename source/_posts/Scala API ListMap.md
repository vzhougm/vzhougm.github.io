---
title: Scala API ListMap
date: 2020-09-19 13:31:43
categories:
 - Scala
 - 集合框架
tags: 
 - scala
 - scala-api
 - collection
---
### Introduction
> sealed class ListMap[K, +V] extends AbstractMap[K, V] with SeqMap[K, V] with StrictOptimizedMapOps[K, V, ListMap, ListMap[K, V]] with MapFactoryDefaults[K, V, ListMap, Iterable] with DefaultSerializable

这个类实现不可变映射使用基于列表的数据结构。ListMap迭代器和遍历方法查看键值对在他们第一次插入的顺序。

条目存储在内部反向插入顺序，该装置的最新键是在该列表的头部。这样，如头部和尾部的方法是为O（n），而最后一个和init是O（1）。其它的操作，如在插入或取出的条目，也为O（n），这使得该集合只适合小数量的元件。

列表地图的实例表示空地图;它们可以通过直接调用的构造，或者通过将所述函数ListMap.empty被任一创建。


**K：此ListMap键的类型**

**V：与键相关联的类型的值**

#### Source
ListMap.scala


### Instance Constructors
newListMap()

### Value Members
#### `def +[V1 >: V](kv: (K, V1)): ListMap[K, V1]`
updated 的别名

#### `def ++[V2 >: V](xs: IterableOnce[(K, V2)]): ListMap[K, V2]`
concat 的别名

#### `final def ++[B >: (K, V)](suffix: IterableOnce[B]): Iterable[B]`
concat 的别名

#### final def -(key: K): ListMap[K, V]
removed 的别名

#### final def --(keys: IterableOnce[K]): ListMap[K, V]
removedAll 的别名

#### def addString(sb: mutable.StringBuilder, start: String, sep: String, end: String): mutable.StringBuilder
使用开始，结束和分隔符字符串将此映射的所有元素追加到字符串生成器。

#### final def addString(b: mutable.StringBuilder): mutable.StringBuilder
将此集合的所有元素追加到字符串生成器。

#### final def addString(b: mutable.StringBuilder, sep: String): mutable.StringBuilder
使用分隔符字符串将此集合的所有元素追加到字符串生成器。

#### def andThen[C](k: PartialFunction[V, C]): PartialFunction[K, C]
用另一个应用于该部分函数结果的部分函数组成该部分函数。

#### def andThen[C](k: (V) => C): PartialFunction[K, C]
用转换函数组成此部分函数，​​该转换函数将应用于该部分函数的结果。

#### def apply(key: K): V
检索与给定键关联的值。

#### def applyOrElse[K1 <: K, V1 >: V](x: K1, default: (K1) => V1): V1
当部分函数包含在函数域中时，将其应用于给定参数。

#### def canEqual(that: Any): Boolean
应该从每个设计良好的equals方法中调用的方法，该方法打开后会在子类中被覆盖。

#### def collect[K2, V2](pf: PartialFunction[(K, V), (K2, V2)]): ListMap[K2, V2]
通过将部分函数应用于此函数在其上定义的该映射的所有元素来构建新集合。

#### def collect[B](pf: PartialFunction[(K, V), B]): Iterable[B]
通过将部分函数应用于在其上定义了该函数的该可迭代集合的所有元素，来构建新的可迭代集合。

#### def collectFirst[B](pf: PartialFunction[(K, V), B]): Option[B]
查找为其定义了给定局部函数的集合的第一个元素，并将局部函数应用于该元素。

#### def compose[R](k: PartialFunction[R, K]): PartialFunction[R, V]
用此部分函数组成另一个部分函数k，以便将该部分函数应用于k的结果。

#### def compose[A](g: (A) => K): (A) => V
在新的Function1中组成Function1的两个实例，最后应用此函数。

#### def concat[V1 >: V](that: IterableOnce[(K, V1)]): ListMap[K, V1]
返回一个新的不可变映射，该映射包含左操作数的元素，后跟右操作数的元素。

#### def concat[B >: (K, V)](suffix: IterableOnce[B]): Iterable[B]
返回一个新的可迭代集合，该集合包含左操作数中的元素以及右操作数中的元素。

#### def contains(key: K): Boolean
测试此映射是否包含键的绑定。

#### def copyToArray[B >: (K, V)](xs: Array[B], start: Int, len: Int): Int
将元素复制到数组，返回写入的元素数。

#### def copyToArray[B >: (K, V)](xs: Array[B], start: Int): Int
将元素复制到数组，返回写入的元素数。

#### def copyToArray[B >: (K, V)](xs: Array[B]): Int
将元素复制到数组，返回写入的元素数。

#### def corresponds[B](that: IterableOnce[B])(p: ((K, V), B) => Boolean): Boolean
通过满足测试谓词，测试此集合的迭代器的每个元素是否与另一个集合的相应元素相关。

#### def count(p: ((K, V)) => Boolean): Int
计算集合中满足谓词的元素的数量。

#### def default(key: K): V
定义了默认值计算的Map，当钥匙没有在这里找到实现的方法抛出一个异常，但它可能在子类中重写返回。

#### def drop(n: Int): ListMap[K, V]
选择除前n个元素外的所有元素。

#### def dropRight(n: Int): ListMap[K, V]
集合的其余部分不包含最后n个元素。

#### def dropWhile(p: ((K, V)) => Boolean): ListMap[K, V]
删除满足谓词的元素的最长前缀。

#### def elementWise: ElementWiseExtractor[K, V]
返回带有unapplySeq方法的提取器对象，该方法提取序列数据的每个元素。

#### def empty: ListMap[K, V]
与该迭代器相同类型的空迭代器

#### def equals(o: Any): Boolean
AnyRef中定义的通用相等方法。

#### def exists(p: ((K, V)) => Boolean): Boolean
测试谓词是否对该集合的至少一个元素成立。

#### def filter(pred: ((K, V)) => Boolean): ListMap[K, V]
选择此可迭代集合中满足谓词的所有元素。

#### def filterNot(pred: ((K, V)) => Boolean): ListMap[K, V]
选择此可迭代集合中不满足谓词的所有元素。

#### def find(p: ((K, V)) => Boolean): Option[(K, V)]
查找满足谓词（如果有）的集合的第一个元素。

#### `def flatMap[K2, V2](f: ((K, V)) => IterableOnce[(K2, V2)]): ListMap[K2, V2]`
通过对Map的所有元素应用函数并使用结果集合的元素来构建新Map。

#### `def flatMap[B](f: ((K, V)) => IterableOnce[B]): Iterable[B]`
通过将函数应用于此可迭代集合的所有元素并使用所得集合的元素来构建新的可迭代集合。

#### def flatten[B](implicit toIterableOnce: ((K, V)) => IterableOnce[B]): Iterable[B]
将此可遍历集合的可迭代集合转换为由这些可遍历集合的元素形成的可迭代集合。

#### def fold[A1 >: (K, V)](z: A1)(op: (A1, A1) => A1): A1
使用指定的关联二进制运算符折叠此集合的元素。

#### def foldLeft[B](z: B)(op: (B, (K, V)) => B): B
将二进制运算符应用于起始值和该集合的所有元素，从左到右。

#### def foldRight[B](z: B)(op: ((K, V), B) => B): B
将二进制运算符应用于此集合的所有元素和一个从右到左的起始值。

#### def forall(p: ((K, V)) => Boolean): Boolean
测试谓词是否对该集合的所有元素成立。

#### `def foreach[U](f: ((K, V)) => U): Unit`
将f应用于每个元素的副作用注意：[U]参数需要帮助scalac的类型推断。

#### def foreachEntry[U](f: (K, V) => U): Unit
将f应用于每个键/值对，以产生副作用。注意：需要[U]参数以帮助进行scalac类型推断。

#### def get(key: K): Option[V]
（可选）返回与键关联的值。

#### def getOrElse[V1 >: V](key: K, default: => V1): V1
返回与键关联的值，如果映射中不包含键，则返回默认值。

#### `def groupBy[K](f: ((K, V)) => K): Map[K, ListMap[K, V]]`
根据某些区分函数，将此可迭代集合划分为可迭代集合的映射。

#### `def groupMap[K, B](key: ((K, V)) => K)(f: ((K, V)) => B): Map[K, Iterable[B]]`
根据标识符功能键将此可迭代集合划分为可迭代集合的映射。

#### `def groupMapReduce[K, B](key: ((K, V)) => K)(f: ((K, V)) => B)(reduce: (B, B) => B): Map[K, B]`
根据标识符功能键将此可迭代集合划分为映射。

#### def grouped(size: Int): Iterator[ListMap[K, V]]
在固定大小的可迭代集合中对元素进行分区。

#### def hashCode(): Int
引用类型的hashCode方法。

#### def head: (K, V)
选择此可迭代集合的第一个元素。

#### def headOption: Option[(K, V)]
（可选）选择第一个元素。

#### def init: ListMap[K, V]
集合的初始部分，没有最后一个元素。

#### def inits: Iterator[ListMap[K, V]]
迭代此可迭代集合的init。

#### def isdefinedAt(key: K): Boolean
测试此映射是否包含键的绑定。

#### def isEmpty: Boolean
测试列表映射是否为空。

#### def isTraversableAgain: Boolean
测试此可迭代集合是否可以重复遍历。

#### def iterableFactory: IterableFactory[Iterable]
此不可变集合的伴随对象，提供了各种工厂方法。

#### def iterator: Iterator[(K, V)]
迭代器只能使用一次

#### def keySet: Set[K]
集中收集该Map的所有键。

#### def keyStepper[S <: Stepper[_]](implicit shape: StepperShape[K, S]): S
返回此映射键的步进器。

#### def keys: Iterable[K]
以可迭代的方式收集此映射的所有键。

#### def keysIterator: Iterator[K]
为所有键创建一个迭代器。

#### def knownSize: Int
#### def last: (K, V)
选择最后一个元素。

#### def lastOption: Option[(K, V)]
（可选）选择最后一个元素。

#### def lazyZip[B](that: collection.Iterable[B]): LazyZip2[(K, V), B, ListMap.this.type]
与zip类似，除了每个集合中的元素只有在对返回的LazyZip2装饰器调用严格的操作后才会使用。

#### def lift: (K) => Option[V]
将此部分函数转换为简单函数，返回Option结果。

#### `def map[K2, V2](f: ((K, V)) => (K2, V2)): ListMap[K2, V2]`
通过对Map的所有元素应用功能来构建新Map。

#### `def map[B](f: ((K, V)) => B): Iterable[B]`
通过将函数应用于此可迭代集合的所有元素来构建新的可迭代集合。

#### def mapFactory: MapFactory[ListMap]
此Map的伴随对象，提供了各种工厂方法。

#### def max[B >: (K, V)](implicit ord: math.Ordering[B]): (K, V)
查找最大的元素。

#### `def maxBy[B](f: ((K, V)) => B)(implicit cmp: math.Ordering[B]): (K, V)`
查找第一个元素，该元素产生通过函数f测量的最大值。

#### `def maxByOption[B](f: ((K, V)) => B)(implicit cmp: math.Ordering[B]): Option[(K, V)]`
查找第一个元素，该元素产生通过函数f测量的最大值。

#### def maxOption[B >: (K, V)](implicit ord: math.Ordering[B]): Option[(K, V)]
查找最大的元素。

#### def min[B >: (K, V)](implicit ord: math.Ordering[B]): (K, V)
查找最小的元素。

#### `def minBy[B](f: ((K, V)) => B)(implicit cmp: math.Ordering[B]): (K, V)`
查找第一个元素，该元素的最小值由函数f测量。

#### `def minByOption[B](f: ((K, V)) => B)(implicit cmp: math.Ordering[B]): Option[(K, V)]`
查找第一个元素，该元素的最小值由函数f测量。

#### def minOption[B >: (K, V)](implicit ord: math.Ordering[B]): Option[(K, V)]
查找最小的元素。

#### final def mkString: String
以字符串显示此集合的所有元素。

#### final def mkString(sep: String): String
使用分隔符字符串以字符串形式显示此集合的所有元素。

#### final def mkString(start: String, sep: String, end: String): String
使用开始，结束和分隔符字符串以字符串形式显示此集合的所有元素。

#### def nonEmpty: Boolean
测试集合是否不为空。

#### def orElse[A1 <: K, B1 >: V](that: PartialFunction[A1, B1]): PartialFunction[A1, B1]
将此部分功能与后备部分功能组成，该功能将在未定义此部分功能的情况下应用。

#### def partition(p: ((K, V)) => Boolean): (ListMap[K, V], ListMap[K, V])
一对，第一，所有满足谓词p的元素，第二，所有不满足谓词的元素。

#### `def partitionMap[A1, A2](f: ((K, V)) => Either[A1, A2]): (Iterable[A1], Iterable[A2])`
将函数f应用于可迭代集合的每个元素，并返回一对可迭代集合：第一个由f返回的值组成，这些值包装在scala.util.Left中，第二个由那些返回的值包装在scala.util.Left中。实用权。

#### def product[B >: (K, V)](implicit num: math.Numeric[B]): B
将这个集合的元素相乘。

#### def reduce[B >: (K, V)](op: (B, B) => B): B
使用指定的关联二进制运算符减少此集合的元素。

#### `def reduceLeft[B >: (K, V)](op: (B, (K, V)) => B): B`
将二进制运算符应用于此集合的所有元素，从左到右。

#### `def reduceLeftOption[B >: (K, V)](op: (B, (K, V)) => B): Option[B]`
（可选）将二进制运算符应用于此集合的所有元素，从左到右。

#### def reduceOption[B >: (K, V)](op: (B, B) => B): Option[B]
使用指定的关联二进制运算符减少此collection的元素（如果有）。

#### def reduceRight[B >: (K, V)](op: ((K, V), B) => B): B
将二进制运算符应用于此集合的所有元素，从右到左。

#### def reduceRightOption[B >: (K, V)](op: ((K, V), B) => B): Option[B]
（可选）将二进制运算符应用于此集合的所有元素，从右到左。

#### def removed(key: K): ListMap[K, V]
从该Map中删除键，并返回一个新Map。

#### def removedAll(keys: IterableOnce[K]): ListMap[K, V]
通过删除另一个集合的所有元素，从该不可变映射表创建一个新的不可变映射表。

#### def runWith[U](action: (V) => U): (K) => Boolean
用动作函数组成该部分函数，​​该动作函数将应用于该部分函数的结果。

#### def scan[B >: (K, V)](z: B)(op: (B, B) => B): Iterable[B]
计算集合元素的前缀扫描。

#### def scanLeft[B](z: B)(op: (B, (K, V)) => B): Iterable[B]
产生一个可迭代的集合，其中包含应用运算符从左到右的累积结果，包括初始值。

#### def scanRight[B](z: B)(op: ((K, V), B) => B): Iterable[B]
产生一个包含从右到左应用运算符的累积结果的集合。

#### def size: Int
该列表图的大小。

#### def sizeCompare(that: collection.Iterable[_]): Int
比较此可迭代集合的大小与另一个可迭代集合的大小。

#### def sizeCompare(otherSize: Int): Int
将此可迭代集合的大小与测试值进行比较。

#### final def sizeIs: SizeCompareOps
返回一个包含用于将该可迭代集合的大小与测试值进行比较的操作的值类。

#### def slice(from: Int, until: Int): ListMap[K, V]
选择元素的间隔。

#### def sliding(size: Int, step: Int): Iterator[ListMap[K, V]]
通过在固定大小的块上传递“滑动窗口”来对元素进行分组（与分组方式不同，这与对它们进行分区相反）。

#### def sliding(size: Int): Iterator[ListMap[K, V]]
通过在固定大小的块上传递“滑动窗口”来对元素进行分组（与分组方式不同，这与对它们进行分区相反）。

#### def span(p: ((K, V)) => Boolean): (ListMap[K, V], ListMap[K, V])
根据谓词将此可迭代集合拆分为前缀/后缀对。

#### def splitAt(n: Int): (ListMap[K, V], ListMap[K, V])
在给定位置将此可迭代集合拆分为一个前缀/后缀对。

#### def stepper[S <: Stepper[_]](implicit shape: StepperShape[(K, V), S]): S
返回此collection的元素的scala.collection.Stepper。

#### def sum[B >: (K, V)](implicit num: math.Numeric[B]): B
总结此集合的元素。

#### def tail: ListMap[K, V]
集合的其余部分不包含第一个元素。

#### def tails: Iterator[ListMap[K, V]]
迭代此可迭代集合的尾巴。

#### def take(n: Int): ListMap[K, V]
选择前n个元素。

#### def takeRight(n: Int): ListMap[K, V]
包含此集合的最后n个元素的集合。

#### def takeWhile(p: ((K, V)) => Boolean): ListMap[K, V]
接受满足谓词的元素的最长前缀。

#### `def tapEach[U](f: ((K, V)) => U): ListMap[K, V]`
将副作用函数应用于此集合中的每个元素。

#### def to[C1](factory: Factory[(K, V), C1]): C1
给定一个集合工厂工厂，将此集合转换为当前元素类型A的适当表示形式。

#### def toArray[B >: (K, V)](implicit arg0: ClassTag[B]): Array[B]
将集合转换为数组。

#### final def toBuffer[B >: (K, V)]: Buffer[B]
#### def toIndexedSeq: IndexedSeq[(K, V)]
#### final def toIterable: ListMap.this.type
#### def toList: List[(K, V)]
#### final def toMap[K2, V2](implicit ev: <:<[(K, V), (K2, V2)]): Map[K2, V2]
#### def toSeq: Seq[(K, V)]
#### def toSet[B >: (K, V)]: Set[B]
#### def toString(): String
创建此对象的String表示形式。

#### def toVector: Vector[(K, V)]
#### def transform[W](f: (K, V) => W): ListMap[K, W]
此函数使用函数f转换此映射中包含的映射的所有值。

#### def transpose[B](implicit asIterable: ((K, V)) => collection.Iterable[B]): Iterable[Iterable[B]]
将此可迭代集合的可迭代集合转换为可迭代集合的可迭代集合。

#### def unapply(a: K): Option[V]
尝试从A提取模式匹配表达式中的B。

#### def unlift: PartialFunction[K, B]
将可选函数转换为部分函数。

#### def unzip[A1, A2](implicit asPair: ((K, V)) => (A1, A2)): (Iterable[A1], Iterable[A2])
将此对的可迭代集合转换为每个对的前半部分和后半部分的两个集合。

#### def unzip3[A1, A2, A3](implicit asTriple: ((K, V)) => (A1, A2, A3)): (Iterable[A1], Iterable[A2], Iterable[A3])
将此三元组的可迭代集合转换为每个三元组的第一个，第二个和第三个元素的三个集合。

#### def updated[V1 >: V](key: K, value: V1): ListMap[K, V1]
创建一个新Map，该Map是通过使用给定的键/值对更新此Map而获得的。

#### def updatedWith[V1 >: V](key: K)(remappingFunction: (Option[V]) => Option[V1]): ListMap[K, V1]
更新指定键及其当前可选映射值的映射（如果存在当前映射，则为某些，否则为None）。

#### def valueStepper[S <: Stepper[_]](implicit shape: StepperShape[V, S]): S
返回此映射值的步进器。

#### def values: collection.Iterable[V]
在可迭代的集合中收集此映射的所有值。

#### def valuesIterator: Iterator[V]
为该映射中的所有值创建一个迭代器。

#### def view: MapView[K, V]
对该集合的元素的看法。

#### def with default[V1 >: V](d: (K) => V1): Map[K, V1]
具有给定默认功能的相同映射。

#### def with defaultValue[V1 >: V](d: V1): Map[K, V1]
具有给定默认值的相同映射。

#### def withFilter(p: ((K, V)) => Boolean): MapOps.WithFilter[K, V, [x]Iterable[x], [x, y]ListMap[x, y]]
创建此Map的非严格过滤器。

#### def zip[B](that: IterableOnce[B]): Iterable[((K, V), B)]
通过将成对的相应元素组合在一起，返回从此可迭代集合和另一个可迭代集合形成的可迭代集合。

#### def zipAll[A1 >: (K, V), B](that: collection.Iterable[B], thisElem: A1, thatElem: B): Iterable[(A1, B)]
通过将成对的相应元素组合在一起，返回从此可迭代集合和另一个可迭代集合形成的可迭代集合。

#### def zipWithIndex: Iterable[((K, V), Int)]
用索引压缩此可迭代的集合。
