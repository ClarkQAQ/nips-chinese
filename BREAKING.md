# 重大变更

这是可能破坏现有实现的 NIP 变更历史记录，按时间倒序排列。

| 日期        | 提交      | NIP      | 变更 |
| ----------- | --------- | -------- | ------ |
| 2025-02-14  | [81908b6e](https://github.com/nostr-protocol/nips/commit/81908b6e) | [07](07_ZH.md), [46](46_ZH.md), [55](55_ZH.md) | 移除了 `getRelays` 和 `get_relays` |
| 2025-02-07  | [0023ca81](https://github.com/nostr-protocol/nips/commit/0023ca81) | [10](10_ZH.md) | 移除了 `"mention"` 标记 |
| 2025-01-31  | [6a4b125a](https://github.com/nostr-protocol/nips/commit/6a4b125a) | [71](71_ZH.md) | 视频事件更改为常规事件 |
| 2024-12-05  | [6d16019e](https://github.com/nostr-protocol/nips/commit/6d16019e) | [46](46_ZH.md) | 消息加密更改为 NIP-44 |
| 2024-11-12  | [2838e3bd](https://github.com/nostr-protocol/nips/commit/2838e3bd) | [29](29_ZH.md) | 移除了 `kind: 12` 和 `kind: 10`（改用 `kind: 1111`） |
| 2024-11-12  | [926a51e7](https://github.com/nostr-protocol/nips/commit/926a51e7) | [46](46_ZH.md) | 移除了 NIP-05 登录 |
| 2024-11-12  | [926a51e7](https://github.com/nostr-protocol/nips/commit/926a51e7) | [46](46_ZH.md) | 移除了 `create_account` 方法 |
| 2024-11-12  | [926a51e7](https://github.com/nostr-protocol/nips/commit/926a51e7) | [46](46_ZH.md) | 更改了 `connect` 参数和结果 |
| 2024-10-29  | [f1e8d2c4](https://github.com/nostr-protocol/nips/commit/f1e8d2c4) | [46](46_ZH.md) | bunker URL 应该使用 `remote-signer-key` |
| 2024-10-15  | [1cda2dcc](https://github.com/nostr-protocol/nips/commit/1cda2dcc) | [71](71_ZH.md) | 一些标签被替换为 `imeta` 标签 |
| 2024-10-15  | [1cda2dcc](https://github.com/nostr-protocol/nips/commit/1cda2dcc) | [71](71_ZH.md) | 废弃了 `kind: 34237` |
| 2024-10-07  | [7bb8997b](https://github.com/nostr-protocol/nips/commit/7bb8997b) | [55](55_ZH.md) | 更改了一些字段和传递数据 |
| 2024-08-18  | [3aff37bd](https://github.com/nostr-protocol/nips/commit/3aff37bd) | [54](54_ZH.md) | 内容应该是 Asciidoc 格式 |
| 2024-07-31  | [3ea2f1a4](https://github.com/nostr-protocol/nips/commit/3ea2f1a4) | [45](45_ZH.md) | 恢复了 [444ad28d](https://github.com/nostr-protocol/nips/commit/444ad28d) |
| 2024-07-30  | [444ad28d](https://github.com/nostr-protocol/nips/commit/444ad28d) | [45](45_ZH.md) | NIP-45 被弃用 |
| 2024-07-26  | [ecee40df](https://github.com/nostr-protocol/nips/commit/ecee40df) | [19](19_ZH.md) | `nrelay` 被弃用 |
| 2024-07-23  | [0227a2cd](https://github.com/nostr-protocol/nips/commit/0227a2cd) | [01](01_ZH.md) | 事件应该在 created_at 之后按 id 排序 |
| 2024-06-06  | [58e94b20](https://github.com/nostr-protocol/nips/commit/58e94b20) | [25](25_ZH.md) | 恢复了 [8073c848](https://github.com/nostr-protocol/nips/commit/8073c848) |
| 2024-06-06  | [a6dfc7b5](https://github.com/nostr-protocol/nips/commit/a6dfc7b5) | [55](55_ZH.md) | NIP 编号被更改 |
| 2024-05-25  | [5d1d1c17](https://github.com/nostr-protocol/nips/commit/5d1d1c17) | [71](71_ZH.md) | 移除了 `aes-256-gcm` 标签 |
| 2024-05-07  | [8073c848](https://github.com/nostr-protocol/nips/commit/8073c848) | [25](25_ZH.md) | e-tags 更改为不包含整个线程 |
| 2024-04-30  | [bad88262](https://github.com/nostr-protocol/nips/commit/bad88262) | [34](34_ZH.md) | 移除了 `earliest-unique-commit` 标签（改用 `r` 标签） |
| 2024-02-25  | [4a171cb0](https://github.com/nostr-protocol/nips/commit/4a171cb0) | [18](18_ZH.md) | 引用转发应该使用 `q` 标签 |
| 2024-02-21  | [c6cd655c](https://github.com/nostr-protocol/nips/commit/c6cd655c) | [46](46_ZH.md) | 参数被字符串化 |
| 2024-02-16  | [cbec02ab](https://github.com/nostr-protocol/nips/commit/cbec02ab) | [49](49_ZH.md) | 密码首先规范化为 NFKC |
| 2024-02-15  | [afbb8dd0](https://github.com/nostr-protocol/nips/commit/afbb8dd0) | [39](39_ZH.md) | 移除了 PGP 身份 |
| 2024-02-07  | [d3dad114](https://github.com/nostr-protocol/nips/commit/d3dad114) | [46](46_ZH.md) | 连接令牌格式被更改 |
| 2024-01-30  | [1a2b21b6](https://github.com/nostr-protocol/nips/commit/1a2b21b6) | [59](59_ZH.md) | `p` 标签变为可选 |
| 2023-01-27  | [c2f34817](https://github.com/nostr-protocol/nips/commit/c2f34817) | [47](47_ZH.md) | 应该遵守可选的过期标签 |
| 2024-01-10  | [3d8652ea](https://github.com/nostr-protocol/nips/commit/3d8652ea) | [02](02_ZH.md), [51](51_ZH.md) | 列表条目应该按时间顺序 |
| 2023-12-30  | [29869821](https://github.com/nostr-protocol/nips/commit/29869821) | [52](52_ZH.md) | 移除了 `name` 标签（改用 `title` 标签） |
| 2023-12-27  | [17c67ef5](https://github.com/nostr-protocol/nips/commit/17c67ef5) | [94](94_ZH.md) | 移除了 `aes-256-gcm` 标签 |
| 2023-12-03  | [0ba45895](https://github.com/nostr-protocol/nips/commit/0ba45895) | [01](01_ZH.md) | WebSocket 状态码 `4000` 被 `CLOSED` 消息替换 |
| 2023-11-28  | [6de35f9e](https://github.com/nostr-protocol/nips/commit/6de35f9e) | [89](89_ZH.md) | `client` 标签值被更改 |
| 2023-11-20  | [7822a8b1](https://github.com/nostr-protocol/nips/commit/7822a8b1) | [51](51_ZH.md) | `kind: 30001` 被弃用 |
| 2023-11-20  | [7822a8b1](https://github.com/nostr-protocol/nips/commit/7822a8b1) | [51](51_ZH.md) | `kind: 30000` 的含义被更改 |
| 2023-11-11  | [cbdca1e9](https://github.com/nostr-protocol/nips/commit/cbdca1e9) | [84](84_ZH.md) | 移除了 `range` 标签 |
| 2023-11-10  | [c945d8bd](https://github.com/nostr-protocol/nips/commit/c945d8bd) | [32](32_ZH.md) | 移除了 `l` 标签注释 |
| 2023-11-07  | [108b7f16](https://github.com/nostr-protocol/nips/commit/108b7f16) | [01](01_ZH.md) | `OK` 消息必须有 4 个项目 |
| 2023-10-17  | [cf672b76](https://github.com/nostr-protocol/nips/commit/cf672b76) | [03](03_ZH.md) | 移除了 `block` 标签 |
| 2023-09-29  | [7dc6385f](https://github.com/nostr-protocol/nips/commit/7dc6385f) | [57](57_ZH.md) | 在 `zap receipt` 中包含了可选的 `a` 标签 |
| 2023-08-21  | [89915e02](https://github.com/nostr-protocol/nips/commit/89915e02) | [11](11_ZH.md) | 移除了 `min_prefix` |
| 2023-08-20  | [37c4375e](https://github.com/nostr-protocol/nips/commit/37c4375e) | [01](01_ZH.md) | 具有相同时间戳的可替换事件应保留 id 最小的事件 |
| 2023-08-15  | [88ee873c](https://github.com/nostr-protocol/nips/commit/88ee873c) | [15](15_ZH.md) | `countries` 标签重命名为 `regions` |
| 2023-08-14  | [72bb8a12](https://github.com/nostr-protocol/nips/commit/72bb8a12) | [12](12.md), [16](16.md), [20](20.md), [33](33.md) | NIP-12, 16, 20 和 33 合并到 NIP-01 |
| 2023-08-11  | [d87f8617](https://github.com/nostr-protocol/nips/commit/d87f8617) | [25](25_ZH.md) | 空的 `content` 应被视为 "+" |
| 2023-08-01  | [5d63b157](https://github.com/nostr-protocol/nips/commit/5d63b157) | [57](57_ZH.md) | `zap` 标签被更改 |
| 2023-07-15  | [d1814405](https://github.com/nostr-protocol/nips/commit/d1814405) | [01](01_ZH.md) | `since` 和 `until` 过滤器应该是 `since <= created_at <= until` |
| 2023-07-12  | [a1cd2bd8](https://github.com/nostr-protocol/nips/commit/a1cd2bd8) | [25](25_ZH.md) | 支持自定义表情符号 |
| 2023-06-18  | [83cbd3e1](https://github.com/nostr-protocol/nips/commit/83cbd3e1) | [11](11_ZH.md) | `image` 重命名为 `icon` |
| 2023-04-13  | [bf0a0da6](https://github.com/nostr-protocol/nips/commit/bf0a0da6) | [15](15_ZH.md) | 不同的 NIP 重新添加为 NIP-15 |
| 2023-04-09  | [fb5b7c73](https://github.com/nostr-protocol/nips/commit/fb5b7c73) | [15](15_ZH.md) | NIP-15 合并到 NIP-01 |
| 2023-03-29  | [599e1313](https://github.com/nostr-protocol/nips/commit/599e1313) | [18](18_ZH.md) | NIP-18 重新回归 |
| 2023-03-15  | [e1004d3d](https://github.com/nostr-protocol/nips/commit/e1004d3d) | [19](19_ZH.md) | `1: relay` 更改为可选 |

2023-03-01 之前的重大变更尚未记录。

## 注意事项

- 如果不清楚某个变更是否为重大变更，我们会将其列出。
- 日期是合并日期，不一定是提交日期。
