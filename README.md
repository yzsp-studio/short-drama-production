<div align="center">

# 🎬 short-drama-production

## 派单提示词逐字透传，谁也不许改写压缩
## Verbatim dispatch or no dispatch: frozen prompts pass through untouched

📮 **商务合作 / Business：wow.vip@foxmail.com**
🎬 **抖音 / Douyin：[v.douyin.com/07EjIDGVErY](https://v.douyin.com/07EjIDGVErY/)**

</div>

---

## 🇨🇳 中文

短剧、动画、微电影、剧情视频、AI 视频、电影、影视化、动态漫、宣传片与预告片的制作主规划入口，按固定链路 planner → scriptwriter → storyboard → assets → frame → prompt 路由任务，不得跳阶段；一旦发现上游依赖缺失，就停在最早缺失阶段继续处理。技能被加载后不能直接进入制作，必须先弹窗确认是否进入该影视制作链路，进入即默认锁定，禁止改走通用制作链路、禁止简化或压缩已定义阶段。正式派发任何图片或视频任务前，必须在本链路内形成并冻结 [FINAL_DISPATCH_PROMPT]，派单时逐字透传，并附 [CHAIN_AUDIT] 审计块标注阶段、上游文档、来源单元、必需资产与派单模式。

### ✨ 能做什么

- 六大阶段固定路由：planner → scriptwriter → storyboard → assets → frame → prompt，缺上游就停在最早缺失阶段
- 加载后先弹窗确认是否进入影视制作链路，进入即锁定，禁止 fallback 到通用制作链路
- 冻结 [FINAL_DISPATCH_PROMPT]，正式派单逐字透传，禁止改写、压缩、翻译、摘要或合并任务
- 每个任务附 [CHAIN_AUDIT]：阶段、上游文档、source_unit、必需资产、dispatch_mode 与 fallback_allowed
- 制作阶段图片参数强制锁：gpt-image-2、16:9、1k、high，参数不符即视为派单错误并停止重派
- 生成视频前先澄清时长、画幅等基础参数，确认后才调用视频生成工具

### 📦 包含的技能

- `short-drama-production` —— 见 [`short-drama-production/SKILL.md`](short-drama-production/SKILL.md)

### 🚀 安装

```bash
git clone https://github.com/yzsp-studio/short-drama-production.git
cp -r short-drama-production/* "$DSH_HOME/skills/"
```

> 技能是**纯文本说明**（`SKILL.md` + `references/`），零依赖、零 API key，用你当前 agent 的会话额度。

---

## 🇬🇧 English

The main planning entry point for short dramas, animation, micro-films, narrative videos, AI video, films, dynamic comics, promos and trailers. It routes tasks through a fixed chain — planner → scriptwriter → storyboard → assets → frame → prompt — and no stage may be skipped; when an upstream dependency is missing, work stops at the earliest missing stage and continues there. Once loaded it may not enter production directly: a confirmation dialog first asks whether to use this film-production chain, and entering locks it, forbidding fallback to a generic chain and forbidding simplification or compression of defined stages. Before any image or video task is dispatched, a [FINAL_DISPATCH_PROMPT] must be formed and frozen inside the chain, then passed through verbatim, wrapped in a [CHAIN_AUDIT] block naming the stage, upstream documents, source unit, required assets and dispatch mode.

### ✨ What it does

- Fixed six-stage routing — planner → scriptwriter → storyboard → assets → frame → prompt — stopping at the earliest missing upstream stage
- Confirmation dialog on load asks whether to use the film-production chain; entering locks it and blocks fallback to a generic chain
- Freezes [FINAL_DISPATCH_PROMPT] and passes it through verbatim, forbidding rewriting, compression, translation, summary or task merging
- Attaches a [CHAIN_AUDIT] block per task: stage, upstream documents, source_unit, required assets, dispatch_mode and fallback_allowed
- Locks production-stage image parameters to gpt-image-2, 16:9, 1k, high; any mismatch is a dispatch error that must stop and be re-sent
- Clarifies runtime, aspect ratio and other base parameters before any video generation tool is called

### 📦 Skills included

- `short-drama-production` — [`short-drama-production/SKILL.md`](short-drama-production/SKILL.md)

### 🚀 Install

```bash
git clone https://github.com/yzsp-studio/short-drama-production.git
cp -r short-drama-production/* "$DSH_HOME/skills/"
```

> Skills are **plain-text instructions** (`SKILL.md` + `references/`). Zero dependencies, no API keys — they run on your agent's existing session quota.

---

## 📮 联系 / Contact

| | |
|---|---|
| 🎬 **抖音 / Douyin** | [https://v.douyin.com/07EjIDGVErY/](https://v.douyin.com/07EjIDGVErY/) |
| 📮 **商务合作 / Business** | **wow.vip@foxmail.com** |
| 🐙 **GitHub** | [https://github.com/yzsp-ai](https://github.com/yzsp-ai) |
| 💖 **打赏 / Support** | [afdian.com/a/yzsp-ai](https://afdian.com/a/yzsp-ai) |

## 📄 License

[MIT](LICENSE) · © 2026 [yzsp-ai](https://github.com/yzsp-ai)
