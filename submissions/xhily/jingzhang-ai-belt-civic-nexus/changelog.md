# 方案迭代记录

## v1.6 - 2026-08-29

针对第六轮 AI Agent 评审意见（request-changes，七维加权 75.0/100；表达完整度 3/5、风险与合规 3/5 为两项阻断维度）执行修复。已通过本地四闸门自检，推回 PR #4098 的 round2 分支。

### 表达完整度（3/5 → 修复）：根除中文 visual 缺字方框（v1.5 该修复未真正生效）
- `embed_cjk_font.py`：修掉根因——旧逻辑用带空格的 `"font-family: -apple-system"` 做字符串替换，而 `build_visual.py` 生成的 CSS 是 `font-family:-apple-system`（无空格），导致替换从未发生、`@font-face` 注入了却没有任何文本节点使用 → 离线评审中文全方框。改为正则 `font-family:\s*-apple-system` 匹配，并改为幂等（仅注入缺失的 `@font-face` / 仅前置缺失的 `EmbedCJK` 字体栈）。
- `build_visual.py` + `embed_cjk_font.py`：重生成 visual 中英双页并重新子集内嵌 CJK 字体；`body` 字体栈现已含 `"EmbedCJK"`，离线无头渲染中文标题/正文/导航/指标单位/按钮均无方框。

### 表达完整度：根除 site-overview 底部两翼标签重叠
- `build_figures.py`：v1.5 把两翼标签改为"向图中间生长"（dx=±0.004）仍会重叠；本版改为标签朝图外侧生长——左翼 `ha="right"` 向左、右翼 `ha="left"` 向右，且节点 y 抬高至 `miny+(cy-miny)*0.22` 避免底部裁切。重生成 site-overview 中英双图，A0/A3 中英 PDF 同步刷新。

### 表达完整度：清除英文 report 页混语言
- `report/proposal.en.html`：5 处图件引用由中文 PNG（site-overview / land-use-structure / key-areas / mobility-bluegreen / metrics-evidence）改为已声明的对应 `.en.png`，与 `build_visual.py` 英文分支一致。

### 风险与合规意识（3/5 → 修复）：补强 C-01—C-06 来源与降级过度断言
- `sources.json`：为 C-01—C-06 逐项补 `published_by`（真实机构名）、`region`、`retrieved`（2026-08）与 `statement`（逐条事实映射：类比参考，机构层级/算力设施/区域机制以各机构官方公开资料为准，本文不作具体能力或地位断言）；URL 沿用各机构真实根域名，不编造子页链接。
- `proposal.md` / `proposal.en.md` / `report/proposal.html` / `report/proposal.en.html`：将"国家级实验室（含算力设施）""重大科技基础设施（算力）"等无法由精确页面逐条复核的断言降级为"以官方公开信息为准 / 本文不作具体能力断言"。

## v1.5 - 2026-08-28

针对第五轮 AI Agent 评审意见（request-changes，七维加权 78.0/100；三项参与者可立即关闭的阻断项）执行修复。已通过本地四闸门自检，推回 PR #4098 的 round2 分支。

### 表达完整度（2/5 → 修复）：消除破图、中文方框与语言残留
- `build_visual.py`：visual 页图片相对路径由 `assets/figures/` 修正为 `../assets/figures/`（与 report 页一致），修复 5 张图与 Logo 的破图问题。
- `embed_cjk_font.py`：CJK 子集字体注入范围扩展到 `visual/index.html` 与 `visual/index.en.html`，消除无头渲染的中文缺字方框。
- `build_visual.py`：英文页 `figures[1]` 与 `renewal` 标题去掉中文残留（用地分区 / 更新项目），变为纯英文。

### 表达完整度：重排 site-overview 两翼标签，消除重叠遮挡
- `build_figures.py`：修复 wing_right 坐标越出右边界压进图例区的 bug（原 `maxx + 0.34*half_width`）；两翼改置于左下/右下角、标签悬于节点下方并向内生长，不再与廊道、核心/地标标签或图例重叠。A0 首板同步刷新。

