---
title: Akka Actors入门案例解析
date: 2020-10-29 13:03:43
comments: true
categories:
 - Scala
 - Akka
tags: 
 - akka
 - actors
 - scala
---

# 1. HelloWorld

```scala
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.scaladsl.LoggerOps
import akka.actor.typed.{ActorRef, ActorSystem, Behavior}

object HelloWorld {

  /**
   * 某人打招呼
   *
   * @param whom    谁（某人）
   * @param replyTo 消息的回复者
   */
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])

  /**
   * 确认已打过招呼
   *
   * @param whom 谁
   * @param from 来自哪里（发送者）
   */
  final case class Greeted(whom: String, from: ActorRef[Greet])

  /**
   * 默认执行方法，响应 Greet
   *
   * @return
   */
  def apply(): Behavior[Greet] = Behaviors.receive {
    (context, message) =>
      // 发送消息
      context.log.info("发送消息 => Hello {}!", message.whom)
      // 回复已经收到消息
      message.replyTo ! Greeted(message.whom , context.self)
      // 行为重用
      Behaviors.same
  }
}

object HelloWorldBot {

  /**
   * 响应 Greeted
   *
   * @param max 总问候次数
   * @return
   */
  def apply(max: Int): Behavior[HelloWorld.Greeted] = {
    bot(0, max)
  }

  private def bot(greetingCounter: Int, max: Int): Behavior[HelloWorld.Greeted] =
    Behaviors.receive { (context, message) =>
      val n = greetingCounter + 1
      // 收到消息
      context.log.info2("收到消息 => Greeting {} for {}", n, message.whom)
      if (n == max) {
        // 达到总数时，停止。
        Behaviors.stopped
      } else {
        message.from ! HelloWorld.Greet(message.whom+ " 回复 ", context.self)
        bot(n, max)
      }
    }
}


object HelloWorldMain {

  // 入口
  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      // 从给定的行为并给定名称创建子Actor
      val greeter = context.spawn(HelloWorld(), "greeter")
      // 创建简单的参与者
      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)
        greeter ! HelloWorld.Greet(message.name, replyTo)
        Behaviors.same
      }
    }

}

object Main extends App {

  val system: ActorSystem[HelloWorldMain.SayHello] =
    ActorSystem(HelloWorldMain(), "hello")

  system ! HelloWorldMain.SayHello("World")
  system ! HelloWorldMain.SayHello("Akka")
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201029105039793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

---

## 响应打招呼信息
```scala
object HelloWorld {

