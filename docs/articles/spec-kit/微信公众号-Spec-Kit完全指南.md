# 别再「凭感觉写代码」了：一篇讲透 GitHub Spec Kit

> **适合人群**：开发者、技术负责人、产品经理、正在用 AI 写代码的同学  
> **阅读建议**：先看「各自是什么」与「关系图」，再对照文末示例上手

---

![封面：从规格到代码的 Spec Kit 工作流](./images/spec-kit-cover.jpg)

你是不是也遇到过这种场景：

对 AI 说一句「帮我做一个相册应用」，它很快吐出一大坨代码——能跑一点，但总差那么点意思：登录怎么做？拖拽排序算不算？错误态谁管？技术栈是不是你团队要的？

GitHub 官方把它叫作 **vibe coding（凭感觉编程）**：看起来对，细看不对；适合 demo，不适合认真交付。

GitHub 开源的 **Spec Kit**，就是为了把这件事扳回来——让**规格（Spec）成为可执行的真相源**，而不是写完就被丢掉的脚手架。

---

## 一、先把几个概念讲清楚

很多高阅读量文章会把「SDD」「Spec Kit」「Specify CLI」「斜杠命令」混着说。其实它们是一层套一层的关系，先拆开：

| 概念 | 它是什么 | 一句话定位 |
| --- | --- | --- |
| **Spec-Driven Development（SDD / 规范驱动开发）** | 一种开发方法论 | **先定义「做什么、为什么」，再让实现跟着规格走** |
| **Spec Kit** | GitHub 开源工具包（MIT） | 把 SDD **落地成可复用流程**，适配 Copilot / Claude Code / Cursor 等助手 |
| **Specify CLI（`specify`）** | Spec Kit 的命令行入口 | 负责 `init`、装模板、接 AI 集成、装扩展 |
| **`/speckit.*` 命令 / Skills** | AI 助手里的工作流步骤 | 真正驱动「宪法 → 规格 → 方案 → 任务 → 实现」 |
| **Artifacts（产物文件）** | `constitution.md`、`spec.md`、`plan.md`、`tasks.md` 等 | 每一步留下**可审阅、可回退、可对齐**的 Markdown 真相 |

![对比：左侧 vibe coding，右侧 Spec 驱动](./images/spec-kit-vs-vibe.jpg)

**官方核心论断（GitHub Blog）可以记一句：**

> 问题往往不在 AI「不会写代码」，而在我们把它当搜索引擎用；它需要的是**明确、可验证的指令**，而不是模糊愿望。

---

## 二、Spec Kit 里「各自是什么」

可以把 Spec Kit 想成一套「给 AI 的工程流水线」。主线命令如下（命令名以官方当前形式 `/speckit.*` 为准）：

### 1. Constitution（项目宪法）——`/speckit.constitution`

**是什么**：项目级「非谈判原则」——质量底线、测试要求、体验一致性、性能红线、合规约束等。  
**产物**：通常是 `.specify/memory/constitution.md`。  
**特点**：一般**每个项目做一次**，后续所有步骤都拿它当评判标准。

> 高赞实践文的共识：宪法写得越清楚，后面 AI「自由发挥」就越少跑偏。

### 2. Specify（规格）——`/speckit.specify`

**是什么**：用自然语言描述**用户旅程、成功标准、边界与非目标**。  
**关键纪律**：只写 **What / Why**，**不要在这里堆技术栈**。  
**产物**：`spec.md`（以及需求检查清单相关内容）。

### 3. Clarify（澄清）——`/speckit.clarify`（质量门）

**是什么**：针对规格里含糊的地方，一次最多抛出约 5 个高影响问题，把你的回答**写回 spec**。  
**何时用**：需求有歧义、边界不清、生产级功能——强烈建议在 Plan 之前跑。

### 4. Plan（技术方案）——`/speckit.plan`

**是什么**：把「做什么」翻译成「怎么做」——技术栈、架构、集成约束、性能与合规要求。  
**产物**：`plan.md`，以及研究/设计类附属文档（如 `research.md` 等）。  
**特点**：公司强制技术栈、遗留系统对接、安全基线，都应该写在这里。

