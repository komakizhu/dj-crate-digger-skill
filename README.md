# Crate Digger Skill

面向 DJ、音乐制作人与电子音乐听众的 Codex Skill：把自然语言 Digging 需求转成经过联网搜索、逐曲验证、去重、排序和播放顺序设计的 Markdown 报告。

## 本版重点

- 中文 DJ 口语与场景语义归一化，区分参考艺人与专题目标。
- SoundCloud → Apple Music → Spotify 默认搜索顺序；用户首选、允许和禁用平台可覆盖默认值。
- 每首最终曲目逐条打开直接页面核验；未知 BPM、流派、发行时间和流行度不补造。
- 四个版本共享同一验证候选池；综合版采用 Warm Up → Groove → Peak → Closing，并跨段去重。
- 动态内容权重只在风格、场景、流行度之间分配；平台顺序不混入内容排序。
- 目标平台缺曲时按 preferred / exclusive 分层降级，不自动替代、不越权导出。

## 目录

- `SKILL.md`：主工作流与安全边界
- `references/`：搜索验证、动态排序、报告模板和平台导出规则
- `evals/evals.json`：10 个行为评测，覆盖中文需求、平台优先级、逐曲核验、四版本、去重、播放顺序和降级
- `evals/trigger-evals.json`：触发与反触发样例
