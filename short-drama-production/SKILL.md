---
name: short-drama-production
description: 当用户提出短剧、动画、微电影、剧情视频、AI视频、电影、影视化、动态漫、宣传片、预告片等制作需求，或包含“做个短剧”、“弄个动画片”、“写个剧本”、“画个分镜”、“搞个人设/场景资产”、“出个关键帧”、“写图生视频提示词/Seedance提示词”等表达时调用。适用于需要按“规划-剧本-分镜-资产-关键帧-视频生成”推进完整视频生产流程的任何场景。
author: yzsp-ai
repository: https://github.com/yzsp-ai
---

# 短剧视频全流程
## 定位
本 skill 是短剧 / 剧情视频生成的主规划入口，负责按固定流程路由任务：
```text
planner -> scriptwriter -> storyboard -> assets -> frame -> prompt
```
`planner` 就是当前主 `SKILL.md`。其它阶段按需读取 refs：
| 阶段 | 读取文件 |
|---|---|
| scriptwriter | `references/scriptwriter.md` |
| storyboard | `references/storyboard.md` |
| assets | `references/assets.md` |
| frame | `references/frame.md` |
| prompt | `references/prompt.md` |
不得跳阶段。只要发现上游依赖缺失，就停在最早缺失阶段继续处理。

## 影视制作链路进入确认 + 默认锁定（新增功能，CRITICAL）
当本 skill 因短剧 / 动画 / 微电影 / 剧情视频 / AI 视频 / 电影 / 影视化 / 动态漫 / 宣传片 / 预告片 / 剧本 / 分镜 / 资产 / 关键帧 / Seedance 提示词等制作意图被加载时，agent **不得直接进入正式制作链路**，必须先用 `question` 弹窗询问用户是否进入本 skill 链路。禁止用普通文本开放题替代弹窗。

弹窗问题必须使用以下含义（可按上下文微调文案，但不得改变选项含义）：

```text
是否使用“影视制作”skill 链路制作，不要走线上通用制作链路？

选项 A：使用“影视制作”skill 链路制作（推荐）
选项 B：先不进入制作链路，只做咨询 / 讨论 / 方案建议
选项 C：不使用本 skill 链路，改用其它链路或由我另行说明
```

用户选择 A，或用户在当前消息中已经明确写出“使用我的 skill 制作链路 / 使用影视制作 skill / 使用‘影视制作’skill链路制作 / 按影视制作链路 / 按 short-drama-production 链路 / 不要走线上制作链路 / 保持我的 skill 链路”等同义表达时，视为已确认进入。本 skill 一旦进入正式制作流程，即**默认锁定**：

```text
production_chain_id: short-drama-production
production_chain_name: 影视制作
workflow_source: user_skill_locked
fallback_allowed: false
```

锁定后必须按本 skill 与其下游 drama workflow 的 STEP 顺序执行，禁止改走线上通用制作链路、禁止 light dispatch、禁止跳过已定义阶段、禁止用“优化 / 简化 / 压缩 / 线上流程更快”等理由绕开本 skill。若 agent 无法确认用户是否选择进入本链路，或找不到本链路要求的上游文档 / 资产 / 分镜 / 提示词来源，必须停下询问用户，不能自行 fallback 到其它制作链路。

用户选择 B 时，只回答咨询 / 讨论 / 能力说明 / 方案建议，不进入 planner -> scriptwriter -> storyboard -> assets -> frame -> prompt 制作链路，不生成 `[CHAIN_AUDIT]`，不冻结 `[FINAL_DISPATCH_PROMPT]`。用户选择 C 时，停止本 skill 链路，由用户后续指定其它链路或继续澄清。

## FINAL_DISPATCH_PROMPT 逐字派单锁（新增功能，CRITICAL）
正式派发任何图片或视频任务前，必须先在本 skill 链路内形成该任务已冻结的 `[FINAL_DISPATCH_PROMPT]`。一旦冻结，正式派单时必须逐字透传，禁止改写、压缩、翻译、摘要、普通段落转述、重新组织、合并多个任务、拆碎后让下游 agent 自行补全。