  /**
   * 某人打招呼
   *
   * @param whom    谁（某人）
   * @param replyTo 消息的回复者
   */
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])

  /**
   * 确认已打过招呼
   *
   * @param whom 谁
   * @param from 来自哪里（发送者）
   */
  final case class Greeted(whom: String, from: ActorRef[Greet])

  /**
   * 默认执行方法，响应 Greet
   *
   * @return
   */
  def apply(): Behavior[Greet] = Behaviors.receive {
    (context, message) =>
      // 发送消息
      context.log.info("发送消息 => Hello {}!", message.whom)
      // 回复已经收到消息
      message.replyTo ! Greeted(message.whom , context.self)
      // 行为重用
      Behaviors.same
  }
}
```
定义了两个消息类 Greet、Greeted


### Behavior[T]

> The behavior of an actor defines how it reacts to the messages that it receives. The message may either be of the type that the Actor declares and which is part of the ActorRef signature, or it may be a system Signal that expresses a lifecycle event of either this actor or one of its child actors.
> Behaviors can be formulated in a number of different ways, either by using the DSLs in akka.actor.typed.scaladsl.Behaviors and akka.actor.typed.javadsl.Behaviors or extending the abstract ExtensibleBehavior class.
> Closing over ActorContext makes a Behavior immobile: it cannot be moved to another context and executed there, and therefore it cannot be replicated or forked either.
> This base class is not meant to be extended by user code. If you do so, you may lose binary compatibility.
> Not for user extension.
> \
> actor的行为定义了它对收到的消息的反应。 该消息可以是Actor声明的类型，并且是ActorRef签名的一部分，也可以是表示该Actor或其子Actor的生命周期事件的系统Signal。
> 可以通过使用akka.actor.typed.scaladsl.Behaviors和akka.actor.typed.javadsl.Behaviors中的DSL或扩展抽象的ExtensibleBehavior类来以多种不同方式来表示行为。
> 在ActorContext上关闭将使行为不可移动：无法将其移动到另一个上下文并在该上下文中执行，因此也无法复制或分支它。
> 此基类并不打算由用户代码扩展。 如果这样做，则可能会失去二进制兼容性。
> 不用于用户扩展。

消息订阅

### Behaviors.receive
> Construct an actor behavior that can react to both incoming messages and lifecycle signals. After spawning this actor from another actor (or as the guardian of an ActorSystem) it will be executed within an ActorContext that allows access to the system, spawning and watching other actors, etc.
> Compared to using AbstractBehavior this factory is a more functional style of defining the Behavior. Processing the next message results in a new behavior that can potentially be different from this one. State is maintained by returning a new behavior that holds the new immutable state.
>  \
> 构造一个既可以对传入消息又可以对生命周期信号做出反应的参与者行为。 从另一个角色（或作为ActorSystem的守护者）派生此actor之后，它将在ActorContext中执行，该ActorContext允许访问系统，派生和观看其他actor等。
> 与使用AbstractBehavior相比，该工厂是定义行为的更具功能性的样式。 处理下一条消息会导致一种新行为，该行为可能与此行为有所不同。 通过返回拥有新的不可变状态的新行为来维护状态。

构造一个既可以对传入消息又可以对生命周期信号做出反应的参与者。

### def self: ActorRef[T]

> The identity of this Actor, bound to the lifecycle of this Actor instance. An Actor with the same name that lives before or after this instance will have a different ActorRef.
> This field is thread-safe and can be called from other threads than the ordinary actor message processing thread, such as Future callbacks.
> \
> 此Actor的身份，绑定到此Actor实例的生命周期。 在此实例之前或之后存在的同名Actor将具有不同的ActorRef。
> 该字段是线程安全的，可以从普通actor消息处理线程以外的其他线程中调用，例如Future回调。

### def !(msg: T): Unit = ref.tell(msg)
> Send a message to the Actor referenced by this ActorRef using *at-most-once* messaging semantics.
> \
> 使用“至多一次”消息传递语义将消息发送到此ActorRef引用的Actor。

### Behaviors.same

> Return this behavior from message processing in order to advise the system to reuse the previous behavior. This is provided in order to avoid the allocation overhead of recreating the current behavior where that is not necessary.
> \
> 从消息处理中返回此行为，以建议系统重用以前的行为。 提供此功能是为了避免不必要的重新创建当前行为的分配开销。

---

## 响应回复消息
```scala
object HelloWorldBot {

  /**
   * 响应 Greeted
   *
   * @param max 总问候次数
   * @return
   */
  def apply(max: Int): Behavior[HelloWorld.Greeted] = {
    bot(0, max)
  }

  private def bot(greetingCounter: Int, max: Int): Behavior[HelloWorld.Greeted] =
    Behaviors.receive { (context, message) =>
      val n = greetingCounter + 1
      // 收到消息
      context.log.info2("收到消息 => Greeting {} for {}", n, message.whom)
      if (n == max) {
        // 达到总数时，停止。
        Behaviors.stopped
      } else {
        message.from ! HelloWorld.Greet(message.whom+ " 回复 ", context.self)
        bot(n, max)
      }
    }
}
```

### Behaviors.stopped

> Return this behavior from message processing to signal that this actor shall terminate voluntarily. If this actor has created child actors then these will be stopped as part of the shutdown procedure.
> The PostStop signal that results from stopping this actor will be passed to the current behavior. All other messages and signals will effectively be ignored.
> \
> 从消息处理中返回此行为，以表示该参与者应自愿终止。 如果此角色已创建子角色，则这些子角色将作为关闭过程的一部分而停止。
> 停止此参与者所产生的PostStop信号将传递给当前行为。 所有其他消息和信号将被有效忽略。

---

## 入口
```scala
object HelloWorldMain {

