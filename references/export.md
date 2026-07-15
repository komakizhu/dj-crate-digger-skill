# 平台导出与授权边界

## 平台顺序

默认导出优先级：SoundCloud → Apple Music → Spotify。用户指定 `export_platform` 时直接采用该目标；如果它同时出现在用户的 forbidden 列表中，先指出冲突并澄清，不自行选择另一个平台。

已知官方能力边界：

- SoundCloud：官方 OAuth 后可创建或更新 playlist。
- Apple Music：MusicKit 用户授权后可创建用户资料库 playlist。
- Spotify：OAuth 授权并获得 playlist modify scope 后可创建 playlist 并添加曲目。

不要仅凭这份说明假定当前运行环境已经拥有连接器、Client ID 或有效授权。每次先检查实际可用工具。

## 导出状态机

### 1. Prepare

先读取需求卡的 `export_intent`。`explain` 只返回流程；`prepare` 才做能力检查并生成确认摘要；只有当前上下文中确认过该摘要才能进入 `execute`。从 `export_platform` 确定目标平台，不要用 `search_platform` 或首选搜索平台代替。确定：

- 目标平台
- 要导出的版本，默认综合版
- 歌单名称
- 曲目数量
- 目标平台已匹配和未匹配数量
- 匹配依据（ISRC 或艺人 + 完整曲名 + mix/version）
- public / private；用户未指定时默认 private

### 2. Capability Check

检查当前环境是否存在：

- 官方平台连接器
- 支持官方 OAuth 的浏览器会话
- 用户已配置且允许使用的安全凭证

工具不可用时，不要声称可以一键创建。返回只含目标平台或用户允许平台的导出清单，并说明所缺能力；不要为了“方便导入”加入被禁止的平台链接。

### 3. Authorization

首次授权必须让用户在官方页面完成登录和同意。不要：

- 询问、读取或代填用户密码
- 绕过 MFA、CAPTCHA 或平台限制
- 将 access token、refresh token 写入报告、Skill、普通 JSON、`.env` 或 Git

默认只在当前会话使用授权。用户明确同意持久化后，只能使用连接器安全存储、系统 Keychain 或等价的受保护凭证存储。没有安全存储能力时拒绝持久化，但仍可继续当前会话。

### 4. Confirm

任何外部写入前，展示一行确认摘要：

```text
准备在 SoundCloud 创建 private 歌单 “Sunset Organic House”：综合版，30 首；已匹配 27 首，缺失 3 首。是否创建？
```

没有确认就停在此处。确认只覆盖摘要中这一个动作，不扩展为后续修改权限。

### 5. Create

确认后调用官方连接器或官方 API。一次只创建一个平台歌单。保持推荐顺序；平台批量限制需要分批时，最终验证曲目总数和顺序。

### 6. Reconcile

返回：

- 创建成功的歌单 URL
- 实际加入数量
- 未找到或失败的原始曲目清单
- 是否使用了替代曲目

目标平台找不到曲目时默认跳过并列出。只有用户明确允许相似替代后，才搜索替代，并逐项标注“原曲 → 替代曲 + 原因”；替代曲仍必须逐曲核验，不得因为标题相似自动写入。

## 匹配规则

平台内匹配依次使用：

1. ISRC
2. 艺人 + 完整曲名 + mix/version
3. 艺人 + 规范化曲名，并人工检查版本

禁止仅根据标题相似度自动选择不同录音。无法确认版本时归入未匹配，不冒险写入。

## 无连接器时的降级交付

提供可复制清单：

```markdown
1. Artist — Track (Mix/Version) — target-platform search/direct URL
```

说明缺少的是 OAuth 客户端、平台连接器还是当前会话授权。不要把“需要用户手动导入”表述成“一键导出成功”。
