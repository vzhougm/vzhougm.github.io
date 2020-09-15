---
title: Scala API LazyList
date: 2020-09-15 19:00:00
comments: true
categories:
 - Scala
 - 集合框架
tags: 
 - api
 - scala
 - collection
---

> This class implements an immutable linked list. We call it "lazy" because it computes its elements only when they are needed.
> Elements are memoized; that is, the value of each element is computed at most once.
>
> Elements are computed in-order and are never skipped. In other words, accessing the tail causes the head to be computed first.
>
> How lazy is a `LazyList`? When you have a value of type `LazyList`, you don't know yet whether the list is empty or not. If you learn that it is non-empty, then you also know that the head has been computed. But the tail is itself a `LazyList`, whose emptiness-or-not might remain undetermined.
>
> A `LazyList` may be infinite. For example, `LazyList.from(0)` contains all of the natural numbers 0, 1, 2, and so on. For infinite sequences, some methods (such as `count`, `sum`, `max` or `min`) will not terminate.
>
> 这个类实现了一个不可变链表。我们称它为“懒”，因为仅在需要时，它才会计算它的元素。
>
> 元素是记忆化的；也就是说，每个元素的值最多计算一次。元素是按顺序计算的，不会被跳过。换句话说，**访问尾部会先计算头部。**
>
> 一个`LazyList`有多懒？当你有一个类型为`LazyList`的值时，你还不知道这个列表是否为空。如果你知道它是非空的，那么你也知道头已经被计算过了。但如果最后一个元素是一个`LazyList`，它是否为空是未确定的。
>
> 一个`LazyList`是无限的。例如，`LazyList.from(0)`包含所有自然数0、1、2等等。对于无穷序列，某些方法(如'count'、'sum'、'max'或'min')将不会终止。

example：

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

- 尽管“LazyList”在被访问时改变，但这并不与它的不变性相矛盾。价值观一旦被记忆，就不会改变。还没有被记忆的值仍然“存在”，它们只是还没有被计算。

虽然LazyList改变，因为它是访问，这并不违反其不变性。一旦值被记忆化它们就不会改变。这还没有值仍有待被记忆化的“存在”，它们根本就没有被计算呢。

我们必须谨慎值的记忆化；如果你不小心，它会消耗内存。这是因为LazyList的记忆化创建了一个类似[[scala.collection.immutable.List]]的结构。只要将其赋值给了对象，那它就从头部到尾部的数据一直保持。如果没有赋值到对象（例如，我们用def来定义LazyList），那么一旦它不再被使用，它就会消失。

请注意，一些操作，包括[[drop]]、[[dropWhile]]、[[flatMap]]或[[collect]]可以在返回前处理大量中间元素。

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

#### def #::[B >: A](elem: => B): LazyList[B]

构造一个LazyList，该LazyList由给定的第一个元素和随后的另一个LazyList的元素组成。

#### def #:::[B >: A](prefix: LazyList[B]): LazyList[B]

构造一个由给定LazyList和另一个LazyList的串联组成的LazyList。

#### final def ++[B >: A](suffix: IterableOnce[B]): LazyList[B]

concat 的别名

#### final def ++:[B >: A](prefix: IterableOnce[B]): LazyList[B]

prependedAll 的别名

#### final def +:[B >: A](elem: B): LazyList[B]

prepended 的别名

#### final def :+[B >: A](elem: B): LazyList[B]

appended 的别名

#### final def :++[B >: A](suffix: IterableOnce[B]): LazyList[B]

appendedAll 的别名

#### def addString(sb: mutable.StringBuilder, start: String, sep: String, end: String): mutable.StringBuilder

使用开始，结束和分隔符字符串将此延迟列表的所有元素追加到字符串生成器。

#### final def addString(b: mutable.StringBuilder): mutable.StringBuilder

将此集合的所有元素追加到字符串生成器。

#### final def addString(b: mutable.StringBuilder, sep: String): mutable.StringBuilder

使用分隔符字符串将此集合的所有元素追加到字符串生成器。