### 5. Checklist（质量清单）——`/speckit.checklist`（质量门）

**是什么**：给「英文写的需求」做单元测试式检查——完整性、清晰度、一致性。  
**比喻**：像给规格本身写测试。

### 6. Tasks（任务拆解）——`/speckit.tasks`

**是什么**：把 Spec + Plan 拆成**可独立实现、可单独验收**的小任务。  
**产物**：`tasks.md`（按用户故事与依赖排序，常带并行标记）。  
**为什么重要**：AI 最怕「做一个认证系统」这种巨型任务；它需要「创建校验邮箱格式的注册接口」这种粒度。

### 7. Analyze（交叉分析）——`/speckit.analyze`（质量门）

**是什么**：在动手写代码前，检查 Spec / Plan / Tasks 之间是否有遗漏、冲突、覆盖不足。  
**建议时机**：`tasks` 之后、`implement` 之前。

### 8. Implement（实现）——`/speckit.implement`

**是什么**：按 `tasks.md` 的依赖顺序执行实现（可分阶段，避免一次塞爆上下文）。  
**你的角色**：从「写一万行」变成「审一小块、验一小块」。

### 9. Converge（收敛）——`/speckit.converge`

**是什么**：对照 Spec / Plan / Tasks 审查当前代码库，把缺口追加成新任务（append-only），再交回 Implement。  
**意义**：补上「实现后反馈环」，减少规范漂移（Spec Drift）。

### 10. 周边能力（了解即可）

| 能力 | 说明 |
| --- | --- |
| `/speckit.taskstoissues` | 把任务同步成 GitHub Issues，便于团队跟踪 |
| **Bug 扩展** | `assess → fix → test`，修 bug 也有证据链 |
| **Assess 扩展** | `intake → research → define → shape → decide`，先决策要不要做 |
| **Extensions / Presets / Bundles** | 扩展、预设、面向角色的一键配置包 |

---

## 三、它们之间是什么关系？

一句话：

> **人负责意图与把关，Spec Kit 负责把意图固化成产物，AI 助手负责在产物约束下写代码。**

![组件关系：意图 → 产物链 → AI 执行](./images/spec-kit-components.jpg)

### 1. 依赖关系（数据怎么流）

```text
Constitution（约束全局）
        ↓
Specify（定义目标） → Clarify（消歧，可选但推荐）
        ↓
Plan（技术落地） → Checklist（质量门，可选）
        ↓
Tasks（可执行切片） → Analyze（一致性门，可选）
        ↓
Implement（写代码） ⇄ Converge（对照规格收敛）
```

官方也强调：严格必经的最短主线是  
**Specify → Plan → Tasks → Implement（→ Converge）**；  
Clarify / Checklist / Analyze 是**有歧义、有风险时加上的质量闸门**。

![工作流管道示意](./images/spec-kit-workflow.jpg)

### 2. 职责关系（谁管什么）

| 层级 | 管什么 | 不管什么 |
| --- | --- | --- |
| Constitution | 长期原则与红线 | 单个功能细节 |
| Spec | 用户价值与行为契约 | 具体框架选型 |
| Plan | 技术路径与约束 | 逐行代码 |
| Tasks | 可验收的工作切片 | 产品愿景本身 |
| Implement | 交付可运行增量 | 偷偷改需求定义 |
| Converge | 发现缺口并回流任务 | 直接改写 Spec/Plan 当「事后圆谎」 |

### 3. 和「传统文档」的本质区别

传统流程里，PRD / 设计文档常常在编码开始后**失去权威**。  
SDD + Spec Kit 的翻转是：**规格驱动生成计划、任务与实现**；文档不是附属品，而是流水线的输入。

这也是 jqknono、博客园等高阅读解读文反复强调的一点——  
**不是「多写了几份 Markdown」，而是「Markdown 成了可执行契约」。**

---

## 四、什么时候该用？三个高价值场景