每个正式图片 / 视频任务的 `task_description` 必须使用以下格式包裹冻结提示词，不能以普通段落形式传递：

````markdown
[CHAIN_AUDIT]
production_chain_id: short-drama-production
production_chain_name: 影视制作
workflow_source: user_skill_locked
chain_step: <STEP 编号与名称>
upstream_documents: <制作计划 / 分镜列表 / 视频提示词主文档 / 资产登记表等来源>
source_unit: <资产名 / 分镜组 / Panel 编号>
required_assets: <本任务必须引用的角色卡 / 场景卡 / 道具卡 / 分镜图>
dispatch_mode: Verbatim prompt
fallback_allowed: false
[/CHAIN_AUDIT]

Verbatim prompt N (use as-is):
```text
(FINAL_DISPATCH_PROMPT 全文)
```
````

N 为同批任务序号（1/2/3/4）。同批多个任务必须分别写 `Verbatim prompt 1/2/3/4 (use as-is)`，每个代码块内只放该任务自己的冻结提示词全文。缺少 `CHAIN_AUDIT`、缺少 `Verbatim prompt N (use as-is)`、缺少 fenced code block、或代码块内容与 `[FINAL_DISPATCH_PROMPT]` 不一致时，禁止派发任务，必须回到对应 STEP 修正。

## 核心规则
- **制作阶段图片参数强制锁（CRITICAL）**：在本 skill 的制作阶段（分镜切分 / 资产设定 / 关键帧生成）派发任何 image agent 任务时，必须强制使用：`model: gpt-image-2`（即 Image 2）、`aspect_ratio: 16:9`、`resolution: 1k`、`quality: high`。这条规则优先级高于通用 `models.md` 中关于“内部素材默认 quality=medium”的路由建议；通用模型路由只能用于能力核对，不能覆盖本 skill 明确写死的制作参数。若派单参数不是上述四项，视为派单错误，必须停止并修正后重派。
- 在制作（分镜切分）（资产设定）（关键帧生成）默认使用模型：Image 2，比例：16:9，清晰度：1k，画质：高。
- 当用户要求生成视频时，不要立即开始生成，也不要立即调用视频生成工具，必须先澄清并确认以下基础参数：
 - 时长：视频长度，例如 5s、10s、15秒。
 - 当用户调用image_video且task_type=r2v，比例支持：'16:9','4:3','1:1','3:4','9:16','21:9'
 - 当用户调用image_video且task_type=f2v，比例仅支持保持原比例，如果用户要求调整，必须跟用户说明。
 - 生成详细内容：画面主体、动作、场景、风格、镜头运动、情绪氛围、文字、声音或其他限制。
 - 一次最多生成2个视频，超出必须对用户进行澄清：一次仅能生成两个视频。
 - **即时输出原则**：在资产设定（如角色设定图、场景图等）和视频生成阶段，**每生成完一张图或一个视频，必须立即将其展示并返回给用户**。绝对禁止等待所有图片或所有视频全部生成完毕后再统一打包返回。
 - **图片质量审查原则**：在完成任何一个阶段的图片生成之后，都应检查图片是否存在问题：
   - **人设图审查**：重点审查左侧面部特写是否是正脸，三视图是否与左侧面部特征保持一致，三视角是否是同一个人物。
   - **关键帧审查**：重点审查画面是否和对应剧情对的上，人物、场景、道具是否和资产设定图中的对应特征对的上。
 - **角色形象变化资产更新原则**：随着剧情进展，如果人物形象发生较大变化（如换装、发型变化、受伤/毁容、年龄阶段变化、身份伪装、盔甲/制服/礼服等关键造型变化），必须先重新生成对应阶段的角色资产设定图，并经用户确认后，才能继续生成后续关键帧或视频。禁止沿用旧资产图硬套新造型。
 - 只有在用户明确确认参数后，才可以继续生成。无论用户是否已经一次性提供完整参数，都必须先输出一次参数摘要并等待确认；不能因为信息看似完整就直接进入生成。明确确认可以是 确认、可以生成、开始生成、没问题、yes、looks good、ok等表达。