#### def andThen[C](k: PartialFunction[A, C]): PartialFunction[Int, C]

用另一个应用于该部分函数结果的部分函数组成该部分函数。用另一个应用于该部分函数结果的部分函数组成该部分函数。

#### def andThen[C](k: (A) => C): PartialFunction[Int, C]

用转换函数组成此部分函数，该转换函数将应用于该部分函数的结果。

#### def appended[B >: A](elem: B): LazyList[B]

此懒惰列表的副本，附加了一个元素。

#### def appendedAll[B >: A](suffix: IterableOnce[B]): LazyList[B]

Returns a new lazy list containing the elements from the left hand operand followed by the elements from the right hand operand.

#### def apply(n: Int): A

Get the element at the specified index.

#### def applyOrElse[A1 <: Int, B1 >: A](x: A1, #### def ault: (A1) => B1): B1

Applies this partial function to the given argument when it is contained in the function domain.

#### def canEqual(that: Any): Boolean

Method called from equality methods, so that user-#### def ined subclasses can refuse to be equal to other collections of the same kind.

#### def collect[B](pf: PartialFunction[A, B]): LazyList[B]

Builds a new lazy list by applying a partial function to all elements of this lazy list on which the function isdefined.

#### def collectFirst[B](pf: PartialFunction[A, B]): Option[B]

Finds the first element of the lazy list for which the given partial function isdefined, and applies the partial function to it.

#### def combinations(n: Int): Iterator[LazyList[A]]

Iterates over combinations.

#### def compose[R](k: PartialFunction[R, Int]): PartialFunction[R, A]

Composes another partial function k with this partial function so that this partial function gets applied to results of k.

#### def compose[A](g: (A) => Int): (A) => A

Composes two instances of Function1 in a new Function1, with this function applied last.

#### final def concat[B >: A](suffix: IterableOnce[B]): LazyList[B]

Returns a new sequence containing the elements from the left hand operand followed by the elements from the right hand operand.

#### def contains[A1 >: A](elem: A1): Boolean

Tests whether this sequence contains a given value as an element.

#### def containsSlice[B](that: collection.Seq[B]): Boolean

Tests whether this sequence contains a given sequence as a slice.

#### def copyToArray[B >: A](xs: Array[B], start: Int, len: Int): Int

Copy elements to an array, returning the number of elements written.

#### def copyToArray[B >: A](xs: Array[B], start: Int): Int

Copy elements to an array, returning the number of elements written.

#### def copyToArray[B >: A](xs: Array[B]): Int

Copy elements to an array, returning the number of elements written.

#### def corresponds[B](that: collection.Seq[B])(p: (A, B) => Boolean): Boolean

Tests whether every element of this sequence relates to the corresponding element of another sequence by satisfying a test predicate.

#### def corresponds[B](that: IterableOnce[B])(p: (A, B) => Boolean): Boolean

Tests whether every element of this collection's iterator relates to the corresponding element of another collection by satisfying a test predicate.

#### def count(p: (A) => Boolean): Int

Counts the number of elements in the collection which satisfy a predicate.

#### def diff[B >: A](that: collection.Seq[B]): LazyList[A]

Computes the multiset difference between this lazy list and another sequence.

#### def distinct: LazyList[A]

Selects all the elements of this sequence ignoring the duplicates.

#### def distinctBy[B](f: (A) => B): LazyList[A]

Selects all the elements of this sequence ignoring the duplicates as determined by == after applying the transforming function f.

#### def drop(n: Int): LazyList[A]

Selects all elements except first n ones.

#### def dropRight(n: Int): LazyList[A]

Selects all elements except last n ones.

#### def dropWhile(p: (A) => Boolean): LazyList[A]

Drops longest prefix of elements that satisfy a predicate.

#### def elementWise: ElementWiseExtractor[Int, A]

Returns an extractor object with a unapplySeq method, which extracts each element of a sequence data.

#### def empty: LazyList[A]

The empty iterable of the same type as this iterable

#### def endsWith[B >: A](that: collection.Iterable[B]): Boolean

Tests whether this sequence ends with the given sequence.

