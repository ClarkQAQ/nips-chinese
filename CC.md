NIP-CC
======

地理寻宝事件
-----------------

`draft` `optional`

本 NIP 定义了在 Nostr 上进行地理寻宝的事件种类。这些事件允许用户以去中心化方式创建、分享和记录地理藏宝。

## 地理藏宝列表事件（Kind 37516）

地理藏宝列表事件是 kind `37516` 的可寻址事件，结构如下：

```json
{
  "kind": 37516,
  "content": "<cache description>",
  "tags": [
    ["d", "<cache-identifier>"],
    ["name", "<cache-name>"],
    ["g", "<geohash>"],
    ["D", "<1-5>"],
    ["T", "<1-5>"],
    ["S", "<size>"],
    ["t", "<type>"],
    ["n", "<type-modifier>"],
    ["hint", "<plaintext hint>"],
    ["mission", "<key quest mission>"],
    ["image", "<image-url>"],
    ["r", "<relay-url>"],
    ["verification", "<verification-pubkey-hex>"]
  ]
}
```

列表事件需要所有关于藏宝的信息以及与寻找藏宝相关的信息。这些包括 `name`（名称）、位置（`g`）、难度（`D`）和地形（`T`）评分以及大小（`S`）。藏宝类型（`t`）是可选的，如果未指定，默认为 `traditional`（传统）。

藏宝类型由各个客户端决定，常见类型包括 `traditional`（传统）、`multi`（多步）和 `mystery`（谜题）。客户端应根据其实现需求决定支持哪些藏宝类型。