## 澄清门禁
在剧情设计、剧本设定和视频生成开始前，必须澄清会显著影响结果的关键参数。每轮只问 1-3 个问题。用户说“你来定”“随便”“直接做”等表达时，可以内部选择合理默认值并继续推进非生成阶段。
真正生成视频前，必须确认以下基础参数：
- 时长：例如 5s、10s、15s，或每个片段的目标时长。
- 比例：例如 `16:9`、`4:3`、`1:1`、`3:4`、`9:16`、`21:9`。
- 生成模式：文生视频、图生视频、参考视频生成、首帧驱动视频，或工具侧的 `r2v` / `f2v` 等模式。
- 画面详细内容：主体、动作、场景、风格、镜头运动、情绪氛围、可见文字、台词、声音和限制。
- 生成数量：单批最多 2 个视频，超出必须对用户进行澄清，一次仅能生成两个视频。
- 确认表达：只有用户明确回复“确认”“可以生成”“开始生成”“没问题”“yes”“looks good”等，才可以继续生成。
### 分阶段澄清
| 阶段 | 需要澄清 |
|---|---|
| planner | 目标、产物形式、受众、全流程范围、已有素材、希望从哪个阶段开始。 |
| scriptwriter | 故事主题、题材、目标时长或集数、主角、核心冲突、结局方向、情绪基调、禁用内容。剧情设计和剧本设定缺关键参数时必须先澄清。 |
| storyboard | 要处理的集数、单集时长、片段结构、镜头密度、台词/字幕/声音要求。 |
| assets | 角色、场景、道具、视觉风格、参考图用途、一致性锚点、音色或台词锚点。 |
| frame | 哪一集/哪些片段、首尾关键帧要求、场景引用、角色引用、构图、光影、角色物理状态连续性。 |
| prompt | 工具模式、时长、比例、首帧、参考资产、时间戳动作规划、台词/声音、负向约束、批次数量。 |
如果某个缺失值只影响最终生成、不影响规划，可以继续规划，并把它标为生成前确认点。
## 阶段判断
按顺序检查，并停在第一个缺失依赖：
1. 只有 idea、标题、简短梗概、不完整故事或松散角色概念时，开始剧本创作。
2. 有完整故事但没有按集/片段/镜头切分时，开始分镜切分。
3. 有分镜但没有可用角色、场景、道具资产引用或详细视觉基准时，开始资产设定。
4. 有资产和分镜但没有片段关键帧图时，开始关键帧生成。
5. 有关键帧但没有最终视频提示词执行包时，开始视频提示词生成并生成对应视频。
## 规划输出
启动或接续项目时，先输出简短规划报告，再读取下一阶段 ref：
```markdown
## 短剧生产规划报告
### 项目信息
- 项目名称：
- 类型：
- 风格：
- 默认比例：
- 默认时长：
### 当前完成度检查
| 阶段 | 要求 | 是否完成 | 缺失内容 |
|---|---|---|---|
| 1. 剧本创作 | 完整故事梗概、角色、人设、关键场景 | 是/否 | |
| 2. 分镜切分 | 每集/片段/镜头规划 | 是/否 | |
| 3. 资产设定 | 角色、场景、道具资产与参考图 | 是/否 | |
| 4. 关键帧 | 片段首尾关键帧图 | 是/否 | |
| 5. 视频提示词 | 可执行视频 prompt 与生成参数 | 是/否 | |
### 执行规划
当前阶段：
后续流程：
需要确认：
```
## 路由
- 当前阶段是剧本创作时，读取 `references/scriptwriter.md`。
- 当前阶段是分镜切分时，读取 `references/storyboard.md`。
- 当前阶段是资产设定时，读取 `references/assets.md`。
- 当前阶段是关键帧生成时，读取 `references/frame.md`。
- 当前阶段是最终视频生成时，读取 `references/prompt.md`。
**每阶段完成后都必须用户确认：**
在剧本、分镜、资产、关键帧的任何一个阶段完成输出后，**必须停下来，并明确询问用户是否满意、是否需要修改**。绝对禁止在没有获得用户明确的“确认/没问题/继续/yes/ok”等许可下，擅自读取下一个 Reference 文件并自动推进到下一阶段。
## 本地模板源强制锁 + 制作前 QC（新增功能，CRITICAL）
当用户要求使用“影视制作”skill 链路，或当前项目已经锁定 `production_chain_id: short-drama-production` 时，agent 在**每次制作任何产物之前**都必须执行本节。这里的“任何产物”包括但不限于：剧本、分镜、资产设定图、关键帧、视频提示词、视频生成任务、阶段更新文档。

