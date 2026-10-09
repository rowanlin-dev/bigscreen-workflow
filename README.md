# 大屏工作流（bigscreen-workflow）

全程 AI 辅助开发数据大屏：**人只需要给需求、给数据、说"确认"**。

- **skill 内核**（`skill/`）：六阶段流水线 + 决策分级门禁 + 两套场景预设，AI 每阶段产出可看的东西再停顿。
- **网页外壳**（`web/`）：同一份状态的可视化面板——看产物（配色卡/布局缩略图/截图/裁决表）、点确认、生成命令。

两者共享唯一真相源 `<项目>/.bigscreen/state.json`，**中断可恢复**。

---

## 快速开始

### ① 初始化一个新大屏项目

```bash
node skill/scripts/init.js --root <项目绝对路径> --name "某运营指挥中心" --preset delivery-map --mode staged
```

### ② AI 按阶段干活，每阶段提交产物并挂起

```bash
# 提交产物，等人确认
node skill/scripts/advance.js --root <项目> --stage S0 --artifact "00-任务与规范/00-任务拆解.md" --label "任务拆解" --await
# 人确认后推进
node skill/scripts/advance.js --root <项目> --stage S0 --decision confirmed
```

**门禁是机制不是约定**：硬门禁阶段没执行过 `--await` 就 `--decision confirmed`，脚本直接拒绝（退出码 1）。AI 想跳过确认跳不过去。

### ③ 网页面板

直接双击打开 `web/index.html`（默认加载内置示例），或：

- 点右上角「载入 state.json」→ 选择你项目下的 `.bigscreen/state.json`；
- 或把 `.bigscreen/state.js` 复制到 `web/` 目录覆盖示例；
- 或访问 `web/index.html?state=file:///E:/.../your-project/.bigscreen/state.js`。

面板里点「确认通过 / 驳回」会**生成对应的 advance.js 命令**，复制给 AI 执行即可——网页负责看和选，命令行负责真正写文件。

#### 决策面板里三个控件分别是什么

| 控件 | 作用 | 说明 |
|---|---|---|
| **决策备注**（文本框） | 给 AI 的「决策附言」 | 可选。内容会作为 `--note` 一起拼进命令。它不是「实时改需求」，而是在这一关给 AI 一句话——**驳回**时就是让 AI 重做的具体要求（如"主色再亮一点"），**确认**时则是留给后续阶段的背景说明。留空也行。 |
| **确认通过** | 生成 `--decision confirmed` 命令 | 标记当前阶段产物 OK，允许推进到下一阶段（硬门禁需先经过 AI 的 `--await`）。 |
| **驳回** | 生成 `--decision rejected` 命令 | 标记当前阶段产物不合格，AI 看到后按备注重做本阶段。 |

**关键认知**：面板本身**不直接改文件、不改需求**。点按钮只是把命令填进下方命令框，复制后交给 AI 或在终端运行，真正写 `state.json`、跑服务、截图的是 `advance.js`。所以这是一个「人看产物 → 点选 → 把指令交给 AI」的闭环，不是表单式实时编辑。

---

## 两种模式

| 模式 | 适用 | 行为 |
|---|---|---|
| `staged`（默认） | 只有领导一句话 + 部分数据 | 硬门禁停 4 次（S0/S1/S3/S5），软门禁展示后过（S2），S4 不停 |
| `express` | 已有完整需求 + 数据 | 启动问卷收齐答案后一路跑完，仅 S5 停 |

**随时降级**：任何时刻说"后面你看着办"→ AI 用 `--auto` 推进剩余阶段，产物照出，只是不再停顿。

---

## 决策分级（门禁为什么装在这几处）

| 层 | 谁决定 | 例 | 门禁 |
|---|---|---|---|
| L3 | 必须问人 | 业务目标、指标口径、数据源、观看距离 | 硬 |
| L2 | 出产物后点头 | 布局、图表选型、配色 | 硬/软 |
| L1 | AI 建议默认过 | 字号数值、动效时长、组件命名 | 软 |
| L0 | AI 自决记账 | 目录结构、依赖版本 | 无 |