#### def equals(that: Any): Boolean

The universal equality method #### def ined in AnyRef.

#### def exists(p: (A) => Boolean): Boolean

Tests whether a predicate holds for at least one element of this sequence.

#### def filter(pred: (A) => Boolean): LazyList[A]

Selects all elements of this lazy list which satisfy a predicate.

#### def filterNot(pred: (A) => Boolean): LazyList[A]

Selects all elements of this lazy list which do not satisfy a predicate.

#### def find(p: (A) => Boolean): Option[A]

Finds the first element of the lazy list satisfying a predicate, if any.

#### def findLast(p: (A) => Boolean): Option[A]

Finds the last element of the sequence satisfying a predicate, if any.

#### def flatMap[B](f: (A) => IterableOnce[B]): LazyList[B]

Builds a new lazy list by applying a function to all elements of this lazy list and using the elements of the resulting collections.

#### def flatten[B](implicit asIterable: (A) => IterableOnce[B]): LazyList[B]

Converts this lazy list of traversable collections into a lazy list formed by the elements of these traversable collections.

#### def fold[A1 >: A](z: A1)(op: (A1, A1) => A1): A1

Folds the elements of this collection using the specified associative binary operator.

#### def foldLeft[B](z: B)(op: (B, A) => B): B

LazyList specialization of foldLeft which allows GC to collect along the way.

#### def foldRight[B](z: B)(op: (A, B) => B): B

Applies a binary operator to all elements of this collection and a start value, going right to left.

#### def forall(p: (A) => Boolean): Boolean

Tests whether a predicate holds for all elements of this sequence.

#### def force: LazyList.this.type

Evaluates all un#### def ined elements of the lazy list.

#### def foreach[U](f: (A) => U): Unit

Apply the given function f to each element of this linear sequence (while respecting the order of the elements).

#### def groupBy[K](f: (A) => K): Map[K, LazyList[A]]

Partitions this iterable collection into a map of iterable collections according to some discriminator function.

#### def groupMap[K, B](key: (A) => K)(f: (A) => B): Map[K, LazyList[B]]

Partitions this iterable collection into a map of iterable collections according to a discriminator function key.

#### def groupMapReduce[K, B](key: (A) => K)(f: (A) => B)(reduce: (B, B) => B): Map[K, B]

Partitions this iterable collection into a map according to a discriminator function key.

#### def grouped(size: Int): Iterator[LazyList[A]]

Partitions elements in fixed size lazy lists.

#### def hashCode(): Int

The hashCode method for reference types.

#### def head: A

Selects the first element of this lazy list.

#### def headOption: Option[A]

Optionally selects the first element.

#### def indexOf[B >: A](elem: B): Int

Finds index of first occurrence of some value in this sequence.

#### def indexOf[B >: A](elem: B, from: Int): Int

Finds index of first occurrence of some value in this sequence after or at some start index.

#### def indexOfSlice[B >: A](that: collection.Seq[B]): Int

Finds first index where this sequence contains a given sequence as a slice.

#### def indexOfSlice[B >: A](that: collection.Seq[B], from: Int): Int

Finds first index after or at a start index where this sequence contains a given sequence as a slice.

#### def indexWhere(p: (A) => Boolean, from: Int): Int

Finds index of the first element satisfying some predicate after or at some start index.

#### def indexWhere(p: (A) => Boolean): Int

Finds index of the first element satisfying some predicate.

#### def indices: Range

Produces the range of all indices of this sequence.

#### def init: LazyList[A]

The initial part of the collection without its last element.

#### def inits: Iterator[LazyList[A]]

Iterates over the inits of this iterable collection.

#### def intersect[B >: A](that: collection.Seq[B]): LazyList[A]

Computes the multiset intersection between this lazy list and another sequence.

#### def is#### def inedAt(x: Int): Boolean

Tests whether this sequence contains given index.

#### def isEmpty: Boolean

Tests whether the lazy list is empty.

#### def isTraversableAgain: Boolean

Tests whether this iterable collection can be repeatedly traversed.