GitHub Blog 点名了三类最吃香的场景；中文实践文基本也围绕它们展开：

![三大场景：从 0 到 1、存量加功能、遗留现代化](./images/spec-kit-scenarios.jpg)

### 场景 A：Greenfield（从 0 到 1）

**痛点**：一上来就写代码，AI 会用「最常见模板」猜你的产品。  
**用法**：先 Constitution + Specify，把成功标准写死，再 Plan。  
**收益**：少返工，骨架更像「你的产品」而不是「通用脚手架」。

### 场景 B：存量系统加功能（N → N+1）——往往最值钱

**痛点**：新功能很容易「长成外挂」，和旧架构打架。  
**用法**：Spec 写清与旧系统的交互边界；Plan 写清必须遵守的架构约束。  
**收益**：AI 生成的代码更「像本仓库的孩子」。

### 场景 C：遗留系统现代化

**痛点**：业务意图散落在老代码、口头传统和过期 Wiki 里。  
**用法**：先用 Spec 捞出「必须保留的业务契约」，再用 Plan 设计新架构，最后 Implement 重建。  
**收益**：现代化时带走业务，不带走历史包袱。

### 额外好用场景（实践文常见补充）

- **快速原型**：最短路径跑通 Specify → Plan → Tasks → Implement  
- **多技术栈对照**：同一 Spec，换两套 Plan，比较实现代价  
- **团队工程对齐**：把安全、合规、设计系统写进 Constitution / Plan，而不是靠口头提醒  

---

## 五、完整示例：用 Spec Kit 做一个「相册整理」小应用

下面这个例子综合了官方 Quickstart 与多篇高阅读实践文的写法，可直接改成你的业务。

### Step 0：安装与初始化

```bash
# 安装 Specify CLI（建议钉版本，把 vX.Y.Z 换成最新 Release）
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z

# 初始化项目并接入你的 AI 助手（示例：Copilot）
specify init photo-album --integration copilot
cd photo-album
```

> 也可用 PyPI：`uv tool install specify-cli`。  
> 在 Cursor / Claude Code 等环境中，按官方集成参数选择即可。

### Step 1：写下项目宪法

在 AI 助手中：

```text
/speckit.constitution
创建一套原则：代码可读优先；关键路径必须有测试；
UI 交互保持一致；首屏操作 100ms 内有反馈；
禁止引入重型框架，除非 Plan 阶段明确批准。
```

### Step 2：只谈产品，不谈技术栈

```text
/speckit.specify
我想做一个帮助整理照片的应用：
- 用户可以把照片放进不同相册
- 相册默认按日期分组
- 在主页支持拖拽重新组织相册
- 未登录也可浏览示例相册，登录后才能创建与编辑
请聚焦用户故事、边界情况与验收标准，先不要选择技术栈。
```

### Step 3：澄清歧义（强烈建议）

```text
/speckit.clarify
重点澄清：拖拽冲突、删除相册时照片归属、以及移动端是否必须支持拖拽。
```

把 AI 的问题答完——答案应回写进 `spec.md`。

### Step 4：再谈怎么做

```text
/speckit.plan
技术方向：Vite + 尽量原生 HTML/CSS/JS；
本地优先存储；拖拽用成熟轻量方案；
需要说明数据模型、页面结构、测试策略与已知风险。
```

### Step 5：拆任务 →（可选）分析 → 实现 → 收敛

```text
/speckit.tasks
/speckit.analyze
/speckit.implement
/speckit.converge
```

大功能建议分段实现，例如：

```text
/speckit.implement 只实现数据模型与本地存储相关任务，做到可单独验证后停止。
```

若 Converge 报告尚未收敛，继续 Implement，直到状态为 **Converged**。

### 你会得到什么？

一个典型目录会类似：

```text
.specify/
  memory/constitution.md
  specs/001-photo-albums/
    spec.md
    plan.md
    tasks.md
    ...
```

**你的工作方式也会变：**

1. 改需求 → 优先改 Spec（必要时再 Clarify）  
2. 改技术约束 → 改 Plan  
3. 改执行切片 → 改 Tasks  
4. 让 AI 在约束内实现，而不是从零「猜你想要什么」