  // 入口
  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      // 从给定的行为并给定名称创建子Actor
      val greeter = context.spawn(HelloWorld(), "greeter")
      // 创建简单的参与者
      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)
        greeter ! HelloWorld.Greet(message.name, replyTo)
        Behaviors.same
      }
    }
    
}
```

### def spawn[U](behavior: Behavior[U], name: String, props: Props = Props.empty): ActorRef[U]

> Create a child Actor from the given Behavior and with the given name.
> *Warning*: This method is not thread-safe and must not be accessed from threads other than the ordinary actor message processing thread, such as Future callbacks.
> \
> 从给定的行为并给定名称创建子Actor。
> *警告*：此方法不是线程安全的，并且不能从普通actor消息处理线程以外的线程（例如Future回调）中访问。

中转

### Behaviors.receiveMessage

> Simplified version of Behaviors.Receive with only a single argument - the message to be handled. Useful for when the context is already accessible by other means, like being wrapped in an setup or similar.
> Construct an actor behavior that can react to both incoming messages and lifecycle signals. After spawning this actor from another actor (or as the guardian of an ActorSystem) it will be executed within an ActorContext that allows access to the system, spawning and watching other actors, etc.
> Compared to using AbstractBehavior this factory is a more functional style of defining the Behavior. Processing the next message results in a new behavior that can potentially be different from this one. State is maintained by returning a new behavior that holds the new immutable state.
> \
> 行为的简化版本。仅使用一个参数即可接收-要处理的消息。 当上下文已经可以通过其他方式访问时有用，例如包装在设置中或类似内容中。
> 构造一个既可以对传入消息又可以对生命周期信号做出反应的参与者行为。 从另一个角色（或作为ActorSystem的守护者）派生此actor之后，它将在ActorContext中执行，该ActorContext允许访问系统，派生和观看其他actor等。
> 与使用AbstractBehavior相比，该工厂是定义行为的更具功能性的样式。 处理下一条消息会导致一种新行为，该行为可能与此行为有所不同。 通过返回拥有新的不可变状态的新行为来维护状态。

行为的简化版本，少了 context

---

## 运行
```scala
object Main extends App {

  val system: ActorSystem[HelloWorldMain.SayHello] =
    ActorSystem(HelloWorldMain(), "hello")

  system ! HelloWorldMain.SayHello("World")
  system ! HelloWorldMain.SayHello("Akka")
}
```

---

## 运行结果
```scala
2020-10-29 11:28:12.458  INFO  --- [lt-dispatcher-7] HelloWorld$                              : 发送消息 => Hello World! 
2020-10-29 11:28:12.459  INFO  --- [lt-dispatcher-7] HelloWorld$                              : 发送消息 => Hello Akka! 
2020-10-29 11:28:12.460  INFO  --- [lt-dispatcher-7] HelloWorldBot$                           : 收到消息 => Greeting 1 for Akka 
2020-10-29 11:28:12.460  INFO  --- [lt-dispatcher-7] HelloWorld$                              : 发送消息 => Hello Akka 回复 ! 
2020-10-29 11:28:12.460  INFO  --- [lt-dispatcher-7] HelloWorldBot$                           : 收到消息 => Greeting 2 for Akka 回复  
2020-10-29 11:28:12.460  INFO  --- [lt-dispatcher-7] HelloWorld$                              : 发送消息 => Hello Akka 回复  回复 ! 
2020-10-29 11:28:12.460  INFO  --- [lt-dispatcher-7] HelloWorldBot$                           : 收到消息 => Greeting 3 for Akka 回复  回复  
2020-10-29 11:28:12.460  INFO  --- [lt-dispatcher-6] HelloWorldBot$                           : 收到消息 => Greeting 1 for World 
2020-10-29 11:28:12.462  INFO  --- [lt-dispatcher-6] HelloWorld$                              : 发送消息 => Hello World 回复 ! 
2020-10-29 11:28:12.463  INFO  --- [lt-dispatcher-6] HelloWorldBot$                           : 收到消息 => Greeting 2 for World 回复  
2020-10-29 11:28:12.463  INFO  --- [lt-dispatcher-6] HelloWorld$                              : 发送消息 => Hello World 回复  回复 ! 
2020-10-29 11:28:12.464  INFO  --- [lt-dispatcher-3] HelloWorldBot$                           : 收到消息 => Greeting 3 for World 回复  回复  
```

# 2. ChatRoom 

```scala
import java.net.URLEncoder
import java.nio.charset.StandardCharsets