#### def iterableFactory: SeqFactory[LazyList]

The companion object of this lazy list, providing various factory methods.

#### def iterator: Iterator[A]

Iterator can be used only once

#### def knownSize: Int

This method preserves laziness; elements are only evaluated individually as needed.

#### def last: A

Selects the last element.

#### def lastIndexOf[B >: A](elem: B, end: Int = length - 1): Int

Finds index of last occurrence of some value in this sequence before or at a given end index.

#### def lastIndexOfSlice[B >: A](that: collection.Seq[B]): Int

Finds last index where this sequence contains a given sequence as a slice.

#### def lastIndexOfSlice[B >: A](that: collection.Seq[B], end: Int): Int

Finds last index before or at a given end index where this sequence contains a given sequence as a slice.

#### def lastIndexWhere(p: (A) => Boolean, end: Int): Int

Finds index of last element satisfying some predicate before or at given end index.

#### def lastIndexWhere(p: (A) => Boolean): Int

Finds index of last element satisfying some predicate.

#### def lastOption: Option[A]

Optionally selects the last element.

#### def lazyAppendedAll[B >: A](suffix: => IterableOnce[B]): LazyList[B]

The lazy list resulting from the concatenation of this lazy list with the argument lazy list.

#### def lazyZip[B](that: collection.Iterable[B]): LazyZip2[A, B, LazyList.this.type]

Analogous to zip except that the elements in each collection are not consumed until a strict operation is invoked on the returned LazyZip2 decorator.

#### def length: Int

The length (number of elements) of the sequence.

#### def lengthCompare(that: collection.Iterable[_]): Int

Compares the length of this sequence to the size of another Iterable.

#### def lengthCompare(len: Int): Int

Compares the length of this sequence to a test value.

#### final def lengthIs: SizeCompareOps

Returns a value class containing operations for comparing the length of this sequence to a test value.

#### def lift: (Int) => Option[A]

Turns this partial function into a plain function returning an Option result.

#### def map[B](f: (A) => B): LazyList[B]

Builds a new lazy list by applying a function to all elements of this lazy list.

#### def max[B >: A](implicit ord: math.Ordering[B]): A

Finds the largest element.

#### def maxBy[B](f: (A) => B)(implicit cmp: math.Ordering[B]): A

Finds the first element which yields the largest value measured by function f.

#### def maxByOption[B](f: (A) => B)(implicit cmp: math.Ordering[B]): Option[A]

Finds the first element which yields the largest value measured by function f.

#### def maxOption[B >: A](implicit ord: math.Ordering[B]): Option[A]

Finds the largest element.

#### def min[B >: A](implicit ord: math.Ordering[B]): A

Finds the smallest element.

#### def minBy[B](f: (A) => B)(implicit cmp: math.Ordering[B]): A

Finds the first element which yields the smallest value measured by function f.

#### def minByOption[B](f: (A) => B)(implicit cmp: math.Ordering[B]): Option[A]

Finds the first element which yields the smallest value measured by function f.

#### def minOption[B >: A](implicit ord: math.Ordering[B]): Option[A]

Finds the smallest element.

#### final def mkString: String

Displays all elements of this collection in a string.

#### final def mkString(sep: String): String

Displays all elements of this collection in a string using a separator string.

#### final def mkString(start: String, sep: String, end: String): String

Displays all elements of this collection in a string using start, end, and separator strings.

#### def nonEmpty: Boolean

Tests whether the collection is not empty.

#### def orElse[A1 <: Int, B1 >: A](that: PartialFunction[A1, B1]): PartialFunction[A1, B1]

Composes this partial function with a fallback partial function which gets applied where this partial function is not #### def ined.

#### def padTo[B >: A](len: Int, elem: B): LazyList[B]

A copy of this lazy list with an element value appended until a given target length is reached.

#### def partition(p: (A) => Boolean): (LazyList[A], LazyList[A])

A pair of, first, all elements that satisfy predicate p and, second, all elements that do not.

#### def partitionMap[A1, A2](f: (A) => Either[A1, A2]): (LazyList[A1], LazyList[A2])