---

## 六、和「直接让 AI 写代码」差在哪？

| 维度 | 纯 Prompt / vibe coding | Spec Kit（SDD） |
| --- | --- | --- |
| 真相源 | 聊天记录与临时代码 | Spec / Plan / Tasks 文件 |
| 失败模式 | 看起来能跑、细节大量臆测 | 在 Plan/Tasks 阶段就暴露缺口 |
| 评审粒度 | 一次看上千行 | 按任务增量审阅 |
| 需求变更 | 推倒重来或补丁叠补丁 | 更新 Spec → 再生 Plan/Tasks |
| 组织知识 | 散落在 Wiki / 群聊 | 进入 Constitution 与 Plan，AI 真用得上 |
| 适合 | Demo、探索 | 要交付、要协作、要演进的功能 |

这也是为什么社区文章常把 Spec Kit 描述成：  
**给 AI 编程加「工程护栏」，而不是又一个代码生成器。**

---

## 七、上手建议（来自高阅读量文章的共识）

1. **先小后大**：用周末项目跑通最短路径，再上生产功能。  
2. **宪法别敷衍**：团队红线写进 Constitution，比事后骂 AI「又不遵守规范」有效得多。  
3. **Specify 阶段禁止提早选栈**：过早谈 React/Go，会污染需求纯度。  
4. **每个阶段都人工把关**：AI 产文档，人做「这是不是我想要的」判决。  
5. **复杂需求必开 Clarify / Analyze**：便宜的问题会议，贵的是做错的架构。  
6. **用 Converge 闭环**：没有收敛步骤，SDD 容易退化成「多写了三份说明文档」。  
7. **人机分工要清醒**：Spec Kit 提升的是「意图→实现」的可控性；安全审查、产品判断、最终责任仍在人。

---

## 八、一分钟速记

- **SDD** 是思想：规格可执行，代码为规格服务。  
- **Spec Kit** 是工具包：把思想变成命令、模板与产物链。  
- **Specify CLI** 是入口；**`/speckit.*`** 是日常驾驶盘。  
- **关系**：Constitution 约束全局 → Spec 定目标 → Plan 定路径 → Tasks 定切片 → Implement 交付 → Converge 对齐。  
- **最值场景**：存量加功能、严肃的新项目、遗留重建。  
- **口诀**：**先对齐意图，再生成实现；先写清楚，再写代码。**

---

## 参考与延伸阅读

1. GitHub 官方仓库：[github/spec-kit](https://github.com/github/spec-kit)  
2. GitHub Blog：《Spec-driven development with AI: Get started with a new open source toolkit》  
3. 官方文档：[Spec Kit Documentation](https://github.github.com/spec-kit/) / Quickstart  
4. 中文深度解读（架构与场景较完整）：jqknono《GitHub Spec Kit：官方规格驱动开发工具包深度解析》  
5. 实战向长文：RbBtSn0w《Spec-Driven Development 深度指南 — Part 2: GitHub Spec Kit 实战指南》  
6. 落地案例向：博客园《AI规范编程：从SDD理念到Spec-Kit落地实践》

---

### 公众号发布小贴士（可删）

- **标题备选**  
  - 别再凭感觉让 AI 写代码了：GitHub Spec Kit 一篇讲透  
  - Spec Kit 是什么？Constitution / Specify / Plan / Tasks 关系全拆解  
  - 从 vibe coding 到规范驱动：Spec Kit 实战指南  
- **封面**：使用 `images/spec-kit-cover.jpg`（16:9）  
- **文中插图顺序建议**：封面 → 对比图 → 组件关系 → 工作流 → 场景图  
- **标签建议**：AI编程、GitHub、工程效能、Copilot、开发工具

---

*本文基于 GitHub Spec Kit 官方说明与公开高阅读解读文章整理创作，命令名称以你本地 `specify` 版本与官方文档为准；若版本升级导致命令别名变化，以官方 README / quickstart 为准。*