### 1. 主 SKILL.md 强制读取
每次开始或继续制作前，必须重新读取当前主 skill 文件：

```text
C:\Users\王硕\Movies\Hub\skills\short-drama-production\SKILL.md
```

读取后必须确认当前链路仍为：

```text
production_chain_id: short-drama-production
production_chain_name: 影视制作
workflow_source: user_skill_locked
fallback_allowed: false
```

禁止只凭上下文记忆、旧摘要、历史执行记录或线上 agent 默认规则继续制作。若无法证明已读取当前主 `SKILL.md`，必须停止制作并先读取。

### 2. 阶段 reference 强制读取
主 skill 读取后，必须根据当前阶段读取本 skill 目录下对应的 reference 文件，且该文件必须作为当前阶段唯一模板来源：

| 当前阶段 | 必须读取并作为模板源的文件 |
|---|---|
| 剧本创作 | `C:\Users\王硕\Movies\Hub\skills\short-drama-production\references\scriptwriter.md` |
| 分镜切分 | `C:\Users\王硕\Movies\Hub\skills\short-drama-production\references\storyboard.md` |
| 资产设定 | `C:\Users\王硕\Movies\Hub\skills\short-drama-production\references\assets.md` |
| 关键帧生成 | `C:\Users\王硕\Movies\Hub\skills\short-drama-production\references\frame.md` |
| 最终视频生成 | `C:\Users\王硕\Movies\Hub\skills\short-drama-production\references\prompt.md` |

如果当前阶段无法判断，必须停下询问用户，不能自行套用其它 workflow。

### 3. `template_source` 锁
每次制作前必须在内部确认：

```text
template_source = C:\Users\王硕\Movies\Hub\skills\short-drama-production\references\<当前阶段>.md
```

当前阶段产物的结构、字段、命名、提示词模板、时间戳格式、镜头格式、资产格式、关键帧格式或视频提示词格式，必须来自这个 `template_source`。禁止只写链路锁字段但实际套用其它模板。

### 4. 产物结构 QC
制作前必须把将要产出的内容与 `template_source` 内的模板进行结构核对，核对通过后才能制作。核对必须覆盖：

- 当前产物是否属于当前阶段；
- 当前产物的字段是否来自对应 reference；
- 当前产物的命名与阶段要求是否一致；
- 当前产物是否缺少 reference 要求的必填项；
- 当前产物是否混入其它 workflow 的字段、术语或模板。

例如：当前阶段是“分镜切分”时，必须按 `references/storyboard.md` 核对：

- 是否按片段拆分；
- 是否包含绝对时间戳；
- 是否每个镜头包含景别、视角类型、机位状态、角度与朝向、画面动作、台词、旁白、音效、音乐、字幕、转场中适用的信息；
- 是否符合 `references/storyboard.md` 中的输出格式。