Applies a function f to each element of the lazy list and returns a pair of lazy lists: the first one made of those values returned by f that were wrapped in scala.util.Left, and the second one made of those wrapped in scala.util.Right.

#### def patch[B >: A](from: Int, other: IterableOnce[B], replaced: Int): LazyList[B]

Produces a new lazy list where a slice of elements in this lazy list is replaced by another sequence.

#### def permutations: Iterator[LazyList[A]]

Iterates over distinct permutations.

#### def prepended[B >: A](elem: B): LazyList[B]

A copy of the lazy list with an element prepended.

#### def prependedAll[B >: A](prefix: IterableOnce[B]): LazyList[B]

As with :++, returns a new collection containing the elements from the left operand followed by the elements from the right operand.

#### def product[B >: A](implicit num: math.Numeric[B]): B

Multiplies up the elements of this collection.

#### def reduce[B >: A](op: (B, B) => B): B

Reduces the elements of this collection using the specified associative binary operator.

#### def reduceLeft[B >: A](f: (B, A) => B): B

LazyList specialization of reduceLeft which allows GC to collect along the way.

#### def reduceLeftOption[B >: A](op: (B, A) => B): Option[B]

Optionally applies a binary operator to all elements of this collection, going left to right.

#### def reduceOption[B >: A](op: (B, B) => B): Option[B]

Reduces the elements of this collection, if any, using the specified associative binary operator.

#### def reduceRight[B >: A](op: (A, B) => B): B

Applies a binary operator to all elements of this collection, going right to left.

#### def reduceRightOption[B >: A](op: (A, B) => B): Option[B]

Optionally applies a binary operator to all elements of this collection, going right to left.

#### def reverse: LazyList[A]

Returns new lazy list with elements in reversed order.

#### def reverseIterator: Iterator[A]

An iterator yielding elements in reversed order.

#### def runWith[U](action: (A) => U): (Int) => Boolean

Composes this partial function with an action function which gets applied to results of this partial function.

#### def sameElements[B >: A](that: IterableOnce[B]): Boolean

Are the elements of this collection the same (and in the same order) as those of that?

#### def scan[B >: A](z: B)(op: (B, B) => B): LazyList[B]

Computes a prefix scan of the elements of the collection.

#### def scanLeft[B](z: B)(op: (B, A) => B): LazyList[B]

Produces a lazy list containing cumulative results of applying the operator going left to right, including the initial value.

#### def scanRight[B](z: B)(op: (A, B) => B): LazyList[B]

Produces a collection containing cumulative results of applying the operator going right to left.

#### def search[B >: A](elem: B, from: Int, to: Int)(implicit ord: Ordering[B]): SearchResult

Search within an interval in this sorted sequence for a specific element.

#### def search[B >: A](elem: B)(implicit ord: Ordering[B]): SearchResult

Search this sorted sequence for a specific element.

#### def segmentLength(p: (A) => Boolean, from: Int): Int

Computes the length of the longest segment that starts from some index and whose elements all satisfy some predicate.

#### final def segmentLength(p: (A) => Boolean): Int

Computes the length of the longest segment that starts from the first element and whose elements all satisfy some predicate.

#### final def size: Int

The size of this sequence.

#### final def sizeCompare(that: collection.Iterable[_]): Int

Compares the size of this sequence to the size of another Iterable.

#### final def sizeCompare(otherSize: Int): Int

Compares the size of this sequence to a test value.

#### final def sizeIs: SizeCompareOps

Returns a value class containing operations for comparing the size of this iterable collection to a test value.

#### def slice(from: Int, until: Int): LazyList[A]

Selects an interval of elements.

#### def sliding(size: Int, step: Int): Iterator[LazyList[A]]

Groups elements in fixed size blocks by passing a "sliding window" over them (as opposed to partitioning them, as is done in grouped.)

#### def sliding(size: Int): Iterator[LazyList[A]]

Groups elements in fixed size blocks by passing a "sliding window" over them (as opposed to partitioning them, as is done in grouped.)