import akka.NotUsed
import akka.actor.typed.scaladsl.{Behaviors, LoggerOps}
import akka.actor.typed.{ActorRef, ActorSystem, Behavior, Terminated}

/**
 * Functional Style
 *
 * @author https://doc.akka.io/docs/akka/current/typed/actors.html#first-example
 * @version v1 2020/10/29 10:14
 */


// 定义聊天室
sealed trait RoomCommand

// 获取会话
final case class GetSession(screenName: String, replyTo: ActorRef[SessionEvent]) extends RoomCommand

// 定义会话事件
sealed trait SessionEvent

// 允许会话
final case class SessionGranted(handle: ActorRef[PostMessage]) extends SessionEvent

// 拒绝会话
final case class SessionDenied(reason: String) extends SessionEvent

// 消息已发送
final case class MessagePosted(screenName: String, message: String) extends SessionEvent

// 定义会话命令
trait SessionCommand

// 消息
final case class PostMessage(message: String) extends SessionCommand

// 通知客户端
private final case class NotifyClient(message: MessagePosted) extends SessionCommand

/**
 * 聊天室
 */
object ChatRoom {

  // 发布会话消息
  private final case class PublishSessionMessage(screenName: String, message: String) extends RoomCommand

  // 默认空会话
  def apply(): Behavior[RoomCommand] = chatRoom(List.empty)

  private def chatRoom(sessions: List[ActorRef[SessionCommand]]): Behavior[RoomCommand] =
    Behaviors.receive { (context, message) =>
      message match {
        // 处理获取会话请求
        case GetSession(screenName, client) =>
          // create a child actor for further interaction with the client

          // 创建会话
          val ses = context.spawn(
            session(context.self, screenName, client),
            name = URLEncoder.encode(screenName, StandardCharsets.UTF_8.name))

          // 允许会话
          client ! SessionGranted(ses)

          // 得到会话，再次回到聊天室（发送消息）
          chatRoom(ses :: sessions)

        // 处理发布会话消息请求
        case PublishSessionMessage(screenName, message) =>
          println(s"\n聊天室 => 发送消息$message")
          val notification = NotifyClient(MessagePosted(screenName, message))
          // 向会话发送信息
          println("聊天室 => 告诉当前会话：已经发送了消息")
          sessions.foreach(_ ! notification)
          Behaviors.same
      }
    }

  /**
   * 创建会话
   *
   * @param room       聊天室房间（处理消息发布）
   * @param screenName 显示屏幕？
   * @param client     客户端（处理会话事件）
   * @return
   */
  private def session(
                       room: ActorRef[PublishSessionMessage],
                       screenName: String,
                       client: ActorRef[SessionEvent]): Behavior[SessionCommand] =
    Behaviors.receiveMessage {
      case PostMessage(message) =>
        println(s"\nsession收到客户端任务 => 客户端发送的消息：$message")
        // from client, publish to others via the room
        println(s"session => 告诉聊天室要发送这条消息（$message）。")
        room ! PublishSessionMessage(screenName, message)
        Behaviors.same
      case NotifyClient(message) =>
        println(s"\nsession收到聊天室任务 => NotifyClient，告诉客户端消息（$message）已发送。")
        // published from the room
        client ! message
        Behaviors.same
    }
}

/**
 * 客户端
 */
object Gabbler {

  def apply(): Behavior[SessionEvent] =
    Behaviors.setup { context =>
      Behaviors.receiveMessage {
        case SessionGranted(handle) =>
          println("\n客户端收到消息 => 会话已创建")
          handle ! PostMessage("Hello World!")
          println("客户端发送 => Hello World!")
          Behaviors.same
        case MessagePosted(screenName, message) =>
          println("\n客户端收到消息 => 消息已发送")
          context.log.info2("message has been posted by '{}': {}", screenName, message)
          Behaviors.stopped
      }
    }
}

