# 搜索、验证与去重

## 搜索策略

把需求拆成多个查询，不依赖一次宽泛搜索。至少覆盖：

1. 核心风格 + 参考艺人或参考曲目
2. 场景 + mood + energy
3. 平台 + 风格 + 年代或 freshness
4. 相邻子流派或同厂牌线索
5. 用户明确的热门、冷门或熟悉度要求

用户未指定平台时，按 SoundCloud → Apple Music → Spotify 的顺序查找，并用 Bandcamp、Beatport、Discogs、MusicBrainz、艺人官网和厂牌官网补充验证。用户指定平台时，目标平台中的可用性优先于跨平台覆盖。

推荐类信息可能变化，使用联网工具获取当前结果。优先并行搜索不同查询，但最终逐条核验候选。

默认平台顺序要落实到实际检索，不只是写在报告里：先做 SoundCloud 的发现与逐曲核验，再做 Apple Music，再做 Spotify，最后才用 Bandcamp、Beatport、Discogs、MusicBrainz、艺人官网或厂牌官网补元数据。用户给出平台顺序时替换这三个平台阶段。候选在前一阶段已有可访问、版本吻合的直接页时，不要无理由把后面平台或权威发行页设为主链接。

### 平台约束的执行顺序

在发出查询前，把平台分成三类：

1. `priority`：用户给出的首选顺序；没有时才使用默认顺序。
2. `allowed`：可以作为最终链接或补充来源的平台。
3. `forbidden`：不要搜索、不要引用、不要放进“来源”段落的平台。

“优先/最好有”只改变链接顺序和回退顺序；“只要/仅接受”则要求最终曲目必须有该平台直接页面。不要因为某平台更容易找到结果，就把它偷偷升级为用户的唯一目标。候选记录的 `primary_source` 应是第一顺序中已验证的允许平台页；Bandcamp、Beatport 等权威页放入补充证据，只有没有任何允许平台直接页时才可作为非 exclusive 报告的主证据。

### 中文 DJ 需求的检索展开

保留用户原词，同时为搜索生成等价的中英文线索。例如：`暖场/开场/收场/推峰` 可展开为 warm-up、opening、closing、energy arc；`左场/地下/怪` 可展开为 leftfield、underground、experimental；`大家熟/有反应/不俗` 可展开为 familiar、crowd response、crossover、avoid overplayed。展开词只用于提高召回，不把搜索词当作已验证的流派或音频事实。

艺人、歌曲和场景词要分开查询：如果用户说“像 X”或“把 X 当采样语汇”，用 X 发现相邻作品，不把 X 本身的全部目录当成必选结果。

## 来源等级

### A：可直接用于曲目验证

- SoundCloud、Apple Music、Spotify 的官方曲目页面
- Bandcamp、Beatport 的具体发行或曲目页面
- 艺人或唱片公司的官方发行页面

媒体评测、采访、Wikipedia 或普通专辑介绍可以补充背景，但不能单独充当最终曲目的直接播放/发行链接；如果它只列出整张专辑而没有把该曲与发行准确对应，也不能替代逐曲核验。一个通用评测 URL 不能被复制到多首曲目上冒充逐曲证据。

### B：可补充元数据

- MusicBrainz
- Discogs
- 权威音乐媒体或厂牌目录

### C：仅作为发现线索

- 普通搜索摘要
- 用户生成列表、论坛、社交帖子
- 聚合站结果

C 级来源不能单独证明曲目存在、版本正确或元数据准确。不要引用搜索结果页作为最终播放链接。

## 候选记录

为每首候选维护以下字段；无法验证的字段保留 `unknown`：

```yaml
title: official title
artist: official artist name
version: original | remix | edit | live | unknown
platforms:
  soundcloud: url | null
  apple_music: url | null
  spotify: url | null
primary_source: url
source_platform: soundcloud | apple_music | spotify | other
platform_match: preferred | allowed_fallback | exclusive | unavailable
isrc: value | unknown
genres: []
bpm: number | unknown
energy: low | medium | high | unknown
moods: []
release_date: value | unknown
popularity_evidence: description | unknown
verification: verified | partial | unknown
```

在内部候选表中保留每首的直接页面 URL、实际查看过的页面字段、`platform_match`、`verification` 和去重键。最终四个版本只从这张表生成；如果一首歌没有可追溯的记录，就不要凭记忆把它补进报告。

## 验证门槛

进入最终歌单前，曲目必须同时满足：

- 曲目页面或权威发行页面可访问
- 页面中的艺人和曲名与候选一致
- 版本信息没有把 Remix、Edit、Live 与 Original 混淆
- 至少一个直接来源链接可提供给用户

验证动作要逐曲完成：打开最终要给用户的直接页面，核对页面中的艺人、完整曲名和 mix/version；仅看到搜索摘要、URL slug 或聚合站条目不算通过。链接失效或页面信息冲突时剔除候选，或在证据足够但非 exclusive 平台缺失时标记 `partial`。如果平台为 exclusive，缺少该平台直接页的候选不能进入最终曲目表，只能进入缺失清单。最终报告中的每一行都要有自己可追溯的曲目/发行证据，不能只靠一条泛化媒体页面背书。

## 去重规则

1. 相同 ISRC 视为同一录音。
2. 无 ISRC 时，规范化大小写、Unicode 标点、feat./featuring、大小写混写和空格后比较艺人 + 曲名 + 版本；不要删除 Remix、Dub、Edit、Live 等版本词再去重。
3. 同一录音在多个平台出现时合并平台链接，不重复占位。
4. Remix、Edit、Dub、Instrumental、Live、Radio Edit 与 Extended Mix 保留为独立版本，但避免同一版本的拼写差异造成重复。
5. 同一艺人连续占据过多位置时，在排序阶段处理，不要在去重阶段误删有效曲目。

去重发生在四种排序和播放编排之前。综合版在分配 Warm Up、Groove、Peak、Closing 后再做一次全局录音键检查，确保同一录音不会跨段重复；多个平台链接仍合并在同一条记录中。

## 搜索结束条件

达到以下任一条件即可停止扩展候选池：

- 已有足够多的高相关、已验证候选支持四个版本
- 新查询连续只产生重复或低相关结果
- 平台限制使继续搜索无法提高验证质量

候选不足时透明报告，不降低验证门槛。