这些要求是众所周知的，并遵循现有标准，例如 [geocaching.com](https://www.geocaching.com/help/index.php?pg=kb.chapter&id=97) 上概述的标准。

这些事件假定由藏宝的提交者拥有，核心细节应由该提交者维护。然而，社区日志也应提供关于藏宝当前状态和有效性的上下文。

### 内容

content 字段包含藏宝描述以及任何关于藏宝的附加信息。

### 标签

- `d`（必需）- 藏宝的唯一标识符
- `name`（必需）- 藏宝的人类可读名称
- `g`（必需）- 藏宝位置的地理哈希。为支持邻近搜索，包含多个不同精度级别（3-9 个字符）的地理哈希标签
- `D`（必需）- 整数 1-5，表示谜题/寻找难度（可索引）
- `T`（必需）- 整数 1-5，表示地形难度（可索引）
- `S`（必需）- 以下之一：`micro`（微型）、`small`（小型）、`regular`（常规）、`large`（大型）、`other`（其他）（可索引）
- `t`（可选）- 藏宝类型，常见值包括：`traditional`（传统）、`multi`（多步）、`mystery`（谜题）。如果未指定，默认为 `traditional`
- `n`（可选）- 影响生命周期、认领语义或奖品性质的类型修饰符。参见[类型修饰符](#type-modifiers)。可以有多个 `n` 标签，但每个修饰符类别最多一个
- `hint`（可选）- 帮助寻找藏宝的纯文本提示
- `mission`（可选）- 纯文本"关键任务"任务，寻找者需完成才能合法认领藏宝（例如口令、谜语答案或要携带的物品）。一个宝藏 `MUST NOT` 包含多于一个 `mission` 标签；如果存在多个，客户端 `SHOULD` 使用第一个并忽略其余部分。当存在时，客户端 `SHOULD` 限制认领日志的提交仅限于拥有人身在场证明（通常是来自藏宝位置的验证密钥）的发现者。任务的完成情况 `MAY` 记录为 [NIP-GD](NIP-GD.md) 的好事事件，其 `a` 标签引用该藏宝
- `image`（可选）- 与藏宝相关的图片 URL
- `r`（可选）- 日志的首选中继 URL
- `verification`（可选）- 用于验证此藏宝发现的十六进制编码公钥
- `F`（可选）- 锁定首先发现者（first-to-find）获胜者。仅在宝藏带有 `first-to-find` 的 `n` 修饰符时有效。格式：`["F", "<winner-pubkey-hex>"]`。参见[类型修饰符 › `first-to-find`](#claim-semantics)。最多应有一个 `F` 标签

## 发现日志事件（Kind 7516）

发现日志事件记录对地理藏宝的成功访问：

```json
{
  "kind": 7516,
  "content": "<log message>",
  "tags": [
    ["a", "37516:<pubkey>:<d-tag>"]
  ]
}
```

### 标签

- `a`（必需）- 引用被记录的地理藏宝
- `image`（可选）- 访问时的照片
- `verification`（可选）- 嵌入的验证事件（参见"已验证的发现"部分）

## 评论日志事件（Kind 1111）

非发现日志使用评论事件（kind `1111`），遵循 NIP-22 评论结构：

```json
{
  "kind": 1111,
  "content": "<log message>",
  "tags": [
    ["A", "37516:<pubkey>:<d-tag>"],
    ["K", "37516"],
    ["P", "<cache-owner-pubkey>"],
    ["a", "37516:<pubkey>:<d-tag>"],
    ["k", "37516"],
    ["p", "<cache-owner-pubkey>"],
    ["t", "<log-type>"]
  ]
}
```

这些事件通过人工报告捕获关于藏宝的失败、笔记和状态相关信息，遵循 NIP-22 评论线程模型，其中地理藏宝列表既是根内容也是父内容。

评论日志类型包括 `dnf`（未找到）、`note`（有帮助或中立的上下文）和 `maintenance`（藏宝需要注意）。如果不存在 `t` 标签，则假定该评论为普通笔记。

藏宝的所有者可以使用标签 `t` 中的 `archived` 标签值正式退役藏宝，从而允许保留藏宝的历史而不完全删除它。

### 标签

`A`/`K`/`P`（根）和 `a`/`k`/`p`（父）标签遵循 [NIP-22](https://github.com/nostr-protocol/nips/blob/master/22.md)。对于这些顶级评论，根和父是相同的：它们引用地理藏宝列表（`37516:<pubkey>:<d-tag>`）、kind `37516` 和藏宝所有者的公钥。

- `t`（可选）- 日志类型：`dnf`、`note`、`maintenance`、`archived`。如果省略，假定为 `note`
- `image`（可选）- 访问时的照片

## 地理藏宝验证事件（Kind 7517）

验证事件提供某人已实际定位地理藏宝的密码学证明。这些事件由藏宝的验证私钥签名。

```json
{
  "kind": 7517,
  "content": "Geocache verification for <finder-npub>",
  "tags": [
    ["a", "<finder-pubkey-hex>:<geocache-naddr>"]
  ]
}
```

### 内容

content 字段必须遵循静态格式：`"Geocache verification for <finder-npub>"`，其中 `<finder-npub>` 是发现藏宝者的 NIP-19 编码公钥（npub）。

### 标签

- `a`（必需）- 复合标识符，包含发现者的十六进制格式公钥和正在验证的地理藏宝 naddr

### 用法

当发现者获得藏宝的验证私钥（通常通过藏宝位置的二维码）时，会创建验证事件。该事件必须由藏宝的验证私钥签名，并且可以：

1. 作为 JSON 字符串嵌入到已验证的发现日志事件（kind 7516）的 `verification` 标签中。
2. 作为独立事件发布到中继。
3. 同时嵌入和发布以实现冗余。

## 已验证的发现

启用了验证的地理藏宝可以提供密码学证明，证明发现者实际定位了该藏宝。这是通过由藏宝验证密钥签名的验证事件（kind 7517）实现的。

### 验证流程

当藏宝的 `verification` 标签包含公钥时，发现者可以通过以下方式创建已验证的日志：

1. 获取藏宝的验证私钥（通常通过藏宝位置的二维码）
2. 创建由此密钥签名的验证事件（kind 7517）
3. 将验证事件嵌入其日志条目中

### 验证验证

要验证已验证的发现：

1. 检查验证事件是否由预期的验证公钥签名
2. 验证 `a` 标签中的发现者公钥是否与日志作者匹配
3. 确认 `a` 标签中的地理藏宝 naddr 正确引用目标藏宝
4. 使用标准 Nostr 验证方法验证事件签名

## 类型修饰符

地理藏宝列表 `MAY` 包含一个或多个 `n` 标签，以附加的类型修饰符对地理藏宝进行分类。与 `t` 藏宝类型标签（描述*如何*找到藏宝）不同，`n` 修饰符描述*藏宝发布后的行为方式*——其生命周期、认领语义或奖品性质。

`mission` 标签在更广泛的意义上也是一种类型修饰符，但它使用自己专用标签，因为它携带有效载荷（任务文本）。客户端 `SHOULD` 在显示目的上（徽章、过滤器等）一致地对待 `mission` 和 `n` 修饰符。

### 规则

1. 每个修饰符值恰好属于一个**类别**（见下文）。
2. 一个地理藏宝 `SHOULD` 每个类别最多包含一个 `n` 标签。如果存在同一类别的多个值，客户端 `SHOULD` 使用第一个出现并忽略其余部分。
3. 不同类别的修饰符可以自由组合。除非特定修饰符的定义另有说明，否则任何组合都是有效的。
4. 客户端 `SHOULD` 忽略他们不认识识别的 `n` 值，允许在定义新修饰符时向前兼容。

### 类别和修饰符

#### 认领语义

影响如何解释对宝藏的认领的修饰符。

- `first-to-find`（首先发现）——单次认领的地理藏宝列表。第一个已验证的发现日志（kind 7516，带有有效的嵌入 kind 7517）构成独占认领。后续已验证的发现日志仍然是对实际到访的有效记录，但不构成额外认领。一旦存在任何有效的已验证发现日志，客户端 `SHOULD` 将地理藏宝列表视为已有效归档，隐藏发现提交界面并突出显示获胜的发现者。需要在地理藏宝列表事件上有一个 `verification` 标签。

  确定获胜日志（锁定前，临时的）：
  - 获胜日志是 `created_at` 值最早的已验证发现日志。
  - `created_at` 相同时，按事件 `id` 的升序词典比较打破平局。
  - 所有已验证日志都是亲身到访的证据（需要二维码访问才能生成）；独占认领仅归属于最早的日志。

  锁定获胜者（`F` 标签）：
  - 一旦地理藏宝列表创建者确认了认领，创建者 `SHOULD` 发布地理藏宝列表事件的新修订版，既归档列表（添加 `["t", "archived"]`），又通过附加 `F` 标签锁定获胜者：
    `["F", "<winner-pubkey-hex>"]`
  - 值是获胜发现者的公钥（小写十六进制）。特定的获胜已验证发现日志可通过查询其 `a` 标签引用此宝藏且作者与 `F` 公钥匹配的已验证发现日志来恢复。
  - 最多应有一个 `F` 标签。如果存在多个，客户端 `SHOULD` 使用第一个。
  - 当存在 `F` 标签时，客户端 `MUST` 将独占认领归属于 `F` 标签中的公钥，无论当前哪个已验证发现日志看起来最早。由于 `created_at` 由作者提供且可伪造，这保护了锁定的认领不被带有伪造更早时间戳的后续日志所取代。

#### 奖品性质

描述实物宝藏是什么的修饰符。

- `art`（艺术品）——地理藏宝本身是一件实物艺术作品（版画、雕塑、贴纸、zine、彩绘物品、壁画、装置等）。该作品是可带走的、可在原地观看的、仅可拍照的，或通过其他方式互动的，由其他修饰符和宝藏的 `content` 描述决定。

### 向前兼容

新的修饰符 `MAY` 在本 NIP 的未来修订版或补充 NIP 中定义。新的类别 `MAY` 也被引入。实现本 NIP 的客户端 `SHOULD` 忽略未知的 `n` 值，而不是拒绝该事件。

## 地理藏宝策展列表事件（Kind 37517）

策展列表事件是 kind `37517` 的可寻址事件，将地理藏宝分组为策划的集合。客户端可以将其呈现为冒险、路线、寻宝游戏或任何其他主题体验。

```json
{
  "kind": 37517,
  "content": "<full description>",
  "tags": [
    ["d", "<list-identifier>"],
    ["title", "<list-title>"],
    ["description", "<short summary>"],
    ["image", "<banner-image-url>"],
    ["g", "<geohash>"],
    ["theme", "<page-theme>"],
    ["map", "<map-style>"],
    ["a", "37516:<pubkey>:<d-tag>"],
    ["a", "37516:<pubkey>:<d-tag>"]
  ]
}
```

### 内容

content 字段包含策展列表的完整描述——规则、提示、叙述或创建者想要提供的任何其他上下文。

### 标签

- `d`（必需）- 列表的唯一标识符
- `title`（必需）- 列表的人类可读名称
- `a`（必需，1 个以上）- 引用地理藏宝列表事件（kind 37516 或 37515）。顺序是保留的且有意义的
- `description`（可选）- 在浏览/卡片视图中显示的简短摘要
- `image`（可选）- 横幅图片 URL
- `g`（可选）- 列表中心位置的地理哈希。包含多个精度级别（3-6 个字符）以便于发现
- `theme`（可选）- 列表的默认页面主题。客户端在显示列表时应应用此主题，除非用户明确选择了不同的主题。支持的值：`adventure`（冒险）、`mojave`（莫哈韦）
- `map`（可选）- 列表的默认地图样式。客户端在显示列表时应将此作为初始地图样式，但允许用户更改。支持的值：`original`（原始）、`dark`（深色）、`satellite`（卫星）、`adventure`（冒险）

### 跨作者引用

策展列表可以引用任何公开的地理藏宝，无论作者是谁。`a` 标签使用标准的 Nostr 可寻址事件坐标（`<kind>:<pubkey>:<d-tag>`），允许单个列表跨越来自多个创建者的地理藏宝。

## 客户端

为了获得最佳地理寻宝体验，实现地理寻宝支持的客户端应：

- 支持提示编码，如 ROT13，以防止剧透。
- 根据近期日志模式确定藏宝状态。多个 DNF 条目和/或维护笔记表明藏宝存在问题。
- 在可用时，将日志发布到藏宝 `r` 标签中指定的中继。
- 在接受藏宝提交之前，验证地理哈希精度满足最低要求（8 个以上字符，微型藏宝 9 个以上字符）。

## 示例

### 基本藏宝

```json
{
  "kind": 37516,
  "content": "The first Nostr treasure, left in the aftermath of Oslo Freedom Forum!",
  "tags": [
    ["d", "first-treasure-1748619568668"],
    ["name", "First Treasure"],
    ["g", "u4x"],
    ["g", "u4xs"],
    ["g", "u4xsu"],
    ["g", "u4xsu6"],
    ["g", "u4xsu6r"],
    ["g", "u4xsu6ry"],
    ["g", "u4xsu6ryb"],
    ["D", "1"],
    ["T", "1"],
    ["S", "small"],
    ["t", "traditional"],
    ["hint", "In the branches"],
    ["image", "https://blossom.primal.net/74efe01a767b27dead71b8a9bb8278a108360438e78e55194ed9ab14a9382dd3.jpg"]
  ]
}
```

### 已验证的藏宝

```json
{
  "kind": 37516,
  "content": "High-security treasure requiring physical verification!",
  "tags": [
    ["d", "verified-treasure-1748619568669"],
    ["name", "Verified Treasure"],
    ["g", "u4xsu6ry"],
    ["D", "3"],
    ["T", "2"],
    ["S", "small"],
    ["t", "traditional"],
    ["hint", "Look for the secret code"],
    ["verification", "6805d4e5c0df48b4f76e2fdcb67a2acb1d97567b01c6fe17a236dc32f34f1c07"]
  ]
}
```

### 带关键任务的藏宝

```json
{
  "kind": 37516,
  "content": "Solve the riddle to claim your prize.",
  "tags": [
    ["d", "key-quest-treasure-1748619568670"],
    ["name", "Riddle of the Old Oak"],
    ["g", "u4xsu6ry"],
    ["D", "4"],
    ["T", "2"],
    ["S", "small"],
    ["t", "mystery"],
    ["hint", "Count the rings on the fallen log"],
    ["mission", "Bring a token of nature you found along the way"],
    ["verification", "6805d4e5c0df48b4f76e2fdcb67a2acb1d97567b01c6fe17a236dc32f34f1c07"]
  ]
}
```

### 首先发现的艺术品

一个单次认领的宝藏，其中藏宝本身是一件实物艺术品。第一个已验证的发现者是独占认领者；作品如何兑现（带回家、拍照等）在藏宝内容中描述。

```json
{
  "kind": 37516,
  "content": "Hand-pulled linocut, edition of 1, signed on the back next to the QR. Whoever finds it keeps it.",
  "tags": [
    ["d", "linocut-aftermath-1748619568671"],
    ["name", "Aftermath (Linocut #1)"],
    ["g", "u4xsu6ry"],
    ["D", "2"],
    ["T", "2"],
    ["S", "small"],
    ["t", "traditional"],
    ["n", "first-to-find"],
    ["n", "art"],
    ["hint", "Behind glass, but not in a frame"],
    ["image", "https://blossom.primal.net/example-linocut.jpg"],
    ["verification", "6805d4e5c0df48b4f76e2fdcb67a2acb1d97567b01c6fe17a236dc32f34f1c07"]
  ]
}
```

### 发现日志

```json
{
  "kind": 7516,
  "content": "Found it! Great hiding spot.",
  "tags": [
    ["a", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:first-treasure-1748619568668"]
  ]
}
```

### 未找到日志

```json
{
  "kind": 1111,
  "content": "Searched for 30 minutes but couldn't find it. Maybe it's missing?",
  "tags": [
    ["A", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:first-treasure-1748619568668"],
    ["K", "37516"],
    ["P", "0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd"],
    ["a", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:first-treasure-1748619568668"],
    ["k", "37516"],
    ["p", "0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd"],
    ["t", "dnf"]
  ]
}
```

### 笔记日志

```json
{
  "kind": 1111,
  "content": "Lots of muggles around during the day. Best to visit in the evening.",
  "tags": [
    ["A", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:first-treasure-1748619568668"],
    ["K", "37516"],
    ["P", "0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd"],
    ["a", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:first-treasure-1748619568668"],
    ["k", "37516"],
    ["p", "0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd"],
    ["t", "note"]
  ]
}
```

### 验证事件

```json
{
  "kind": 7517,
  "content": "Geocache verification for npub1qc0lc5lxnhxnfxlw2lxkv4x4vp6xsf4d5qwvlhfx6qmz6x4nfhqd8h2z3",
  "tags": [
    ["a", "0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:naddr1qqxnzd3e8q6n2dfk8qcnjve48qmnsw3jsqgswaehxw309aex2mrp0yhx6tpdsek6w309aex2mrp0yh56tnwdus8vatjvs6kzdrz956k7tjzw6qzypzgd2dmgxhxf34hnlw2y03nckr8f4g6mw9flxqq65v94zkp77rqfgrf8"]
  ],
  "pubkey": "6805d4e5c0df48b4f76e2fdcb67a2acb1d97567b01c6fe17a236dc32f34f1c07",
  "created_at": 1672531200,
  "sig": "a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456789012345678901234567890abcdef1234567890abcdef1234567890abcdef12"
}
```

### 已验证的发现日志

```json
{
  "kind": 7516,
  "content": "Found it! Great hiding spot.",
  "tags": [
    ["a", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:verified-treasure-1748619568669"],
    ["verification", "{\"kind\":7517,\"content\":\"Geocache verification for npub1qc0lc5lxnhxnfxlw2lxkv4x4vp6xsf4d5qwvlhfx6qmz6x4nfhqd8h2z3\",\"tags\":[[\"a\",\"0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:naddr1qqxnzd3e8q6n2dfk8qcnjve48qmnsw3jsqgswaehxw309aex2mrp0yhx6tpdsek6w309aex2mrp0yh56tnwdus8vatjvs6kzdrz956k7tjzw6qzypzgd2dmgxhxf34hnlw2y03nckr8f4g6mw9flxqq65v94zkp77rqfgrf8\"]],\"pubkey\":\"6805d4e5c0df48b4f76e2fdcb67a2acb1d97567b01c6fe17a236dc32f34f1c07\",\"created_at\":1672531200,\"sig\":\"a1b2c3d4e5f6789012345678901234567890abcdef1234567890abcdef123456789012345678901234567890abcdef1234567890abcdef1234567890abcdef12\"}"]
  ]
}
```

### 地理藏宝策展列表

```json
{
  "kind": 37517,
  "content": "Explore the festival grounds and find all hidden treasures before the jousting tournament!",
  "tags": [
    ["d", "ren-fest-hunt-1748619568670"],
    ["title", "Texas Ren Fest Treasure Hunt"],
    ["description", "Find all the hidden treasures at the festival!"],
    ["image", "https://blossom.primal.net/banner-example.jpg"],
    ["g", "9vk"],
    ["g", "9vk5"],
    ["g", "9vk5b"],
    ["g", "9vk5b7"],
    ["theme", "adventure"],
    ["map", "adventure"],
    ["a", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:first-treasure-1748619568668"],
    ["a", "37516:0461fcbecc4c3374439932d6b8f11269ccdb7cc973ad7a50ae362db135a474dd:verified-treasure-1748619568669"]
  ]
}
```
