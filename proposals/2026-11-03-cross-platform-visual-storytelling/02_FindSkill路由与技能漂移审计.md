# Find Skill × 视觉叙事｜能力发现与安全执行（Draft）

## 1. 结论

**Find Skill 是“先找到可能适合的技能，再决定是否可信并可调用”的发现动作，不是一个默认已经安装、能做所有动图的引擎。**

已核对公开技能仓 README，确实提到“find-skills 可检索”，但这次未能在 `xingtu-skills/skills/find-skills/SKILL.md` 找到同名本地文件。**不可据此推断用户本机、Work、Codex、Muse 是否已安装某个第三方 find-skills，也不可虚构其执行结果。**

## 2. 先复用已有能力（已读取/核对）

| 已有组件 | 能做什么 | 当前问题/边界 | 验证入口 |
|---|---|---|---|
| `concept-anchor-figure` | HTML→PNG 概念锚点图，可编辑文字，须看图验收 | 文档内仍含旧红、行途蓝，与 design 新提案冲突；不等于 GIF | [SKILL.md](https://github.com/xingtu1996/xingtu-skills/blob/main/skills/concept-anchor-figure/SKILL.md) |
| `pub-infographic-supplement` | 正文信息图补充 | 旧图片密度口径须复核；不应为凑数生成 | [Skill 目录](https://github.com/xingtu1996/xingtu-skills/tree/main/skills/pub-infographic-supplement) |
| `wechat-cover-gen` | 公众号封面生产 | “严格黑白交替”和文字主标要求应对齐 PR #2 | [Skill 目录](https://github.com/xingtu1996/xingtu-skills/tree/main/skills/wechat-cover-gen) |
| `xingtu-multiplatform-distribution` | 图卡、富文本、X Thread 和发布操作单 | 已有一稿多平台能力；但发布时序/发布行为需人工把关 | [SKILL.md](https://github.com/xingtu1996/xingtu-skills/blob/main/skills/xingtu-multiplatform-distribution/SKILL.md) |
| `PUBLISHING-PIPELINE.md` | 端到端发布技能路由与硬门禁 | 仍强制交替、图片密度 8–12 张；与设计提案不一致 | [发布总线](https://github.com/xingtu1996/xingtu-skills/blob/main/PUBLISHING-PIPELINE.md) |

上述是**文档可见能力**，不是已在当前机器实际运行成功。

## 3. 动态分镜任务的 Find→Assess→Run 路由

```text
输入：已核验事实/文章母稿 + 目标平台 + 视觉目标
  │
  ├─ A 事实/授权/敏感数据预检（失败即阻断）
  ├─ B 从 xingtu-skills 现有仓 + Work 实际已安装能力检索
  │    └─ 若另用 find-skills 目录/CLI：先核实名称、来源、版本与权限
  ├─ C 候选能力评级：KEEP(已审可用) / PILOT(沙箱待测) / REJECT(高风险)
  ├─ D 决定制品：SVG/HTML 源图 → 静态分镜 PNG
  │    └─ 只有存在**实际通过沙箱验证**的渲染能力和平台需求才增产 GIF/MP4
  ├─ E 真实设备/导出校验：字、流向、帧顺序、比例、体积、静态替代
  └─ F 批准后入发布包；交付源文件/哈希/回执/回退路径
```

**权限红线**：候选 Skill 可能包含 shell、网络、文件修改、安装依赖、OAuth 和发布动作。第三方 Skill 必须审查源码及间接依赖，在隔离环境验证；禁止因为“find”找到它就自动安装或执行。不把私有发布包、截图、token 交给未知 Skill。安装与高权限操作按现行人工授权策略。

## 4. 最小实验设计（Work 实际执行，当前 NOT_RUN）

- T1 **静态可读**：相同 Router 事件故事拆 4–5 张 PNG；375/414px 上图中文字无溢出、路径方向明确、静态单帧可理解。
- T2 **动态一致**：在工具确实支持且得到授权时，使用**同一结构化源**导出 GIF/视频；逐帧对照静态分镜，没有跳帧改变事实/唯一信息仅在动画中。
- T3 **跨平台与回退**：至少出公众号纵向图、小红书 3:4 卡、X 单图，检查裁切、图注/替代文本、体积与加载；动画不兼容时切换静态图，不阻塞文章可读性。
- T4 **能力安全**：记录 Skill 源 URL/commit、安装范围、权限要求、沙箱运行命令与退出码、产生文件 SHA，以及未通过项。
- T5 **文档漂移**：diff design 正本候选与 xingtu-skills、harness-kit 视觉镜像，发现旧口径建立独立 PR；不默默覆盖。

## 5. 能力选择原则

- 若目的是“单张一句话看懂”：先用 `concept-anchor-figure`，但要人工覆盖旧配色并记录尚未修复。
- 若目的是“数据对比”：使用可追溯数据的 HTML/SVG 图表，不走无依据的 AI 生成曲线。
- 若目的是“真实截图”：走 Design PR #3，不能用 AI 模拟官方产品。
- 若目的是“状态迁移”：**先静态连续分镜**；GIF/MP4 是可选增强，不是默认交付。
- 若目的是“多平台复用”：调用 `xingtu-multiplatform-distribution` 生成变体计划；它不授予自动群发权限。

**验证状态**：仓库读取=已验证；用户设备具体安装、动态图渲染、微信公众号媒体兼容、发布结果=NOT_RUN。