核心判断：**问题不在"问几次"，在"哪些必须问"**。L3 问卷能问清；L2 必须先看到配色卡/布局缩略图才答得出——所以 S2/S3 的门禁是"展示产物后确认"，不是文字问卷。

---

## 目录结构

```
bigscreen-workflow/
├── skill/
│   ├── SKILL.md                    # AI 入口：三铁律、模式判定、执行流程、坑清单
│   ├── references/
│   │   ├── 01-阶段定义与门禁.md      # 每阶段：输入/产物/完成判据/问什么
│   │   ├── 02-问题清单.md           # 分级问题库（含默认值与问法）
│   │   ├── 03-状态文件规范.md       # state.json 完整 schema 与脚本契约
│   │   ├── 04-图表选型规则.md       # 永久否决项、条件否决、性能红线
│   │   ├── 05-图片反推流程.md       # 甲方给图照做：能读什么/读不出什么/降级策略
│   │   └── 06-向用户呈现样式的方法.md # 布局用 ASCII、颜色用图的标准动作
│   ├── presets/
│   │   ├── index.json              # 预设清单（含成熟度/置信度/复用边界）
│   │   ├── industrial/             # 工业制造（冷蓝绿 / 图表网格）· verified
│   │   ├── delivery-map/           # 生活服务+地图（暖橙 / 地图主导）· verified
│   │   ├── park-demo/              # 从 HTML 模板抽取的示例 · derived
│   │   ├── client-health/          # 从参考图反推的示例 · derived
│   │   ├── ext-logistics/          # 从外部仓库下载抽取的示例 · derived
│   │   └── external/               # 外部资源索引（catalog.json + 版权纪律）
│   ├── package.json                # 零依赖；engines.node>=18、bin、test 脚本
│   ├── test/
│   │   └── smoke.js                # 冒烟测试（init→advance 全流转，6 断言）
│   └── scripts/
│       ├── init.js / status.js / advance.js / decide.js   # 状态机，零依赖
│       ├── presets.js              # 预设候选匹配（list/match/show/layers）
│       ├── extract-preset.js       # HTML 模板 → 预设（配色/字号/布局/图表）
│       ├── spec-to-preset.js       # 看图反推 spec → 预设
│       ├── fetch-external.js       # 外部模板按需下载单页 + 自动抽取
│       ├── build-catalog.js        # 生成外部仓库目录索引（117 个模板）
│       ├── layout-preview.js       # 布局 ASCII 预览 + 体检
│       ├── palette-card.js         # 配色对比色卡（单文件 HTML）
│       └── lib/ (store.js / ascii.js)   # store.js = 状态读写库（原 lib/state.js 重命名）
├── web/                            # 网页面板（纯静态，file:// 可开）
│   ├── index.html / styles.css / app.js
│   └── state.js                    # 内置示例数据（可被你项目的覆盖）
└── examples/
    ├── replay.sh                   # 端到端演示脚本（S0 推到 S3 挂起）
    ├── demo-project/.bigscreen/    # 演示状态（当前停在 S3 等待确认）
    ├── 配色对比卡.html              # palette-card.js 的输出示例
    └── fixtures/                   # 测试素材（示例模板 / 反推 spec）
```

---

## 预设包里有什么

两套预设均来自**实际跑通并验收过的项目**（`E:/0-utility_tool/bigscreen-demo` 与 `delivery-demo`），每套含：

| 资产 | 说明 |
|---|---|
| `preset.json` | 配色 token、字号阶梯、布局网格、指标树、图表裁决、自洽断言（机器可读，init 时自动预填面板预览） |
| `DESIGN.md.tpl` | 设计规范模板（`{{占位符}}` 替换后即为 S2 产物） |
| `图表选型表.md.tpl` | 候选全集裁决表模板（含"不做什么"及理由） |
| `数据字典.md.tpl` | 指标口径、接口清单、自洽断言、已实测的数据真实性陷阱 |

匹配方式：`node scripts/presets.js match --desc "<需求>"` 列出**候选卡片**（含成熟度、置信度、局限），确认后才采用——**预设是候选不是必选**，且只贡献设计 token 层，业务层（指标/断言/图表清单）必须按本次业务重填。