#### def sortBy[B](f: (A) => B)(implicit ord: Ordering[B]): LazyList[A]

Sorts this sequence according to the Ordering which results from transforming an implicitly given Ordering with a transformation function.

#### def sortWith(lt: (A, A) => Boolean): LazyList[A]

Sorts this sequence according to a comparison function.

#### def sorted[B >: A](implicit ord: Ordering[B]): LazyList[A]

Sorts this sequence according to an Ordering.

#### def span(p: (A) => Boolean): (LazyList[A], LazyList[A])

Splits this iterable collection into a prefix/suffix pair according to a predicate.

#### def splitAt(n: Int): (LazyList[A], LazyList[A])

Splits this iterable collection into a prefix/suffix pair at a given position.

#### def startsWith[B >: A](that: IterableOnce[B], offset: Int = 0): Boolean

Tests whether this sequence contains the given sequence at a given index.

#### def stepper[S <: Stepper[_]](implicit shape: StepperShape[A, S]): S

Returns a scala.collection.Stepper for the elements of this collection.

#### def sum[B >: A](implicit num: math.Numeric[B]): B

Sums up the elements of this collection.

#### def tail: LazyList[A]

The rest of the collection without its first element.

#### def tails: Iterator[LazyList[A]]

Iterates over the tails of this sequence.

#### def take(n: Int): LazyList[A]

Selects the first n elements.

#### def takeRight(n: Int): LazyList[A]

Selects the last n elements.

#### def takeWhile(p: (A) => Boolean): LazyList[A]

Takes longest prefix of elements that satisfy a predicate.

#### def tapEach[U](f: (A) => U): LazyList[A]

Applies a side-effecting function to each element in this collection.

#### def to[C1](factory: Factory[A, C1]): C1

Given a collection factory factory, convert this collection to the appropriate representation for the current element type A.

#### def toArray[B >: A](implicit arg0: ClassTag[B]): Array[B]

Convert collection to array.

#### final def toBuffer[B >: A]: Buffer[B]

#### def toIndexedSeq: IndexedSeq[A]

#### final def toIterable: LazyList.this.type

#### def toList: List[A]

#### def toMap[K, V](implicit ev: <:<[A, (K, V)]): Map[K, V]

#### final def toSeq: LazyList.this.type

#### def toSet[B >: A]: Set[B]

#### def toString(): String

This method preserves laziness; elements are only evaluated individually as needed.

#### def toVector: Vector[A]

#### def transpose[B](implicit asIterable: (A) => collection.Iterable[B]): LazyList[LazyList[B]]

Transposes this lazy list of iterable collections into a lazy list of lazy lists.

#### def unapply(a: Int): Option[A]

Tries to extract a B from an A in a pattern matching expression.

#### def unlift: PartialFunction[Int, B]

Converts an optional function to a partial function.

#### def unzip[A1, A2](implicit asPair: (A) => (A1, A2)): (LazyList[A1], LazyList[A2])

Converts this lazy list of pairs into two collections of the first and second half of each pair.

#### def unzip3[A1, A2, A3](implicit asTriple: (A) => (A1, A2, A3)): (LazyList[A1], LazyList[A2], LazyList[A3])

Converts this lazy list of triples into three collections of the first, second, and third element of each triple.

#### def updated[B >: A](index: Int, elem: B): LazyList[B]

A copy of this lazy list with one single replaced element.

#### def view: SeqView[A]

A view over the elements of this collection.

#### def withFilter(p: (A) => Boolean): WithFilter[A, LazyList]

A collection.WithFilter which allows GC of the head of lazy list during processing.

#### def zip[B](that: IterableOnce[B]): LazyList[(A, B)]

Returns a lazy list formed from this lazy list and another iterable collection by combining corresponding elements in pairs.

#### def zipAll[A1 >: A, B](that: collection.Iterable[B], thisElem: A1, thatElem: B): LazyList[(A1, B)]

Returns a lazy list formed from this lazy list and another iterable collection by combining corresponding elements in pairs.

#### def zipWithIndex: LazyList[(A, Int)]

Zips this lazy list with its indices.
