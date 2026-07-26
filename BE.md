> __警告__ `不推荐`：仅实现过一次且不清楚是否正常工作，需要审查

NIP-BE
======

Nostr BLE 通信协议
---------------------------------

`draft` `unrecommended` `optional`

本 NIP 规定了 Nostr 应用如何使用 BLE（蓝牙低功耗）进行相互通信和同步。BLE 协议遵循客户端-服务器模式，因此本 NIP 以类似方式模拟了 WebSocket 结构，但根据其限制进行了一些调整。

## 设备广播
设备通过以下方式广播自身：
- 服务 UUID：`0000180f-0000-1000-8000-00805f9b34fb`
- 数据：ByteArray 格式的设备 UUID

## GATT 服务
设备暴露一个 Nordic UART 服务，具有以下特征：

1. 写入特征
   - UUID：`87654321-0000-1000-8000-00805f9b34fb`
   - 属性：写入

2. 读取特征
   - UUID：`12345678-0000-1000-8000-00805f9b34fb`
   - 属性：通知、读取

## 角色分配

当一个设备最初发现另一个设备在广播该服务时，它将读取服务的数据以获取设备 UUID，并与自身广播的设备 UUID 进行比较。在此通信中，ID 较高的设备将担任 GATT 服务器（中继）的角色，另一设备将被视为 GATT 客户端（客户端）并将继续建立连接。

对于其用途需要单一角色的设备，其设备 UUID 将始终为：

- GATT 服务器：`FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF`
- GATT 客户端：`00000000-0000-0000-0000-000000000000`

## 消息

所有消息将遵循 [NIP-01](/01.md) 消息结构。对于给定的消息，会对消息应用压缩流（DEFLATE）以生成字节数组。根据 BLE 版本，字节数组可能对于单条消息来说过大（BLE 4.2 中为 20-23 字节，BLE > 4.2 中为 256 字节）。在这种情况下，该字节数组将按照以下结构拆分为任意数量的批次：

```
[batch index (first 2 bytes)][batch n][is last batch (last byte)]
```
在接收到所有批次后，另一设备可以将其合并并解压。为确保可靠性，每次只读取/写入 1 条消息。MTU 可以事先协商。消息的最大大小为 64KB；更大的消息将被拒绝。

## 示例

此示例实现了将字节数组分割压缩为数据块的函数，以及将数据块合并解压以获得原始结果的函数：

```kotlin
fun splitInChunks(message: ByteArray): Array<ByteArray> {
   val chunkSize = 500 // define the chunk size
   var byteArray = compressByteArray(message)
   val numChunks = (byteArray.size + chunkSize - 1) / chunkSize // calculate the number of chunks
   var chunkIndex = 0
   val chunks = Array(numChunks) { ByteArray(0) }

   for (i in 0 until numChunks) {
         val start = i * chunkSize
         val end = minOf((i + 1) * chunkSize, byteArray.size)
         val chunk = byteArray.copyOfRange(start, end)

         // add chunk index to the first 2 bytes and last chunk flag to the last byte
         val chunkWithIndex = ByteArray(chunk.size + 2)
         chunkWithIndex[0] = chunkIndex.toByte() // chunk index
         chunk.copyInto(chunkWithIndex, 1)
         chunkWithIndex[chunkWithIndex.size - 1] = numChunks.toByte()

         // store the chunk in the array
         chunks[i] = chunkWithIndex

         chunkIndex++
   }

   return chunks
}

fun joinChunks(chunks: Array<ByteArray>): ByteArray {
   val sortedChunks = chunks.sortedBy { it[0] }
   var reassembledByteArray = ByteArray(0)
   for (chunk in sortedChunks) {
         val chunkData = chunk.copyOfRange(1, chunk.size - 1)
         reassembledByteArray = reassembledByteArray.copyOf(reassembledByteArray.size + chunkData.size)
         chunkData.copyInto(reassembledByteArray, reassembledByteArray.size - chunkData.size)
   }

   return decompressByteArray(reassembledByteArray)
}

```

## 工作流程

### 客户端到中继

- 客户端想要发送给中继的任何消息都将是一条写入消息。
- 客户端从中继接收的任何消息都将是一条读取消息。

### 中继到客户端

中继应通过使用读取特征的通知操作来通知客户端任何与订阅过滤器匹配的新事件。之后，客户端可以继续从中继读取消息。

### 设备同步

考虑到 BLE 的特性，两个设备之间的直接连接可能非常不稳定，存在数小时甚至数天的间隙。这就是为什么遵循 [NIP-77](./77.md) 定义一个同步过程至关重要，但需要根据技术的限制进行调整。

在两个设备成功连接并建立客户端-服务器角色后，设备将使用半双工通信来间歇性地发送和接收消息。

#### 半双工同步

在两个设备连接后，客户端立即开始工作流程，发送第一条消息。

1. 客户端 - 写入 ["NEG-OPEN"](/77.md#initial-message-client-to-relay) 消息。
2. 服务器 - 发送 `write-success`。
3. 客户端 - 发送 `read-message`。
4. 服务器 - 响应 ["NEG-MSG"](./77.md#subsequent-messages-bidirectional) 消息。
5. 客户端 -
   1. 如果客户端有服务器上缺失的消息，则写入一条 `EVENT`。
   2. 如果客户端没有服务器上缺失的消息，则写入 `EOSE`。在这种情况下，后续发送给服务器的消息将为空，而服务器声称还有更多笔记要给客户端。
6. 服务器 - 发送 `write-success`。
7. 客户端 - 发送 `read-message`。
8. 服务器 -
   1. 如果服务器有客户端上缺失的消息，则响应一条 `EVENT`。
   2. 如果客户端没有服务器上缺失的消息，则响应 `EOSE`。在这种情况下，后续对客户端的响应将为空。
9. 如果客户端检测到设备尚未同步，则跳转到步骤 5。
10. 在两个设备检测到两端都没有更多缺失事件后，工作流程将在此时暂停。

#### 半双工事件传播

当两个设备已连接并同步时，其中一个设备可能从另一个连接的对等方收到新消息。设备 `MUST` 记录哪些笔记已发送给其连接的对等方。如果发现新收到的事件在某个已连接且同步的对等方中缺失：

1. 如果对等方是服务器：
   1. 客户端 - 写入 `EVENT`。
   2. 服务器 - 发送 `write-success`。
2. 如果对等方是客户端：
   1. 服务器 - 向客户端发送空通知。
   2. 客户端 - 发送 `read-message`。
   3. 服务器 - 响应 `EVENT`。