object Main {
  def apply(): Behavior[NotUsed] = Behaviors.setup {
    context =>
      // 创建聊天室
      println("\n创建聊天室")
      val chatRoom = context.spawn(ChatRoom(), "chatroom")
      // 创建客户端
      println("创建客户端")
      val gabblerRef = context.spawn(Gabbler(), "gabbler")

      // 监听客户端
      context.watch(gabblerRef)

      // 客户端获取会话请求
      println("\n客户端获取会话请求")
      chatRoom ! GetSession("ol’ Gabbler", gabblerRef)

      // 异常处理
      Behaviors.receiveSignal {
        case (_, Terminated(_)) =>
          Behaviors.stopped
      }
  }

  def main(args: Array[String]): Unit = {
    ActorSystem(Main(), "ChatRoomDemo")
  }

}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020102912530827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dNaW5nWmhvdQ==,size_16,color_FFFFFF,t_70#pic_center)

---


### def watch[U](other: ActorRef[U]): Unit

> Register for Terminated notification once the Actor identified by the given ActorRef terminates. This message is also sent when the watched actor is on a node that has been removed from the cluster when using Akka Cluster.
watch is idempotent if it is not mixed with watchWith.
> It will fail with an IllegalStateException if the same subject was watched before using watchWith. To clear the termination message, unwatch first.
> *Warning*: This method is not thread-safe and must not be accessed from threads other than the ordinary actor message processing thread, such as Future callbacks.
> \
> 一旦给定ActorRef标识的Actor终止，请注册终止通知。 当使用Akka Cluster时，被监视的actor位于已从群集中删除的节点上时，也会发送此消息。
> 如果watchWatch不与watchWith混合使用，则手表是幂等的。
如果在使用watchWith之前观看了相同的主题，它将失败并抛出IllegalStateException。 要清除终止消息，请先取消监视。
> *警告*：此方法不是线程安全的，并且不能从普通actor消息处理线程以外的线程（例如Future回调）中访问。

### Behaviors.receiveSignal

> Construct an actor Behavior that can react to lifecycle signals only.
> \
> 构造一个仅对生命周期信号做出反应的行为者行为。


### Behaviors.setup

> setup is a factory for a behavior. Creation of the behavior instance is deferred until the actor is started, as opposed to Behaviors.receive that creates the behavior instance immediately before the actor is running. The factory function pass the ActorContext as parameter and that can for example be used for spawning child actors.
> setup is typically used as the outer most behavior when spawning an actor, but it can also be returned as the next behavior when processing a message or signal. In that case it will be started immediately after it is returned, i.e. next message will be processed by the started behavior.
> \
> 设置是行为的工厂。 行为实例的创建推迟到actor启动之前，与Behaviors.receive相反，后者在actor运行之前立即创建行为实例。 工厂函数将ActorContext作为参数传递，例如可以用于生成子actor。
> 在生成actor时，setup通常用作最外部的行为，但在处理消息或信号时，也可以将其作为下一个行为返回。 在这种情况下，它将在返回后立即启动，即，下一条消息将由启动的行为处理。

---

## 运行结果
```scala

创建聊天室
创建客户端

客户端获取会话请求

客户端收到消息 => 会话已创建
客户端发送 => Hello World!

session收到客户端任务 => 客户端发送的消息：Hello World!
session => 告诉聊天室要发送这条消息（Hello World!）。

聊天室 => 发送消息Hello World!
聊天室 => 告诉当前会话：已经发送了消息

session收到聊天室任务 => NotifyClient，告诉客户端消息（MessagePosted(ol’ Gabbler,Hello World!)）已发送。

客户端收到消息 => 消息已发送
2020-10-29 12:48:25.889  INFO  --- [lt-dispatcher-3] c.e.eam.service.device.akka.Gabbler$     : message has been posted by 'ol’ Gabbler': Hello World! 

Process finished with exit code 0
```
