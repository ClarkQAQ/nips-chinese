# NIP-EE

## 使用消息层安全（MLS）协议的端到端加密消息

`草案` `可选`

此 NIP 标准化了如何在 Nostr 中使用 [MLS 协议](https://www.rfc-editor.org/rfc/rfc9420.html)进行高效的端到端加密（E2EE）直接消息和群组消息。

## 背景

最初，Nostr 中的一对一直接消息（DM）通过 [NIP-04](04.md) 中定义的方案进行。不推荐使用此 NIP，因为虽然它加密了消息内容（提供了不错的机密性），但泄露了关于对话参与方的大量元数据（完全缺乏隐私性）。

随着 [NIP-44](44.md) 的加入，我们有了一个改进的加密方案，它改善了机密性保证，但没有定义使用此加密方案进行直接消息的新方案。因此，对隐私性几乎没有改善。

最近，[NIP-17](17.md) 将 [NIP-44](44.md) 加密与 [NIP-59](59.md) 礼品包装结合，将加密的直接消息隐藏在另一组事件中，确保无法看到谁在与谁交谈以及消息何时在用户之间传递。这在很大程度上解决了元数据泄露问题；虽然仍然可以看到用户正在接收礼品包装事件，但您无法分辨来自谁以及礼品包装外层事件中包含什么类型的事件。这提供了一定程度的可否认性/否认性，但没有解决前向保密或妥协后安全性。也就是说，如果用户的私钥（或用于加密消息的两个用户之间共享的计算会话密钥）被泄露，攻击者将完全访问这些用户之间发送的所有过去和未来的直接消息。

此外，[NIP-04](04.md) 和 [NIP-17](17.md) 都没有尝试解决群组消息的问题。

### 为什么这很重要？

没有适当的端到端加密，Nostr 无法用作安全消息客户端的协议。虽然像 Signal 这样的客户端在端到端加密方面做得很好，但它们仍然依赖于中心化服务器，因此可能被强大的（即国家级）行为者关闭。Nostr 的目标不仅是保护免受中心化实体审查您和您的通信，还要保护免受国家级行为者完全阻止这些服务存在的能力。通过用去中心化中继器替换中心化服务器，我们几乎不可能让中心化行为者完全阻止个人用户之间的通信。

### 此 NIP 的目标

1. 私密 _且_ 机密的直接消息和群组消息
   1. **私密** 意味着观察者无法分辨 Alice 和 Bob 在互相交谈，或者 Alice 是特定群组的一部分。这必然需要保护元数据。
   2. **机密** 意味着对话内容只能被预期的接收者查看。
2. 前向保密和妥协后安全性
   1. **前向保密** 意味着即使密钥材料泄露，过去的加密内容仍然保持加密。
   2. **妥协后安全性** 意味着泄露密钥材料不允许攻击者无限期地继续阅读未来的消息。
3. 为大型群组高效扩展
4. 允许在单个对话/群组中使用多个设备/客户端。

### 为什么选择 MLS？

此方案调整了消息层安全（MLS）协议以与 Nostr 一起使用。您可以将 MLS 视为 Signal 协议的演进。然而，它显著改善了大型群组消息加密操作的可扩展性（线性 -> 对数），构建用于适应联邦环境，并且还允许随时间优雅地更新密码套件和版本。此外，它非常灵活，对发送的消息内容不可知。

解释 MLS 协议超出了此 NIP 的范围，但您可以在其[架构概述](https://www.ietf.org/archive/id/draft-ietf-mls-architecture-13.html)或 [RFC](https://www.rfc-editor.org/rfc/rfc9420) 中阅读更多信息。MLS 正在成为 IETF 下的互联网标准，因此协议本身经过了极其充分的审查和研究。这也意味着随着 MLS 获得更多采用，未来有可能实现跨网络消息互操作性。

## 核心 MLS 概念

来自 [MLS 架构概述](https://www.ietf.org/archive/id/draft-ietf-mls-architecture-13.html)：

> MLS 为客户端提供了一种在群组内安全通信的方式。例如，一组用户可能使用手机或笔记本电脑上的客户端加入群组并相互通信。群组可能小到两个客户端（例如，用于简单的人对人消息传递）或大到数十万。作为群组一部分的客户端是该群组的成员。随着群组更改成员身份和群组或成员属性，它们从一个时期前进到另一个时期，群组的密码状态也会演变。
>
> 群组表示为树，将成员表示为树的叶子。它用于高效地加密给成员的子集。每个成员都有一个称为 LeafNode 对象的状态，保存客户端的身份、凭据和能力。

MLS 协议的工作是管理和演变群组的密码状态。这包括管理群组的成员身份、群组的密码状态（棘轮树、密钥和消息的加密/解密/身份验证）以及管理群组随时间的演变。

### 群组

群组由其第一个成员创建，然后邀请一个或多个其他成员。群组随时间在称为 `Epochs` 的块中演变。新时期通过一个或多个 `Proposal` 消息提出，然后通过 `Commit` 消息提交。

### 客户端

用户加入群组的设备/客户端对（例如 iOS 上的 Primal 或 web 上的 Coracle）在树中表示为 `LeafNode`。在这方面，术语 `Client` 和 `Member` 是可互换的。不可能在多个 `Clients` 之间共享群组状态。如果用户从 2 个单独的设备加入群组，它们的状态是分开的，它们将被跟踪为群组的 2 个单独成员。

### 消息

在群组内发送几种不同类型的消息。其中一些是用于随时间更新群组状态的控制消息。这些包括 `Welcome`、`Proposal` 和 `Commit` 消息。其他是在群组成员之间发送的实际消息。这些包括 `Application` 消息。

MLS 中的消息是"框架化的"。意味着它们被包装在包含有关发送者、时期、时期内的消息索引和消息内容的信息的数据结构中。这种框架化使得即使消息乱序到达也能正确验证和解密消息。

MLS 对发送的消息的"内容"不可知。这是 MLS 的一个关键特性，允许 MLS 用于各种应用程序。

MLS 也对用于发送消息的传输协议不可知。显然对我们来说，我们将使用 websockets、Nostr 事件和中继器。

## 此 NIP 的重点

此 NIP 重点关注如何使用 Nostr 执行 MLS 协议所需的身份验证服务和传递服务功能。大多数客户端将选择使用 MLS 实现来处理密钥、棘轮、群组状态管理和 MLS 协议本身的其他方面。[OpenMLS](https://github.com/openmls/openmls) 是实现 MLS 的最活跃开发的库。

此 NIP 指定以下内容：

1. Nostr 客户端应该[创建 MLS 群组](#creating-groups)的标准化方法。
2. Nostr 客户端应该使用的 MLS [`Credential`](#mls-credentials) 的必需格式，以在群组中表示 Nostr 用户。
3. 发布到中继器的[密钥包事件](#keypackage-event-and-signing-keys)的结构，允许 Nostr 用户异步添加到群组。
4. 发布到中继器的[群组事件](#group-events)的结构，表示群组状态的演变和在群组中发送的消息内容。

## 安全考虑

这是此 NIP 在各种场景中的安全权衡和考虑的简明概述。NIP 努力完全维护 MLS 安全保证。

### 前向保密和妥协后安全性

- 根据 MLS 规范，密钥一旦用于加密或解密消息就会被删除。这通常由 MLS 实现库本身处理，但客户端需要注意确保它们不会比绝对必要的时间更长地存储秘密（特别是[导出器秘密](#group-events)）。
- 此 NIP 维护 MLS 前向保密和妥协后安全性保证。您可以在 MLS 架构概述的[前向保密和妥协后安全性](https://www.ietf.org/archive/id/draft-ietf-mls-architecture-15.html#name-forward-and-post-compromise)部分阅读更多信息。

### 各种密钥的泄露

- 此 NIP 不依赖用户的 Nostr 身份密钥来处理 MLS 消息协议的任何方面。用户的 Nostr 身份密钥的泄露不会提供对任何基于 MLS 的群组中过去或未来消息的访问。
- 有关 MLS 密钥泄露的完整讨论，请参见 [MLS 架构概述](https://www.ietf.org/archive/id/draft-ietf-mls-architecture-15.html#name-endpoint-compromise)的端点泄露部分。

### 元数据

- 发布到中继器的唯一群组特定元数据是 Nostr 群组 ID 值。此值用于标识群组消息事件（`kind: 445`）的 `h` 标签中的群组。这些事件是临时发布的，此 Nostr 群组 ID 值可以在群组的生命周期内由群组管理员更新。这是一个权衡，以确保群组参与者和群组大小被混淆，但仍然可以有效地将群组消息扇出给所有参与者。此事件的内容字段是使用两种不同方式（使用 NIP-44 和 MLS）以及 MLS 群组状态/密钥加密的值。只有具有最新群组状态的群组成员才能解密和阅读这些消息。
- 用户的密钥包事件可以一次或多次用于添加到群组。这里存在固有的权衡：重用密钥包（初始签名密钥）带有一定程度的风险，但只要用户在加入群组后立即轮换其签名密钥，这种风险就会得到缓解。此步骤还改善了整个群组的前向保密性。

### 设备泄露

实现此 NIP 的客户端应该采取一切预防措施，确保数据以安全的方式存储在设备上，并在设备泄露的情况下防止不必要的访问（例如静态加密、生物识别身份验证等）。也就是说，完全设备泄露应该被视为灾难性事件，被泄露设备所属的任何群组都应该被认为是泄露的，直到它们能够删除该成员并更新其群组状态。一些建议：

- 客户端应该支持并鼓励自毁消息（确保完整的对话历史记录不会永远在设备上可用）。
- 客户端应该定期向群组管理员建议删除不活跃的用户。
- 客户端应该定期建议（或自动）在每个群组中轮换用户的签名密钥。
- 客户端应该使用不属于群组状态或用户的 Nostr 身份密钥的秘密值在设备上加密群组状态和密钥。
- 客户端应该在可能的情况下使用安全飞地存储。

有关 MLS 安全考虑的完整讨论，请参见 [MLS RFC](https://www.rfc-editor.org/rfc/rfc9420.html#name-security-considerations) 的安全考虑部分。

## 创建群组

MLS 群组使用实际上是永久的随机 32 字节 ID 值创建。此 ID 应该被视为群组的私有信息，不得以任何形式发布到中继器。

客户端还必须确保它们在创建群组时使用的密码套件、能力和扩展与它们想要邀请到群组的用户宣传的那些兼容。它们可以通过用户发布的密钥包事件检查此信息。

创建新群组时，必须使用以下 MLS 扩展。

- [`required_capabilities`](https://docs.rs/openmls/latest/openmls/extensions/struct.RequiredCapabilitiesExtension.html)
- [`ratchet_tree`](https://docs.rs/openmls/latest/openmls/extensions/struct.RatchetTreeExtension.html)
- [`nostr_group_data`](https://github.com/rust-nostr/nostr/blob/master/mls/nostr-mls/src/extension.rs)

强烈推荐以下 MLS 扩展（更多[这里](#keypackage-event-and-signing-keys)）：
- [`last_resort`](https://docs.rs/openmls/latest/openmls/extensions/struct.LastResortExtension.html)

对 MLS 群组的更改通过首先创建一个或多个 `Proposal` 事件，然后在 `Commit` 事件中提交一组提案来实现。这些是 MLS 事件，不是 Nostr 事件。然而，为了群组状态正确演变，Commit 事件（表示一组特定的提案 - 如向群组添加新用户）必须发布到中继器供其他群组成员查看。有关更多信息，请参见[群组消息](#group-events)。

## MLS 凭据

MLS 中的 `Credential` 是用户身份的断言与签名密钥的耦合。构建 MLS 的 `Credentials` 时，客户端必须使用 `BasicCredential` 类型，并将 `identity` 值设置为用户 Nostr 身份密钥的 32 字节十六进制编码公钥。客户端不得允许用户更改身份字段，并且必须验证所有 `Proposal` 消息不尝试更改群组中任何凭据的身份字段。

`Credential` 还有一个关联的签名密钥。用户的初始签名密钥包含在密钥包事件中。签名密钥必须与用户的 Nostr 身份密钥不同。此签名密钥应该随时间轮换以提供改进的妥协后安全性。

## Nostr 群组数据扩展

如上所述，`nostr_group_data` 扩展是一个必需的 MLS 扩展，用于以密码学安全和可证明的方式将 Nostr 特定数据与 MLS 群组关联。创建新群组时必须将此扩展包含为必需能力。

扩展存储关于群组的以下数据：

- `nostr_group_id`：群组的 32 字节 ID。这与 MLS 使用的群组 ID 不同的值，可以随时间更改。此值是发送群组消息事件时在 `h` 标签中使用的群组 ID 值。
- `name`：群组的名称。
- `description`：群组的简短描述。
- `admin_pubkeys`：群组管理员的十六进制编码公钥数组。MLS 协议本身没有群组管理员的概念。客户端必须在对群组数据（此扩展中的任何内容）进行任何更改之前，或在更改群组成员身份（添加/删除成员）之前，或更新群组本身的任何其他方面（例如密码套件等）之前检查 `admin_pubkeys` 列表。注意，群组的所有成员都可以发送 `Proposal` 和 `Commits` 消息以更改其自己的凭据（例如更新其签名密钥）。
- `relays`：群组用于发布和接收消息的 Nostr 中继器 URL 数组。

所有这些值都可以使用 MLS `Proposal` 和 `Commit` 事件（由群组管理员）随时间更新。

## 密钥包事件和签名密钥

希望通过基于 MLS 的消息可达的每个用户必须首先发布至少一个密钥包事件。密钥包事件用于验证用户并创建必要的 `Credential` 以异步方式向群组添加成员。用户可以发布具有不同参数的多个密钥包事件（例如支持不同的密码套件或 MLS 扩展）。密钥包包括用于在群组内签署 MLS 消息的签名密钥。此签名密钥不得与用户的 Nostr 身份密钥相同。

应该最小化密钥包重用。然而，在正常的 MLS 使用中，密钥包在加入群组时被消耗。为了减少使用相同密钥包的多个群组邀请之间的竞争条件，Nostr 客户端应该使用"最后手段"密钥包。这需要在密钥包的能力上包含 `last_resort` 扩展（与群组相同）。

重要的是，客户端在通过最后手段密钥包加入群组后立即轮换用户的签名密钥，以改善妥协后安全性。签名密钥（密钥包事件中包含的公钥）用于在群组内签署。因此，实现此 NIP 的客户端必须确保它们保留对它们所属的每个群组的签名密钥的私钥材料的访问权限。

在大多数情况下，假设实现此 NIP 的客户端将管理密钥包事件的创建和轮换。

### 示例密钥包事件

```json
  {
    "id": <id>,
    "kind": 443,
    "created_at": <unix timestamp in seconds>,
    "pubkey": <main identity pubkey>,
    "content": "",
    "tags": [
        ["mls_protocol_version", "1.0"],
        ["ciphersuite", <MLS CipherSuite ID value e.g. "0x0001">],
        ["extensions", <An array of MLS Extension ID values e.g. "0x0001, 0x0002">],
        ["client", <client name>, <handler event id>, <optional relay url>],
        ["relays", <array of relay urls>],
        ["-"]
    ],
    "sig": <signed with main identity key>
}
```

- `content` 十六进制编码的来自 MLS 的序列化 `KeyPackageBundle`。
- `mls_protocol_version` 标签是必需的，必须是正在使用的 MLS 协议版本的版本号。现在，这是 `1.0`。
- `ciphersuite` 标签是此密钥包事件支持的 MLS 密码套件的值。[阅读更多关于 MLS 中的密码套件](https://www.rfc-editor.org/rfc/rfc9420.html#name-mls-cipher-suites)。
- `extensions` 标签是此密钥包事件支持的 MLS 扩展 ID 数组。[阅读更多关于 MLS 扩展](https://www.rfc-editor.org/rfc/rfc9420.html#name-extensions)。
- （可选）`client` 标签帮助其他客户端在接收到群组邀请但无法访问签名密钥时管理用户体验。
- `relays` 标签标识客户端将尝试发布此密钥包事件的每个中继器。这允许稍后删除密钥包事件。
- （可选）`-` 标签可用于确保密钥包事件仅由其经过身份验证的作者发布。在 [NIP-70](70.md) 中阅读更多信息

### 删除密钥包事件

客户端应该在任何时候成功处理给定密钥包事件的群组请求事件时删除所有列出的中继器上的密钥包事件。客户端也可以同时创建新的密钥包事件。

如果客户端无法处理欢迎消息（例如因为签名密钥是在另一个客户端上生成的），客户端不得删除密钥包事件，并且应该向用户显示人类可理解的错误。

### 轮换签名密钥

客户端必须定期轮换用户在它们所属的每个群组中的签名密钥。签名密钥轮换得越频繁，妥协后安全性就越强。此轮换通过 `Proposal` 和 `Commit` 事件完成，并通过群组事件广播给群组。[阅读更多关于 MLS 固有的前向保密和妥协后安全性](https://www.rfc-editor.org/rfc/rfc9420.html#name-forward-secrecy-and-post-co)。

### 密钥包中继器列表事件

`kind: 10051` 事件指示用户将其密钥包事件发布到的中继器。事件必须包含带有中继器 URI 的中继器标签列表。这些中继器应该对任何想要联系它们的用户可读。

```json
{
  "kind": 10051,
  "tags": [
    ["relay", "wss://inbox.nostr.wine"],
    ["relay", "wss://myrelay.nostr1.com"],
  ],
  "content": "",
  //...其他字段
}
```

### 欢迎事件

当通过 MLS `Commit` 消息向群组添加新用户时。向群组发送 `Commit` 消息的成员负责向被添加到群组的用户发送欢迎事件。此欢迎事件作为 [NIP-59](59.md) 礼品包装事件发送给用户。欢迎事件为新成员提供了加入群组并开始发送消息所需的上下文。

创建欢迎事件的客户端应该等到它们收到中继器的确认，表明它们的带有 `Commit` 的群组事件已被接收，然后再发布欢迎事件。

```json
{
   "id": <id>,
   "kind": 444,
   "created_at": <unix timestamp in seconds>,
   "pubkey": <nostr identity pubkey of sender>,
   "content": <serialized Welcome object>,
   "tags": [
      ["e", <ID of the KeyPackage Event used to add the user to the group>],
      ["relays", <array of relay urls>],
   ],
   "sig": <NOT SIGNED>
}
```

- `content` 字段是必需的，是包含 MLS `Welcome` 对象的序列化 MLSMessage 对象。
- `e` 标签是必需的，是用于将用户添加到群组的密钥包事件的 ID。
- `relays` 标签是必需的，是客户端应该查询群组事件的中继器列表。

欢迎事件然后按照 [NIP-59](59.md) 中详述的方式密封和礼品包装，然后发布。像所有密封和礼品包装的事件一样，`kind: 444` 事件绝不能被签名。这确保如果它们被泄露，它们将无法发布到中继器。

#### 大型群组

对于超过约 150 个参与者的群组，欢迎消息将变得大于 Nostr 允许的最大事件大小。目前正在进行 MLS 协议的工作，以支持不需要将完整的棘轮树状态发送给新成员的"轻量级"客户端欢迎。此部分将根据如何处理大型群组的建议进行更新。

## 群组事件

群组事件是在群组内发送的所有消息。这包括随时间更新共享群组状态的所有"控制"事件（`Proposal`、`Commit`）和在群组成员之间发送的消息（`Application` 消息）。

群组事件使用临时 Nostr 密钥对发布，以混淆群组参与者的数量和身份。客户端必须为它们发布的每个群组事件使用新的 Nostr 密钥对。

```json
{
   "id": <id>,
   "kind": 445,
   "created_at": <unix timestamp in seconds>,
   "pubkey": <ephemeral sender pubkey>,
   "content": <NIP-44 encrypted serialized MLSMessage object>,
   "tags": [
      ["h", <group id>]
   ],
   "sig": <signed with ephemeral sender key>
}
```
- `content` 字段是根据 [NIP-44](44.md) 加密的 [tls 样式](https://www.rfc-editor.org/rfc/rfc9420.html#name-the-message-mls-media-type)序列化 [`MLSMessage`](https://www.rfc-editor.org/rfc/rfc9420.html#section-6-4) 对象。然而，不是使用发送者和接收者密钥来派生 `conversation_key`，而是使用从 MLS [`exporter_secret`](https://www.rfc-editor.org/rfc/rfc9420.html#section-8.5) 生成的 Nostr 密钥对进行 NIP-44 加密来计算 `conversation_key` 值。本质上，您使用十六进制编码的 `exporter_secret` 值作为私钥（用作发送者密钥），计算该私钥的公钥（用作接收者密钥），然后继续使用标准 NIP-44 方案加密和解密消息。
- `exporter_secret` 值应该用 32 字节长度生成，标记为 `nostr`。此 `exporter_secret` 值在群组的每个新时期轮换。客户端应该在每次处理有效的 `Commit` 消息时生成新的 32 字节值。
- `pubkey` 是临时发送者的十六进制编码公钥。
- `h` 标签是 nostr 群组 ID 值（来自 Nostr 群组数据扩展）。

### 应用程序消息

应用程序消息是成员在群组内发送的消息。这些包含在 `MLSMessage` 对象中。这些消息的格式应该是适当类型的未签名 Nostr 事件。对于正常的直接消息或群组消息，客户端应该使用 `kind: 9` 聊天消息事件。如果用户对消息做出反应，它将是 `kind: 7` 事件，等等。

这意味着一旦应用程序消息被解密和反序列化，客户端就可以存储这些事件并将它们视为任何其他 Nostr 事件，有效地创建群组活动的私有 Nostr 提要并利用 Nostr 的所有功能。

这些内部未签名的 Nostr 事件必须为 `pubkey` 字段使用成员的 Nostr 身份密钥，客户端必须检查发送消息的成员的身份是否与内部 Nostr 事件的公钥匹配。

这些 Nostr 事件必须保持**未签名**，以确保如果它们泄露到中继器，它们不会被公开发布。这些 Nostr 事件不得包含任何"h"标签或其他会标识它们所属群组的标签。

### `Commit` 消息竞争条件

MLS 协议对几乎所有乱序到达的消息都具有弹性。然而，`Commit` 消息的顺序对于群组状态从一个时期正确移动到下一个时期很重要。鉴于 Nostr 作为去中心化网络的性质，客户端可能同时接收 2 个或更多 `Commit` 消息，所有这些消息都试图同时更新到新时期。

发送提交消息的客户端必须等到它们收到至少一个中继器的确认，表明它们的带有 `Commit` 的群组消息事件已被接收，然后才能将提交应用到它们自己的群组状态。

如果客户端接收到 2 个或更多试图更改同一时期的 `Commit` 消息，它们必须只应用它们接收到的 `Commit` 消息中的一个，由以下确定：

1. 使用 kind `445` 事件上的 `created_at` 时间戳。具有最低 `created_at` 值的 `Commit` 是要应用的消息。其他 `Commit` 消息被丢弃。
2. 如果两个或更多 `Commit` 消息的 `created_at` 时间戳相同，则应用具有最低 `id` 字段值的 `Commit` 消息。

客户端应该保留先前的群组状态一段时间，以便从分叉的群组状态中恢复。