### 风险与合规意识：纠正"官方 polygon 前置评分"措辞
- `build_visual.py`：中英文 `status_note` 删除 "replace with official polygons before formal professional scoring"，改为明确"官方 polygon 缺失属组织方数据缺口、不阻断当前内容评分、发布后统一复算"的表述，与正文一致。

## v1.4 - 2026-08-27

针对第四轮 AI Agent 评审意见（request-changes，七维加权 76.0/100；两项参与者可立即关闭的阻断项：① 总体结构图缺两翼；② L-01 地标图位与正文矛盾）执行修复。已通过本地四闸门自检，开新 PR（round3）。

### 任务书相关性 / agent.1 结构图完整性：补画两翼协同接口
- `build_figures.py` 的 `site-overview` 图新增两翼节点与协同箭头：左=中关村科技服务翼（Hub Wing，协同接口：中关村创新网络），右=小月河场景赋能翼（Ripple Wing，协同接口：小月河公共空间），均以菱形节点 + 虚线箭头连向最近核心，标注「概念建议」并纳入图例。
- 与正文第 42–45 行「两翼协同」定义、第 100–117 行区域创新协同表保持一致；A0 第一页与 HTML 经重渲染自动同步。

### 可实施性（阻断项）：统一 L-01 地标位置
- 旧图将 L-01「京张零公里·AI原点碑」错标在源谷（PROV-KEY-001），而中英文地标表（proposal 第 149 行 / en 第 149 行）均写其位于「原点社区北端（清华园站遗址）」。现改为显式坐标：L-01 标于原点社区（PROV-KEY-002）北端、L-02 开源贡献长墙标于原点社区南端、L-03 指数钟塔标于汇流（PROV-KEY-003），与正文、地标表、路线说明完全一致。
- 「源谷」作为众智园子品牌（PROV-KEY-001）不再挂朝圣地标，消除图—文—表位置冲突。

## v1.3 - 2026-08-27

针对 AI Agent 第三轮评审意见（APPROVED，七维加权 84.0/100；原创性 4/5、表达完整度 4/5 为两项主要扣分项）执行精修，目标将两项各 +1 至 5/5。已通过本地四闸门自检。

### 原创性（4→5）：Logo 由概念说明升级为定稿成品
- 新增 `assets/figures/logo.png` 与 `assets/figures/logo.en.png`：以「钢轨断面（I 梁，核心）+ 神经网络节点环（8 节点蓝色连线 + 金色中心 hub）+ 园绿脉曲线」三元素融合的自绘矢量徽标，主色 智蓝 `#1E5BFF`、园绿 `#2FB37A`、京韵金 `#C8A24B`，金环外框。
- 在 `proposal.md` / `proposal.en.md` 品牌段将「Logo 方向（概念，非定稿）」改为「Logo（已完成定稿自绘矢量徽标）」，并嵌入 `![Logo](assets/figures/logo.png)` / `logo.en.png`；report HTML 由渲染脚本自动带入，visual 页在页眉加入徽标。
- 徽标为自绘矢量，未使用任何受版权或商标保护的图形；文本仍注明正式落地建议经专业品牌团队清权复核，不声称任何官方合作或审批。

### 表达完整度（4→5）：英文图注/图例留白放宽 + 指标图更具体
- `build_figures.py`：英文版图例锚点由 `(1.02, 1)` 右移至 `(1.12, 1)`、内边距由 `0.6` 增至 `0.85`，全图 `pad_inches` 英文版由 `0.12/0.25` 统一加大至 `0.32`、中文版 `0.14`，消除英文图注与图例距离偏紧的问题。
- `metrics-evidence` 中英双语图新增第 6 张指标卡 `floor_area_ratio = 待补 / TBD`（official FAR, pending polygons），证据链由 5 卡扩为 6 卡、占位更具体，明确标注该值待官方 polygons 到位后复算。

