# System Design｜行途视觉资产编译管线 v0.2（Draft）

> **文档类型**：System Design / SDD 候选。**正式规则仍以 main 及已批准发布 SOP 为准**。本文件位于 Design PR #4，不自动修改正在运行的 Skills / 发布脚本。版本：2026-11-04 设计候选 v0.2。
>
> 入口：[本提案目录](https://github.com/xingtu1996/xingtu-design/pull/4) · [对应编排 Skill PR #10](https://github.com/xingtu1996/xingtu-skills/pull/10) · [既有公众号规范 PR #2](https://github.com/xingtu1996/xingtu-design/pull/2) · [截图治理 PR #3](https://github.com/xingtu1996/xingtu-design/pull/3)。

## A. 问题 / 成功定义（Why）

现状：已有封面、信息图、分发 Skills，但各自对品牌色、黑白交替、配图密度有历史口径；只用方框轮流高亮的 GIF 虽然“会动”，不等于读者理解了系统。Chat 的设计裁决和 Work 的实际发稿包也可能版本漂移。

目标：**一次经核验的文章母稿，得到可回溯、可修改、适合每个媒介的视觉资产**；其中动画必须增加对**状态变化、因果、并行或异常处理**的理解。反目标：不追求所有文章都有动图、不建立新编排器、不复制外部公众号的图像、不自动群发。

成功标准是读者能根据画面回答“什么触发、谁决策、发生何种变化、失败后如何回退”，且图与正文不冲突；美观评价只能在事实、隐私与可读性检查通过后进行。

## B. 需求与约束

| ID | 必须满足 | 可验证证据 |
|---|---|---|
| R1 | 母稿事实、设计、出图、平台版本可追溯 | article/claim/asset/commit ID 映射 |
| R2 | 同一内容可生成微信正文插图/小红书图卡/X 主图/朋友圈单图 | 每个平台有独立输出及语义说明 |
| R3 | 静态分镜是默认基线；动画是有理由才启用的增强 | 动效选择理由、首帧/末帧/静态降级件 |
| R4 | 精确汉字、数字、路径保存在可编辑 SVG/HTML/数据层 | 可修改源 + 导出文件 + 逐帧复核 |
| R5 | 出图不会伪造官方截图、实验数字或私有业务场景 | claim→source、representation、permission |
| R6 | 任何未知外部 Skill 先审查来源和权限，后隔离试运行 | 源 URL、commit、依赖、权限、沙箱记录 |
| R7 | 版本冲突可发现且可回滚 | design/skill/pipeline 版本和旧新规则差异 |
| R8 | 发布工作接力不重复建稿/覆盖已有草稿 | current article、draft 状态、人工审批回执 |

非功能约束：普通手机上可读、颜色克制、中文字形准确、源文件可维护、失败可回退、成本和体积受控、运行日志不可泄露私有路径或 token。**不要把每张图都强制设置同样的帧数、像素尺寸、时长；根据内容和平台作实测决定。**

## C. 组件与权威关系

```text
Chat / 用户意图（决策）                当前已有 Work / Codex / Muse（执行宿主）
       │                                            │
       └───────────────┬────────────────────────────┘
                       ↓
         xingtu-vault / 真实母稿、manifest、草稿状态（私有）
                       ↓
        FACT GATE：论点/来源/授权/脱敏/现有版本
                       ↓
    xingtu-design / 设计规范正本 + System Design + Prompt 契约
                       ↓ 显式 version/commit 引用
    xingtu-skills / xingtu-visual-storytelling（编排 Skill）
                       ↓ 按目标查能力、审权限
  已有出图 Skill / SVG·HTML·Remotion / 外部候选（只沙箱）
                       ↓
       可编辑源 → PNG 静态帧 → 可选 GIF/MP4 → 渠道变体
                       ↓
   QUALITY GATE：事实、视觉、隐私、许可、手机预览、哈希
                       ↓
       Vault 发布包 → 私人草稿（仅已有授权）→ 人工公开终审
                       ↓
         发布结果/读者反馈 → 质量记录 → Design 变更 Issue
                       └─────→ 经审核升版，再同步 Skills 和镜像
```

**规范 SSoT**：`xingtu-design`；**执行契约/技能路由**：`xingtu-skills`；**具体文章与生产产物**：私有 `xingtu-vault`；**运行时镜像**：`harness-kit/docs/brand-visual.md`（派生，不是独立规范）。`xingtu-learn` 可存研究原理但不发布第二份设计规则。跨仓引用必须显式指向仓库、路径、版本/commit；GitHub 链接不等于已安装/同步。

## D. AI 可读接口 / 输入输出

最小输入：`article_ref`（PR/本地包）、`article_revision`、`target_platforms`、`permissions`、`design_revision`、`skill_revision`、`claim_sources`。源数据只在授权环境读取。

对应的机器可读结构：[`visual-asset-contract.schema.json`](./visual-asset-contract.schema.json) 和 [`visual-plan.example.json`](./visual-plan.example.json)。
- `claim_id`：一条需要图像解释的事实/假设，注明证据等级与原始来源。
- `asset_id`：一张图或分镜，引用至少一个 `claim_id`；`representation` 区分真实截图/原创示意/AI 概念图/数据图。
- `design_revision`、`skill_revision`、`source_revision`：防上下文与规范漂移。
- `render_status` 与 `review_status`：只能记录实际运行结果；未跑就是 `NOT_RUN`。
- `public_release_approved=false` 是默认值；不代表私人草稿的创建授权。

发布包建议沿用既有 `01_正文/`、`02_排版HTML/`、`03_封面/`、`04_配图/`、`06_发布参考/`，把本契约实例保存在**私有包内**而非公开设计仓。允许资产按目标平台建立变体，但 source→claim→asset 的映射不能丢失。

## E. 制作选择与动效策略（专业裁决）

优先级：**证据截图（有权且有信息） > 清晰静态图 > 连续静态分镜 > 有必要的动态图**，不是技术越炫越高级。动图制作拆三职责：

1. **Storyboard（讲什么）**：一个真实场景，先触发、再路由分支、执行并行/异常，最后给回执；每帧仅一个认知目标。
2. **Motion Direction（怎样动）**：节点/连线/数据包有明确时序、真实层级与缓动；强调当前活动，其余降低对比。拒绝闪烁、乱飞装饰、无意义的镜头推进。
3. **Render & Delivery（如何交付）**：同一源生成 SVG/HTML、静态 PNG、可选 GIF/MP4；实测不同平台时长、尺寸、中文字形、加载及循环行为。

候选能力详见 [`07_动态工具评估与质量基线.md`](./07_动态工具评估与质量基线.md)。工具存在或 Skill 文档可读≠本机安装或平台验收成功。

无障碍参考：[W3C WCAG 2.2 / 2.2.2 Pause, Stop, Hide](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide)。对于自动开始并与正文并行、持续超过 5 秒的运动内容，应考虑暂停/停止控制；微信 GIF 可能不能提供控件，优先使用有限循环/静态替代并实机核对。此条是**设计参考，不等于微信平台认证或通过 WCAG 审核**。

## F. 状态机、失败回退与权限

```text
INPUT_VERIFIED → STORYBOARD_REVIEWED → SOURCE_RENDERED
      → ASSET_QA_PASS → PLATFORM_QA_PASS → PACKAGE_READY
      → PRIVATE_DRAFT_VERIFIED [可选：需要已授权] → HUMAN_RELEASED [人工]
              ↘ BLOCKED / NOT_RUN / ROLLED_BACK
```

- 不存在可靠来源 → 不生成“实测”数据图；可仅生成标注为示意的分镜。
- 中文渲染错误/事实漂移/隐私或版权红线 → BLOCKED，修正或撤图。
- GIF 失真、文件过大、手机播放不可靠 → 采用已审 PNG 静态分镜，不影响正文可读性。
- 当前文章已有私人草稿 → 优先只读回查，不因设计样稿更新而自动覆盖。
- 不从本文件授予任何安装权限、第三方网络权限、公众号建稿权限或公开发布权限。

## G. 验收、监控与演进

1. 先走可审查的 T1–T8 及新增 T9–T12（详见 Skill `evals.md`），**测试结果当前为 NOT_RUN**。
2. 最小一次端到端演示：一篇经脱敏的技术文章、静态分镜 4–5 帧、可选一段动画、公众号 375/414px 预览和小红书 3:4 转换；有真实 `sha256`、导出回读、审核人记录。实际公开发布不是该实验的必要条件。
3. 每篇文章收集：主张错误/错字/溢出、动画失效、素材许可阻断、退回次数、修改成本、读者能否复述流程等；不把主观评分冒充统计显著性。
4. 改进流程与升版机制见 [`06_持续迭代_验收与版本治理.md`](./06_持续迭代_验收与版本治理.md)。PR 审核后才可把设计变更传播到技能/模板/镜像。

## H. 开发/执行者交接

- **Chat**：证据裁决、信息结构、质量门槛；不伪造实测。
- **Work / Codex**：只在持有权限的环境读取真实发布包；以 System Design + Prompt 为输入产生修改与验收，返回 commit/文件/状态；不重复搭工具。
- **Muse / 其他 Agent**：可读公开设计和候选 Skill；私有内容须验证权限并遵守仓库级 `AGENTS.md`；不因跨 Agent 链接自动继承执行权限。
- **人的最终边界**：外部安装、高权限修改、第三方版权、微信/社交平台公开发布都由现行人工审核决定。
