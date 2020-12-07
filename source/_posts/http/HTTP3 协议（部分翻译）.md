---
title: HTTP/3 协议（部分翻译）
date: 2020-12-07 00:01:32
comments: true
categories:
 - HTTP
 - HTTP/3
tags: 
 - http
 - http3
---

## 文档信息

> Workgroup: QUIC
> Internet-Draft: draft-ietf-quic-http
> Published: 2 December 2020
> Intended Status: Standards Track
> Expires: 5 June 2021
> Author: M. Bishop, Ed.Akamai

原文：[Hypertext Transfer Protocol Version 3 (HTTP/3)](https://quicwg.org/base-drafts/draft-ietf-quic-http.html)


## 6.1. 双向流
所有客户端启动的双向流都用于HTTP请求和响应。双向流可确保响应可以轻松地与请求相关联。这些流称为请求流。

这意味着客户端的第一个请求发生在QUIC流0上，随后的请求发生在流4、8上，依此类推。为了允许这些流打开，HTTP / 3服务器应该为允许的流数量和初始流流控制窗口配置非零的最小值。为了避免不必要地限制并行性，一次应至少允许100个请求。

HTTP / 3不使用服务器启动的双向流，尽管扩展可以定义这些流的用法。除非协商了这样的扩展名，否则客户端必须将收到服务器启动的双向流视为H3_STREAM_CREATION_ERROR类型的连接错误（第8节）。

## 6.2. 单向流
任一方向上的单向流都用于多种用途。目的由流类型指示，该流类型在流的开头作为可变长度整数发送。跟随此整数的数据的格式和结构由流类型确定。

```cpp
Unidirectional Stream Header {
  Stream Type (i),
}
```
类型分两种：控制流（0x00）、推送流（0x01）

## 6.2.1. 控制流
**控制流由0x00的流类型指示**。如7.2节所定义，此流上的数据由HTTP / 3 帧组成。

**每一端都必须在连接开始时启动单个控制流，并将其设置帧作为该流的第一帧发送**。如果控制流的第一帧是任何其他帧类型，则必须将其视为H3_MISSING_SETTINGS类型的连接错误。每个对等方仅允许一个控制流；收到第二个流声称是控制流的消息，必须将其视为H3_STREAM_CREATION_ERROR类型的连接错误。**发送者不得关闭控制流，接收者不得请求发送者关闭控制流**。如果任何一个控制流在任何时候都关闭，则必须将其视为H3_CLOSED_CRITICAL_STREAM类型的连接错误。连接错误在第8节中描述 。

**使用一对单向流而不是单个双向流**。这允许任何一个对等方尽快发送数据。根据QUIC连接上是否启用了0-RTT，客户端或服务器可能能够在加密握手完成后首先发送流数据。

## 6.2.2. 推送流
服务器推送是HTTP / 2 中引入的可选功能，该功能允许服务器在发出请求之前启动响应。有关更多详细信息，请参见第4.4节。

**推送流由0x01的流类型指示**，后跟它履行的承诺的Push ID，编码为可变长度整数。该流中的其余数据由第7.2节中定义的HTTP / 3帧组成 ，并通过零个或多个临时HTTP响应以及第4.1节中定义的单个最终HTTP响应来实现承诺的服务器推送 。服务器推送和推送ID在4.4节中进行了描述 。

**只有服务器可以推送**；如果服务器接收到客户端发起的推送流，则必须将其视为类型为H3_STREAM_CREATION_ERROR的连接错误；参见 第8节。

```cpp
Push Stream Header {
  Stream Type (i) = 0x01,
  Push ID (i),
}
```
**每个推送ID只能在推送流标头中使用一次**。如果一个推送流头包含另一个推送流头中使用的推送ID，则客户端务必将此视为H3_ID_ERROR类型的连接错误；见第8节。

## 6.2.3. 保留的流类型
0x1f * N + 0x21保留N的非负整数值格式的流类型，以执行忽略未知类型的要求。**这些流没有语义，可以在需要应用程序层填充时发送**。它们也可以在当前没有数据传输的连接上发送。端点在接收时不得认为这些流具有任何意义。

流的有效负载和长度以发送实现选择的任何方式进行选择。发送保留的流类型时，实现可以干净地终止流或将其重置。重置流时，应使用H3_NO_ERROR错误代码或保留的错误代码（第8.1节）。

# 7. HTTP框架层
如第6节所述，HTTP帧承载在QUIC流上。HTTP / 3定义了三种流类型：**控制流，请求流和推送流**。本节介绍HTTP / 3 帧格式及其允许的流类型。有关概述，请参见表1。附录A.2提供了HTTP / 2 和HTTP / 3 帧之间的比较。

|Frame|	Control Stream| Request Stream | Push Stream|
|:-- |:-- |:-- |:-- |
|DATA	|No |	Yes	 |Yes |
|HEADERS	 |No	 |Yes |	Yes |
|CANCEL_PUSH |	Yes |	No |	No |
|SETTINGS	 |Yes (1) |	No |	No |
|PUSH_PROMISE |	No |	Yes |	No |
|GOAWAY |	Yes |	No |	No |
|MAX_PUSH_ID |	Yes |	No |	No |
|Reserved	 |Yes |	Yes	 |Yes |

## 7.1. 框架布局

```cpp
HTTP/3 Frame Format {
  Type (i),
  Length (i),
  Frame Payload (..),
}
```

框架包括以下字段：

- 类型：标识帧类型的长度可变的整数。
- 长度：一个可变长度的整数，以字节为单位描述帧有效负载的长度。
- 框架有效载荷：有效载荷，其语义由“类型”字段确定。

每个帧的有效载荷必须准确地包含其描述中标识的字段。在所标识的字段之后包含附加字节的帧有效载荷，或在所标识的字段结束之前终止的帧有效载荷必须被视为类型为H3_FRAME_ERROR的连接错误；参见第8节。

当流完全终止时，如果流的最后一帧被截断，则必须将其视为类型为H3_FRAME_ERROR的连接错误；参见 第8节。**突然终止的流可以在帧中的任何点重置。**

## 7.2.1. DATA
**DATA帧（type= 0x0）传达与HTTP请求或响应有效内容主体相关联的任意可变长度的字节序列。数据帧必须与HTTP请求或响应相关联。** 如果在控制流上接收到DATA帧，则接收者必须以H3_FRAME_UNEXPECTED类型的连接错误做出响应；见第8节。

```cpp
DATA Frame {
  Type (i) = 0x0,
  Length (i),
  Data (..),
}
```

## 7.2.2. HEADERS
**HEADERS 	帧（类型= 0x1）用于承载使用QPACK编码的HTTP字段部分。** 有关更多详细信息，请参见[ QPACK ]。

```cpp
HEADERS Frame {
  Type (i) = 0x1,
  Length (i),
  Encoded Field Section (..),
}
```

**HEADERS 帧只能根据请求或推送流发送**。如果在控制流上接收到HEADERS帧，则接收者务必以H3_FRAME_UNEXPECTED类型的连接错误（第8节）做出响应。

## 7.2.3. CANCEL_PUSH
**CANCEL_PUSH帧（type= 0x3）用于在接收到推送流之前请求取消服务器推送。** CANCEL_PUSH帧通过推送ID（请参阅第4.4节）标识服务器推送，该服务器ID被编码为可变长度整数。

**当客户端发送CANCEL_PUSH时，表明它不希望接收承诺的资源。** 服务器应该中止发送资源，但是机制取决于相应的推送流的状态。如果服务器尚未创建推送流，则不会创建推送流。如果推流是开放的，则服务器应该突然终止该流。如果推送流已经结束，则服务器可能仍然会突然终止流，或者可能不采取任何措施。

服务器发送CANCEL_PUSH来指示它将不履行先前发送的承诺。除非它已经收到并处理了承诺的响应，否则客户不能期望相应的承诺得到实现。**无论是否已打开推送流，服务器都应在确定将无法兑现承诺时发送CANCEL_PUSH帧。** 如果已经打开流，则服务器可以中止发送错误码为H3_REQUEST_CANCELLED的流。

发送CANCEL_PUSH帧对现有推送流的状态没有直接影响。当客户端已经收到相应的推送流时，客户端不应该发送CANCEL_PUSH帧。客户端发送了CANCEL_PUSH帧后，推送流可能会到达，因为服务器可能未处理CANCEL_PUSH。客户端应该中止读取错误码为H3_REQUEST_CANCELLED的流。

```cpp
CANCEL_PUSH Frame {
  Type (i) = 0x3,
  Length (i),
  Push ID (..),
}
```

在控制流上发送CANCEL_PUSH帧。在控制流以外的流上接收CANCEL_PUSH帧必须被视为H3_FRAME_UNEXPECTED类型的连接错误。

**CANCEL_PUSH帧携带编码为可变长度整数的Push ID。** 推送ID标识正在取消的服务器推送；参见 第4.4节。如果收到的CANCEL_PUSH帧引用的推送ID大于当前连接允许的推送ID，则必须将其视为H3_ID_ERROR类型的连接错误。

如果客户端收到CANCEL_PUSH帧，则该帧可能标识由于重新排序而尚未被PUSH_PROMISE帧提及的Push ID。如果服务器收到尚未由PUSH_PROMISE帧提及的推送ID的CANCEL_PUSH帧，则必须将其视为H3_ID_ERROR类型的连接错误。

## 7.2.4. SETTINGS
**SETTINGS 帧（类型= 0x4）传达影响端点通信方式的配置参数，例如，对等方行为的偏好和约束。** 单独地，SETTINGS参数也可以称为“设置”；每个设置参数的标识符和值可以称为“设置标识符”和“设置值”。

**SETTINGS帧始终应用于整个HTTP / 3连接，而不是单个流。**  **每个对等方必须将SETTINGS帧作为每个控制流的第一帧发送（请参见6.2.1节），并且不得随后发送。** 如果端点在控制流上收到第二个SETTINGS帧，则端点必须以H3_FRAME_UNEXPECTED类型的连接错误做出响应。

**SETTINGS帧不得在控制流以外的任何其他流上发送。** 如果一个端点在另一个流上接收到一个SETTINGS帧，则该端点必须以H3_FRAME_UNEXPECTED类型的连接错误响应。

**未协商SETTINGS参数；它们描述了接收对等方可以使用的发送对等方的特征。** 但是，可以通过使用SETTINGS来暗示协商-每个对等方都使用SETTINGS来宣传一组支持的值。设置的定义将描述每个对等方如何组合这两个集合以得出将使用哪个选择的结论。SETTINGS未提供识别选择何时生效的机制。

每个对等体可以通告相同参数的不同值。例如，客户端可能愿意占用很大的响应字段部分，而服务器则对请求大小更为谨慎。

**相同的设置标识符不得在SETTINGS帧中出现多次。** 接收者可以将重复的设置标识符的存在视为H3_SETTINGS_ERROR类型的连接错误。

**SETTINGS帧的有效负载包含零个或多个参数。每个参数都包含一个设置标识符和一个值，均被编码为QUIC可变长度整数。**

```cpp
Setting {
  Identifier (i),
  Value (i),
}

SETTINGS Frame {
  Type (i) = 0x4,
  Length (i),
  Setting (..) ...,
}
```
一个实现必须忽略它不理解的任何SETTINGS标识符的内容。

### 7.2.4.1. 已定义的 SETTINGS 参数
HTTP / 3中定义了以下设置： **SETTINGS_MAX_FIELD_SECTION_SIZE（0x6）： 默认值为无限制。** 有关用法，请参见第4.1.1.3节。 保留N的非负整数值的格式0x1f * N + 0x21的设置标识符，以保留忽略未知标识符的要求。此类设置没有定义的含义。端点应该在其SETTINGS帧中至少包含一个这样的设置。端点在接收时不得认为此类设置具有任何意义。 因为该设置没有定义的含义，所以该设置的值可以是实现选择的任何值。 在没有相应的HTTP / 3设置的HTTP / 2中使用的设置标识符也已保留（第11.2.2节）。不得发送这些设置，并且必须将其收据视为H3_SETTINGS_ERROR类型的连接错误。 可以通过HTTP / 3的扩展定义其他设置。有关更多详细信息，请参见第9节。

### 7.2.4.2. 初始化
HTTP实现不得发送基于其对同级设置当前的了解而无效的帧或请求。

所有设置均以初始值开始。每个端点应该使用这些初始值在对等方的SETTINGS帧到达之前发送消息，因为携带设置的数据包可能会丢失或延迟。当“设置”框到达时，所有设置都会更改为其新值。

这样就无需在发送消息之前等待SETTINGS帧。端点在发送SETTINGS帧之前不得要求从对等方接收任何数据；传输准备好发送数据后，必须立即发送设置。

对于服务器，每个客户端设置的初始值为默认值。

对于使用1-RTT QUIC连接的客户端，每个服务器设置的初始值为默认值。 1-RTT密钥将始终在QUIC处理包含SETTINGS的数据包之前变得可用，即使服务器立即发送SETTINGS。客户端不应在发送请求之前无限期地等待SETTINGS到达，而是应该处理接收到的数据报，以增加在发送第一个请求之前处理SETTINGS的可能性。

使用0-RTT QUIC连接时，每个服务器设置的初始值是上一个会话中使用的值。客户端应该在提供恢复信息的HTTP / 3连接中存储服务器提供的设置，但是在某些情况下（例如，如果在SETTINGS帧之前收到了会话票证）可以选择不存储设置。尝试0-RTT时，客户端必须遵守存储的设置-或默认值（如果未存储任何值）。服务器提供新设置后，客户端必须遵守这些值。

服务器可以记住其公布的设置，或者在票证中存储值的完整性受保护的副本，并在接受0-RTT数据时恢复信息。服务器使用HTTP / 3设置值来确定是否接受0-RTT数据。如果服务器无法确定客户端记住的设置与其当前设置兼容，则它不得接受0-RTT数据。如果遵循这些设置的客户端不会违反服务器的当前设置，则记住的设置是兼容的。

服务器可以接受0-RTT，随后在其SETTINGS帧中提供不同的设置。如果服务器接受0-RTT数据，则其“设置”框架不得减少任何限制或更改客户端使用其0-RTT数据可能违反的任何值。服务器必须包括与其默认值不同的所有设置。如果服务器接受0-RTT，然后发送与先前指定的设置不兼容的设置，则必须将其视为H3_SETTINGS_ERROR类型的连接错误。如果服务器接受0-RTT，但随后发送SETTINGS帧，则该帧忽略了客户端可以理解的设置值（除保留的设置标识符之外），该值先前已指定为非默认值，因此必须将此视为连接错误输入H3_SETTINGS_ERROR。

## 7.2.5. PUSH_PROMISE
**PUSH_PROMISE帧（类型= 0x5）用于在请求流上从服务器到客户端传送承诺的请求标头字段部分**，如HTTP / 2中所示。

```cpp
PUSH_PROMISE Frame {
  Type (i) = 0x5,
  Length (i),
  Push ID (i),
  Encoded Field Section (..),
}
```
有效负载包括：

**推送ID：标识服务器推送操作的可变长度整数。** 在推送流标题（第4.4节）和CANCEL_PUSH帧（第7.2.3节）中使用了推送ID。
**编码字段部分：承诺响应的QPACK编码的请求标头字段。** 有关更多详细信息，请参见[QPACK]。

**服务器不得使用大于客户端在MAX_PUSH_ID帧中提供的推送ID（第7.2.7节）。客户端必须将收到的PUSH_PROMISE帧包含的Push ID大于客户端通告为H3_ID_ERROR的连接错误。**

**服务器可以在多个PUSH_PROMISE帧中使用相同的Push ID。** 如果是这样，则解压缩的请求头集必须以相同的顺序包含相同的字段，并且每个字段中的名称和值都必须完全匹配。客户端应该比较请求头部分中多次承诺的资源。如果客户端收到已经承诺的推送ID并检测到不匹配，则它必须以H3_GENERAL_PROTOCOL_ERROR类型的连接错误响应。如果解压缩的字段部分完全匹配，则客户端应将推送的内容与在其上接收到PUSH_PROMISE帧的每个流相关联。

允许重复引用相同的Push ID主要是为了减少由并发请求引起的重复。**服务器应避免长时间重用Push ID。** *客户端很可能会消耗服务器的推送响应，而不会保留它们以供日后重用。* 看到看到PUSH_PROMISE帧使用已使用和丢弃的Push ID的客户端将被迫忽略承诺。

如果在控制流上接收到一个PUSH_PROMISE帧，则客户端必须以H3_FRAME_UNEXPECTED类型的连接错误响应；见第8节。

**客户端不得发送PUSH_PROMISE帧。** 服务器必须将收到的PUSH_PROMISE帧视为H3_FRAME_UNEXPECTED类型的连接错误；见第8节。

有关整个服务器推送机制的说明，请参见第4.4节。

## 7.2.6. GOAWAY
**GOAWAY帧（类型= 0x7）用于通过任一端点启动HTTP / 3连接的正常关闭。** GOAWAY允许端点停止接受新的请求或推送，同时仍然完成对先前接收到的请求和推送的处理。这将启用管理操作，例如**服务器维护**。 GOAWAY本身不会关闭连接。

```cpp
GOAWAY Frame {
  Type (i) = 0x7,
  Length (i),
  Stream ID/Push ID (..),
}
```
**GOAWAY帧始终在控制流上发送。** **在服务器到客户端的方向上，它为编码为可变长度整数的客户端启动的双向流携带QUIC流ID。** 客户端必须将包含任何其他类型的流ID的GOAWAY帧的接收视为H3_ID_ERROR类型的连接错误。 **在客户端到服务器的方向上，GOAWAY帧携带一个Push ID编码为可变长度整数。** GOAWAY帧适用于整个连接，而不是特定的流。客户端必须将控制流以外的流上的GOAWAY帧视为H3_FRAME_UNEXPECTED类型的连接错误；见第8节。 有关使用GOAWAY框架的更多信息，请参见5.2节。

## 7.2.7. MAX_PUSH_ID
**客户端使用MAX_PUSH_ID帧（类型= 0xd）来控制服务器可以发起的服务器推送次数。** 这将设置服务器可以在PUSH_PROMISE和CANCEL_PUSH帧中使用的Push ID的最大值。因此，除了QUIC传输保持的限制之外，这还限制了服务器可以启动的推送流的数量。

**MAX_PUSH_ID帧始终在控制流上发送。** 在任何其他流上接收到MAX_PUSH_ID帧，必须将其视为H3_FRAME_UNEXPECTED类型的连接错误。

**服务器不得发送MAX_PUSH_ID帧。** 客户端必须将收到的MAX_PUSH_ID帧视为H3_FRAME_UNEXPECTED类型的连接错误。

**创建HTTP / 3连接时，未设置最大Push ID，这意味着服务器无法推送，直到收到MAX_PUSH_ID帧为止。** 希望管理承诺的服务器推送次数的客户端可以通过在服务器完成或取消服务器推送时发送MAX_PUSH_ID帧来增加最大Push ID。

```cpp
MAX_PUSH_ID Frame {
  Type (i) = 0xd,
  Length (i),
  Push ID (i),
}
```
**MAX_PUSH_ID帧携带一个可变长度的整数，该整数标识服务器可以使用的Push ID的最大值；** 请参阅第4.4节。 MAX_PUSH_ID框架无法减少最大Push ID；收到的MAX_PUSH_ID帧的值小于**先前接收到的帧的值**，必须将其视为H3_ID_ERROR类型的连接错误。

## 7.2.8. 保留帧类型
**保留N的非负整数值的格式为0x1f * N + 0x21的帧类型，以执行忽略未知类型的要求（第9节）。这些帧没有语义，可以在允许发送帧的任何流上发送。这使得它们可以用于应用程序层填充。** 端点在接收时不得将这些框架视为任何意义。 帧的有效负载和长度以实现方式选择的任何方式进行选择。 在HTTP / 2中使用的没有相应HTTP / 3帧的帧类型也已保留（第11.2.1节）。不得发送这些帧类型，并且必须将其接收视为H3_FRAME_UNEXPECTED类型的连接错误。

# 8. 错误处理
## 8.1. HTTP / 3 错误代码
定义以下错误代码供突然终止流，中止流读取或立即关闭HTTP / 3连接时使用。
<div class="table-wrapper"><table>
<thead>
<tr>
<th align="left">代码</th>
<th align="left">描述</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">H3_NO_ERROR（0x100）</td>
<td align="left">没错 当需要关闭连接或流但没有错误要发信号时使用此方法。</td>
</tr>
<tr>
<td align="left">H3_GENERAL_PROTOCOL_ERROR（0x101）</td>
<td align="left">对等方违反了协议要求，其方式与更具体的错误代码不匹配，或者端点拒绝使用更具体的错误代码。</td>
</tr>
<tr>
<td align="left">H3_INTERNAL_ERROR（0x102）</td>
<td align="left">HTTP堆栈中发生内部错误。</td>
</tr>
<tr>
<td align="left">H3_STREAM_CREATION_ERROR（0x103）</td>
<td align="left">端点检测到其对等方创建了它将不接受的流。</td>
</tr>
<tr>
<td align="left">H3_CLOSED_CRITICAL_STREAM（0x104）</td>
<td align="left">HTTP / 3连接所需的流已关闭或重置。</td>
</tr>
<tr>
<td align="left">H3_FRAME_UNEXPECTED（0x105）</td>
<td align="left">接收到当前状态或当前流中不允许的帧。</td>
</tr>
<tr>
<td align="left">H3_FRAME_ERROR（0x106）</td>
<td align="left">收到不符合布局要求或尺寸无效的框架。</td>
</tr>
<tr>
<td align="left">H3_EXCESSIVE_LOAD（0x107）</td>
<td align="left">端点检测到其对等方正在表现出可能产生过多负载的行为。</td>
</tr>
<tr>
<td align="left">H3_ID_ERROR（0x108）</td>
<td align="left">Stream ID或Push ID使用不正确，例如超过限制，减少限制或被重用。</td>
</tr>
<tr>
<td align="left">H3_SETTINGS_ERROR（0x109）</td>
<td align="left">端点在SETTINGS帧的有效载荷中检测到错误。</td>
</tr>
<tr>
<td align="left">H3_MISSING_SETTINGS（0x10a）</td>
<td align="left">在控制流的开始未接收到SETTINGS帧。</td>
</tr>
<tr>
<td align="left">H3_REQUEST_REJECTED（0x10b）</td>
<td align="left">服务器拒绝了请求而不执行任何应用程序处理。</td>
</tr>
<tr>
<td align="left">H3_REQUEST_CANCELLED（0x10c）</td>
<td align="left">该请求或其响应（包括推送响应）被取消。</td>
</tr>
<tr>
<td align="left">H3_REQUEST_INCOMPLETE（0x10d）</td>
<td align="left">客户端的流终止，但不包含完整的请求。</td>
</tr>
<tr>
<td align="left">H3_CONNECT_ERROR（0x10f）</td>
<td align="left">响应于CONNECT请求而建立的TCP连接被重置或异常关闭。</td>
</tr>
<tr>
<td align="left">H3_VERSION_FALLBACK（0x110）</td>
<td align="left">无法通过HTTP / 3提供请求的操作。对等方应通过HTTP / 1.1重试。</td>
</tr>
</tbody>
</table>
</div>
0x1f * N + 0x21保留N的非负整数值格式的错误代码，以执行将未知错误代码视为等同于H3_NO_ERROR的要求（第9节）。当实现应该发送H3_NO_ERROR时，应从该空间中选择一个错误代码。

# 9. HTTP / 3的扩展
HTTP / 3允许扩展协议。在本节描述的限制范围内，协议扩展可用于提供其他服务或更改协议的任何方面。扩展仅在单个HTTP / 3连接的范围内有效。

这适用于本文档中定义的协议元素。这不会影响用于扩展HTTP的现有选项，例如定义新方法，状态代码或字段。

允许扩展使用新的帧类型（第7.2节），新的设置（第7.2.4.1节），新的错误代码（第8节）或新的单向流类型（第6.2节）。建立用于管理这些扩展点的注册表：帧类型（第11.2.1节），设置（第11.2.2节），错误代码（第11.2.3节）和流类型（第11.2.4节）。

实施必须忽略所有可扩展协议元素中未知或不受支持的值。实现必须丢弃具有未知或不支持类型的帧和单向流。这意味着这些扩展点中的任何一个都可以被扩展安全地使用，而无需事先安排或协商。但是，在要求已知帧类型位于特定位置的情况下，例如将SETTINGS帧作为控制流的第一帧（请参阅第6.2.1节），未知帧类型不能满足该要求，应予以处理作为错误。

可能会更改现有协议组件语义的扩展必须在使用前进行协商。例如，在对等端给出肯定的信号之前，不能使用更改HEADERS帧的布局的扩展名。协调这种修改后的布局何时生效可能会很复杂。这样，为现有协议元素的新定义分配新的标识符可能会更有效。

本文档未强制规定使用扩展的具体方法，但请注意，可以将设置（第7.2.4.1节）用于此目的。如果两个对等方都设置了一个表示愿意使用该扩展名的值，则可以使用该扩展名。如果将设置用于扩展协商，则必须以以下方式定义默认值：如果省略设置，则禁用扩展。