新增预设的三条路：
| 来源 | 命令 | 成熟度 |
|---|---|---|
| 有好看的 HTML 模板 | `node scripts/extract-preset.js --file <html> --id <id> --name "<名>"` | derived（必须过 REVIEW.md） |
| 甲方只给一张图 | AI 读图写 spec → `node scripts/spec-to-preset.js --spec <spec.json> --id <id>` | derived（数值全是估计） |
| 外部模板仓库按需取 | `node scripts/fetch-external.js --source bigdataview --id 041 --out <目录> --extract` | derived（版权纪律见 external/INDEX.md） |

---

## 选样式时怎么问用户

**布局用 ASCII，颜色用图**——文字描述布局每个人脑补的不一样，色值对非技术用户是天书。

```bash
node scripts/layout-preview.js --list --check           # 布局骨架 ASCII + 体检
node scripts/palette-card.js --presets industrial,delivery-map --out 配色卡.html
```

完整方法与话术见 `skill/references/06-向用户呈现样式的方法.md`。

---

## 测试与校验

每次改动脚本后，跑冒烟测试确认状态机没回退：

```bash
cd skill
node test/smoke.js      # 或直接 npm test
# 6 项断言：schema 校验 → S0 推进 → 硬门禁拦截 → --decision auto 拒绝 → 驳回记 feedback
```

测试覆盖：初始化、阶段推进、硬门禁机制、决策语义（`--auto` 仅作标志、`auto` 取值被拒）、驳回写反馈。所有脚本零第三方依赖，`node --check` 即通过。

> 旧项目（在本更新前创建）的 `state.json` 不含 `project.skillDir` 字段：网页面板会回退到 `<skill>/scripts` 占位符并提示，手动替换一次即可；跑一次 `init.js --force` 也能补上该字段。

---

## 开发这套工具时实测踩的坑

1. **参数解析覆盖**：`--artifact` 传两次只记一次（parseArgs 覆盖而非累积）→ 已改为重复键累积成数组，`--label` 与 `--artifact` 按序配对。
2. **`--preview-only` 无效**：先加产物预览回贴功能时忘了和 `--preview-file` 绑定 → 已绑定，只更新面板预览不改变阶段状态（S4 出图后回贴截图就是靠它）。
3. **file:// 下 fetch json 被 CORS 拦** → 网页面板不 fetch，而是加载 `state.js`（`window.__BIGSCREEN_STATE__ = {...}`），双击即用；另提供「载入 state.json」按钮走 FileReader。
4. **内存采样**：验收时的内存稳定性必须 ≥4 次采样看趋势，单点会误判（JS 堆是锯齿形）。
5. **抽取器的颜色误判**（extract-preset.js 三连坑）：深蓝背景在 HSL 里饱和度高被误判成主色 → 改按亮度优先；HSL 的 s 在高亮度时虚高（淡蓝灰算出 0.34）→ 改用绝对色度 chroma；色相区间判定让"暗淡灰蓝"抢走主色 → 改按【饱和度×亮度居中度】打分选主色。
6. **rem 基准被当字号**：模板 `html{font-size:100px}` 是换算基准，抽取时必须排除 html/:root 选择器。
7. **外部模板只下 index.html 抽到全空**：样式在 css/ 子目录、图表代码在 js/ 子目录 → 改为解析引用递归下载。
8. **布局体检误报两条**：全宽条带（Header/KPI）算进主视觉竞争会冤枉好布局 → 排除；网格型大屏（工业）故意无焦点却被判"没焦点" → 先分型再体检（focused 看主导比 / grid 看均衡度）。
9. **sed 转义事故**：`s/\| .../` 在 BRE 里 `\|` 是"或"，空模式导致每行被插前缀、文件损坏 → 改文档用 Edit 工具，别用带管道符的 sed。

---

## 与前两个 demo 的关系

- `E:/0-utility_tool/bigscreen-demo` — 工业大屏实例（这套工作流的"第一次人肉跑通"）
- `E:/0-utility_tool/delivery-demo` — 生活服务地图大屏实例（验证了地图图层与代理坑）
- 本项目把两次跑通的经验**固化成机制**：以后新大屏从 `init.js` 开始，预设继承 + 门禁拦截 + 网页确认。