如果产物出现 `references/storyboard.md` 中没有要求的外部模板结构，例如“STEP 5b”“grid-template”“黑白八宫格模板”“Seedance shot group 模板”“站位记忆表模板”等，必须立即停止，回退到本 skill 的对应 reference 重新制作，除非用户明确授权额外使用这些外部模板。

### 5. 禁止外部模板源混用
当本 skill 链路已锁定时，禁止将以下内容作为当前阶段模板源：

```text
C:\Users\王硕\.hub\.config\knowledge\workflows\drama\...
C:\Users\王硕\.hub\.config\knowledge\references\directing\...
grid-template.md
step-5b-storyboard-grid.md
Seedance shot group / 通用 drama workflow / 线上 agent 默认分镜模板
```

这些文件最多只能在用户明确授权时作为“补充参考”，不得替代本 skill 的 `references/*.md`。如果发现已经读取或套用了非本 skill 模板，必须立即声明错误、停止当前制作、回退到本 skill 对应 reference 重新制作。

### 6. 制作前 QC 标记
每次进入正式制作前，agent 必须在内部完成以下 QC，缺一不可：

```text
[影视制作链路执行前 QC]
1. 已读取主 SKILL.md。
2. 已判定当前阶段。
3. 已读取当前阶段 reference。
4. template_source 指向本 skill 的 references/<当前阶段>.md。
5. 当前产物结构与 template_source 模板一致。
6. 未使用线上通用 workflow / drama workflow / grid-template / 非本 skill 模板作为当前阶段模板源。
7. production_chain_id = short-drama-production。
8. fallback_allowed = false。
```

QC 不通过时禁止制作，必须先修正。不得以“更专业”“更完整”“更快”“线上模板更强”“为了优化效果”等理由绕过本节。

## 工具门禁
以宿主环境实际可用工具为准。旧版工具名按下表理解：
| 旧工具名 | 当前含义 |
|---|---|
| `text_to_image` | 文生图 |
| `image_to_image` | 基于参考图的图像编辑 / 图像生成 |
| `image_to_video` | 基于图片参考的视频生成 |
**工具调用失败处理：**
当阶段规则要求使用 `image_to_image` / I2I 或 `image_to_video` / I2V 时，如果实际调用失败，必须优先重新尝试调用对应工具，仍失败再停下来说明失败原因与所需补充信息。禁止为了绕过失败而擅自降级改用 `text_to_image` / T2I 或纯 `text_to_video` / T2V，这会丢失参考图约束并破坏角色、场景、道具一致性。
## 完成检查
声明完成前必须确认：
- 没有跳过上游阶段。
- 视频生成前已完成必要澄清并获得明确确认。
- 比例、时长、模式、数量、参考素材和限制已记录。
- 角色、场景、道具和关键帧连续性已保持。
- 剧情导致人物形象明显变化时，已补充并确认新的角色资产设定图。
- 应使用 I2I / I2V 的任务没有因工具调用失败而降级为 T2I / T2V；如失败，已重试对应工具或明确说明失败原因。
- 单批生成视频数量不超过 2 个。
- 资产和视频的输出必须遵循“即时输出”原则，生成一个返回一个，不得囤积。
- 所有视频生成结果，无论 `text_to_video` / `image_to_video`、UGC / marketing / corporate，交付前都必须调用 `notify_hunman` 整理输出并展示视频；不得以“非飞书内容”“在线 URL 可直接回复”“链接可展示”“Markdown/HTML video 标签可播放”“用户偏好直接交付”等理由跳过。
- `notify_hunman` 入参必须包含可渲染的视频资源本体：`video_url/output_url/url/result_url` 或本地视频路径；不得只传总结文案、HTML/Markdown `<video>` 片段或裸链接文本。
- 只有确认 `notify_hunman` 已正常上屏视频后，才可以输出 `视频已生成` 或成功格式；如未正常上屏，必须用同一视频资源重试一次，仍失败再说明展示失败并保留链接/路径排查。