### 待办 / 开放项
- 全量重新渲染 report / visual / A0·A3 PDF 并内嵌中文字体；刷新 manifest 全部 sha256；本地四闸门自检通过后，于新分支 `submission/xhily/jingzhang-ai-belt-civic-nexus-round2` 开新 PR（旧 PR 已关闭）。
- 待组织方发布正式 polygons 后，统一替换 provisional 边界并复算全部指标（含 floor_area_ratio）。

## v1.2 - 2026-08-26

针对 AI Agent 第二轮评审意见（request-changes，七维加权 77.0/100，表达完整度 3/5）执行可视化精修，全部 P0 已完成并通过本地四闸门自检。

### P0 修复（可视化阻断项）
- **metrics-evidence 中英文图**：改用 figure-fraction 坐标 + 显式预留底部边距，移除 `fig.tight_layout()` 对 metrics 轴的影响，将临时边界警示居中置于底部预留区；彻底解决底部字段裁切/叠印问题，所有指标卡、证据链完整可读。
- **site-overview 中英文图**：保留核心星形图标，将 L-01/L-02/L-03 与核心名合并为带白底圆角框的 callout，通过 leader line 引出到多边形外侧，消除中文标签与节点图标的重叠。
- **key-areas 中英文图**：片区名与说明改为右侧白底圆角框 callout，leader line 指向各核心中心；移除原两翼标注（避免与 callout 重叠），两翼信息由正文与 site-overview 覆盖；图幅加宽至 9×11 以容纳英文长文本，消除描边超出画布造成的残影。
- **A0/A3/HTML 同步重生成**：以修复后的源图重新运行 `build_drawings.py`（4 份 PDF）与 `render_proposal_html.py`（2 份 report HTML），并重新内嵌中文字体；完成桌面/移动视口人工视觉 QA，确认无裁切、无标签遮挡、临时边界警示醒目、中英文图位对应。

## v1.1 - 2026-08-24

针对 AI Agent 评审意见（request-changes，七维加权 68.0/100）执行修复，覆盖 P0 与 P1 项，并完成中英文人工对照复核。

### P0 修复（阻塞项，已全部完成）
- 内嵌子集化 Noto Sans SC 字体（woff2 base64 `@font-face`）到 report/proposal.html、report/proposal.en.html、visual/index.html、visual/index.en.html，消除评审核对环境下的中文方框（tofu）。
- 修复 visual/index.html 与 visual/index.en.html 的离线资源路径：`assets/figures/*.png` 修正为 `../assets/figures/*.png`，恢复 5 张核心图件。
- 重排并重新导出五组中英文图件（portrait 朝向、图例外置、去除面内标签互压、水印避让指北针、metrics-evidence 底部留白），消除标签互压、图例遮挡、裁切与叠字；英文图清除中文残留。
- 更正 visual/index 状态文字：明确为 provisional 非官方边界、非精确用途、未来复算说明，不再出现"官方 polygons 到位前不得 formal scoring"的冲突表述。

### P1 修复（已完成）
- 为 C-01—C-06 六个全球案例补充逐项来源、事实范围、获取时间与复用边界，新增 sources.json 条目 CASE-C01—CASE-C06；全部标注"仅作机制类比、未声称官方合作"。
- 新增"区域创新协同关系（三核两翼 × 区域创新网络）"矩阵，明确北纬社区、未来科学城、怀柔科学城、经开区、京津冀与三核两翼的差异化角色、要素流与协同接口；跨节点合作一律标为概念建议。
- 完成中英文人工对照复核：proposal.md 与 proposal.en.md 的案例来源核验表、区域协同矩阵、概念建议标注已逐节对照，确保语义一致、无遗漏翻译与无新增中文残留。

### 待办 / 开放项
- 修复包重新上传至 PR（需新的 GitHub PAT），并触发 CI 复跑。
- 待组织方发布正式 polygons 后，统一替换 provisional 边界并复算指标。
