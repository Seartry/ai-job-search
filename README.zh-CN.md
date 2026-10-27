<p align="center">
  <img src="assets/mascot/pip_flight_loop.gif" alt="Pip，那只信使鸟" width="200">
</p>

# AI 求职助手

*跑在你自己机器上的求职流程。*

[English](README.md) | **简体中文**

> 本文件是 [README.md](README.md) 的中文翻译版。内容如有出入，以英文原版为准。

<p align="center">
  <a href="https://trendshift.io/repositories/43622?utm_source=trendshift-badge&amp;utm_medium=badge&amp;utm_campaign=badge-trendshift-43622" target="_blank" rel="noopener noreferrer"><img src="https://trendshift.io/api/badge/trendshift/repositories/43622/daily" alt="MadsLorentzen%2Fai-job-search | Trendshift" width="250" height="55"/></a>
</p>

[![CI](https://github.com/ai-job-search-dev/ai-job-search/actions/workflows/ci.yml/badge.svg)](https://github.com/ai-job-search-dev/ai-job-search/actions/workflows/ci.yml)

一个基于 [Claude Code](https://claude.com/claude-code) 的 AI 求职申请框架。Fork 一份，填好自己的资料，然后让 Claude 评估职位、定制简历、撰写求职信，并帮你准备面试。

> 说明：这是一个独立的开源项目，与 Anthropic 没有关联，也不由 Anthropic 认可、赞助或维护。文中提及 Anthropic 和 Claude Code，仅用于说明本流程所使用的工具链。
>
> 本项目**没有任何关联的加密货币、代币或付费赞助计划**。任何声称与本项目有关的此类说法都未经授权，请一律当作骗局处理。支持本项目的方式只有两种：点击下方的 Ko-fi 链接，或在 GitHub 上参与贡献。

## 它真的管用吗

我是地球物理专业出身。2025 年底岗位被裁掉之后，我写了这个框架来跑自己的求职——就是本仓库里这套 `/scrape`、`/apply`、`/interview` 流程，每周用一次，用在我自己的职业上。和每一位聊过的雇主我都坦白了这件事，结果它通常不但不减分，反而引出一场实打实的技术对话。

六十九份量身定制的申请、二十场首轮面试、一份签下的合同，2026 年 6 月我以 AI 工程师的身份入职。总有人问我：这套东西真有用吗？它帮我拿到了工作。现在，它归你了。

*更完整的版本（含完整的申请漏斗数据）在 [LinkedIn](https://www.linkedin.com/in/mads-lorentzen/) 上。*

<p align="center">
  <i>它帮你省下了一个写求职信的周日？请我喝杯咖啡吧。<br>
  它帮你拿到了工作？那也许该请两杯。</i> ☕
</p>

<p align="center">
  <a href="https://ko-fi.com/madslorentzen">
    <img src="https://storage.ko-fi.com/cdn/kofi3.png?v=6" alt="在 ko-fi.com 请我喝杯咖啡" height="40">
  </a>
</p>

## 这是什么

一套结构化的流程，把 Claude Code 变成一个全栈的求职申请助手。核心流程（自我画像、匹配度评估，以及「撰写—评审」的申请流水线）**不依赖特定语言或国家**。招聘网站搜索 skill 目前是为丹麦市场写的（Jobindex、Jobnet、Akademikernes Jobbank 等），但这套模式本来就是设计成可以替换为你本地的招聘网站的。

```
/setup          /scrape              /apply <url>
  |                |                     |
  v                v                     v
填写个人资料      搜索招聘网站          评估匹配度
完善你的档案      汇总并去重            打分并给出建议
  |                |                     |
  v                v                     v
档案就绪          展示匹配结果          起草简历 + 求职信
                  附匹配度评分          （LaTeX，量身定制）
                   |                     |
                   v                     v
               选定一个职位            评审 agent 挑刺
               -> /apply               -> 修订 -> 最终产出
```

框架把职业指导的最佳实践固化了下来，包括结构化的评估标准、面向未来的求职信叙事，以及可选的薪资对标。

## 前置条件

- [Claude Code](https://claude.com/claude-code)（CLI）。Claude Code 没有免费额度：需要 Claude Pro/Max/Team 订阅，或 Anthropic API 额度（按 token 计费，偶尔使用通常更便宜）。想用别的 agent 工具（Codex、Antigravity、Gemini CLI）？从 [`AGENTS.md`](AGENTS.md) 开始——招聘网站搜索 skill 在那里开箱即用，[社区 fork](https://github.com/ai-job-search-dev/ai-job-search/discussions/78) 则把完整流程也适配了过去。
- Python 3.10+
- [Bun](https://bun.sh)（供求职搜索 CLI 工具使用）
- 装有 `lualatex` 和 `xelatex` 的 LaTeX 发行版：[TeX Live](https://tug.org/texlive/)、[MacTeX](https://tug.org/mactex/)、[TinyTeX](https://yihui.org/tinytex/) 或 [MiKTeX](https://miktex.org/)。简历用 `lualatex` 编译（在新版 MiKTeX 上 pdflatex 常因 `fontawesome5` 字体展开错误而失败）；求职信用 `xelatex` 编译，因为 `cover.cls` 依赖 `fontspec`。如果你用的是 TinyTeX、BasicTeX 这类精简 TeX 安装，还需要安装 [SETUP.md](SETUP.md#minimal-tex-install-tinytexbasictex) 里列出的额外宏包。
- 可选：`pip install pypdf`，用于 `/apply` 的 ATS 可解析性检查（BSD 许可，不需要 Poppler）。Poppler 的 `pdftotext` 仍是备选方案（macOS：`brew install poppler`，Debian/Ubuntu：`apt install poppler-utils`，Windows：`choco install poppler`）。两者都没有时，这项检查会降级为人工目视的关键词复核。

## 快速开始

> 🎥 **想先看看它怎么用？** [The Next New Thing 做了一次实操演示](https://www.youtube.com/watch?v=HoVxjMNFYv4)，从配置到完成一份申请完整走了一遍（录制于 2026 年 8 月，命令此后可能已有变化）。

### 1. Fork 并克隆

```bash
gh repo fork ai-job-search-dev/ai-job-search --clone
cd ai-job-search
```

> [!IMPORTANT]
> **本仓库的 fork 永远是公开的**——GitHub 不允许把公开仓库 fork 成私有仓库——而 `/setup`（下面第 3 步）会把你的个人数据（姓名、联系方式、工作履历、薪资预期）写进**受版本跟踪**的文件里。如果这份副本是给你自己求职用的，而不是为了回馈代码，请改用**私有仓库**、把本仓库设为 `upstream`——两分钟的操作步骤见 [SETUP.md 第 8 节](SETUP.md#8-pulling-upstream-updates-into-your-fork)，所有更新流程完全一样。只在要贡献代码时才需要 fork。

### 2. 安装求职搜索工具

```bash
python3 tools/install_portal_clis.py   # Windows: python tools\install_portal_clis.py
```

它会在 `.agents/skills/*/cli/` 下的每个招聘网站 CLI 里运行 `bun install`——这些目录是自动发现的而不是写死列出的，所以用 `/add-portal` 新加的、或从别的 fork 拷过来的网站，在全新克隆时也会被包含进来。如果某个 CLI 的 `package.json` 声明了安装期的生命周期脚本，它会被跳过并报告出来，而不是直接执行。`--dry-run` 可以预览会安装什么。

对 `linkedin-search` 和 `freehire-search` 来说，安装是可选的：两者都没有运行时依赖，直接用 `bun` 就能跑；`bun install` 只会拉取 TypeScript 的开发类型。

### 3. 配置个人资料

```bash
claude
# 然后在 Claude Code 里：
/setup
```

`/setup` 提供三条路径：读取已有内容的 `documents/` 文件夹（简历 PDF、LinkedIn 导出、学位证、推荐信、过往申请记录），导入一份粘贴到对话里的简历，或者走一遍访谈式问答。它会自动识别你手上有哪些材料，然后来问你。文件夹模式是幂等的，后续往里加材料可以放心重跑；目录结构见 `documents/README.md`。

### 4. 搜索职位

```bash
/scrape
```

它会到多个招聘网站搜索与你画像匹配的职位，去重后按匹配度排序展示。选定一个匹配的职位就能对它直接跑 `/apply`——或者，如果一次抓取返回的职位多到你不乐意逐个看，就先跑 `/rank` 把它们全部按匹配框架批量打分，拿到一份排序好的候选清单。

### 5. 申请职位

```bash
/apply https://jobindex.dk/job/1234567
```

如果 URL 抓不到（有些招聘网站会拦截自动化访问），也可以直接把职位描述粘进去：

```bash
/apply <在这里粘贴完整的职位描述>
```

这会跑完整流程：评估匹配度、起草简历与求职信、交给第二个 agent 评审、修订，最后交付成品。职位描述会先过一道确定性的完整性检查（隐藏文本、不可见字符、形似指令的行、隐藏链接），它的一句话结论会作为评估的开头；负责抓取和评审的 agent 都是只读的。

职位描述被当作不可信输入处理（流程不会执行其中嵌入的任何指令，也不会抓取正文里的链接），但这类 agent 防护是在指令层面生效的，不是沙箱——在陌生的招聘网站上，发送前请先扫一眼抓取回来和写出去的内容。细节见 [SECURITY.md](SECURITY.md)。

## 其他命令

`/setup`、`/scrape`、`/apply` 构成核心流程。资料配置好之后，还有十个命令可以扩展它：

- **`/interview`** 为已跟踪申请中某场已约好的面试做准备。它从该申请的归档里（原始职位描述、面试官实际读到的简历与求职信、前几轮记录的反馈）生成针对该阶段的准备包，按「先核实再使用」的规则调研公司和面试官，把可能被问到的问题映射到你的 STAR 案例上，并可按 `07-interview-prep.md` 里的角色扮演协议陪你做模拟面试。遇到短板会给诚实的过渡式回答，绝不编造经历。
- **`/outcome`** 记录一份申请后来怎么样了——面试轮次、offer、拒信、杳无音信。它把提交过的简历、求职信和职位描述归档到 `documents/applications/<company>_<role>/`，用 `/setup` 路径 A 能解析的格式维护 `outcome.md`，并更新跟踪表。在还没有结果可记的时候它也有用：`/outcome followup` 会把长时间没有回音的申请挑出来（默认 10 天），只用你已提交材料里的事实、按你的写作风格起草一封简短的、适合该渠道的跟进信（只起草，从不发送；每份申请最多两次），并在记录某个面试轮次的同一轮里顺手提议写封感谢信。等几份申请有了结果，它会引导你回到 `/setup`，用真实拿到面试的经历去校准匹配框架。
- **`/notion-sync`** 通过官方 Notion MCP 服务器（OAuth，无需 API key）把流水线单向、只读地发布到 Notion 数据库——每个已打分的职位一行，外加每一份已跟踪的申请，每行配一个只写一次的简报页面。仓库文件仍是唯一事实来源：没有任何数据回写，文档只同步文件名。它和 `/html-report` 互补：后者是你在工位上重新生成的离线深度仪表盘，前者是随时随地（桌面端、网页、手机）能瞄一眼的实时视图。
- **`/gmail-sync`** 读取你的 Gmail（通过 Gmail connector）来获取未结申请的进展信号——面试邀请、测评链接、offer、拒信——并以批次形式提给你确认，在你批准之前不会写入跟踪表或 `outcome.md`，每条建议的改动都会标注来源邮件。offer 环节不会擅自提议 `hired`/`offer_declined`，因为那该由你决定；互相冲突或匹配不上的信号会被标出来，交给你手动跑一遍 `/outcome`，而不是靠猜。
- **`/rank`** 衔接 `/scrape` 和 `/apply`：它把所有新抓到的职位按匹配框架批量打分（并行 agent 逐个抓取职位，给出五个评估维度的分数），返回一份排序清单，附上每个职位诚实的优势与短板。硬性排除项直接否决，临近截止的会有紧迫标记，失效职位会被标记为过期。选定一个编号，就交给完整的 `/apply` 流程。
- **`/expand`** 通过扫描你已经在资料里关联的公开来源（GitHub 仓库、作品集网站、Kaggle、Google Scholar），并为提到的课程和认证查找课程大纲，来充实你的资料。发现的能力会带上来源标签加进资料。刚跑完 `/setup` 之后用它很有用，可以把只靠文档看不出来的技能挖出来。
- **`/upskill`** 分析你的资料、已跟踪职位，以及已打分但未跟踪的职位（`/rank` 在 `job_scraper/seen_jobs.json` 里记录的短板）之间的差距——也可以用 `/upskill <URL>` 针对单个职位。产出一张按优先级排序的技能短板热力图，以及一份带联网检索到的学习资源和时间估算的学习计划。适合在两次申请之间做职业规划。
- **`/html-report`** 根据 `job_search_tracker.csv` 和申请归档生成一个自包含的 HTML 仪表盘——统计卡片、状态/行业/渠道/漏斗图表（内联 SVG，无外部依赖），以及一张可筛选的申请表。直接在浏览器里打开，完全离线。`/apply` 或 `/outcome` 新增条目后，随时可以重跑。
- **`/add-template`** 注册你自己的简历或求职信模板（LaTeX、Typst 或其他工具链），用来替代内置模板。它会记录模板的各项要求（源文件扩展名、编译命令、字体、样式规则、页数上限），执行一次强制的测试编译，并把模板接入 `/apply`。见下文[自定义模板](#自定义模板)。
- **`/add-portal`** 为你所在市场的招聘网站生成一个搜索 skill。它会研究该网站（搜索 URL 规律、结果页结构、访问规则），用与内置 skill 相同的结构搭出 CLI skill 骨架，并在注册前跑一次真实查询做测试。见下文[求职搜索工具](#求职搜索工具)。

另外还有 `/reset`，见下文[重新开始](#重新开始)。

## 文件结构

```
ai-job-search/
├── CLAUDE.md                          # 主候选人资料 + 工作流规则
├── .claude/
│   ├── commands/
│   │   ├── apply.md                   # /apply 流程（撰写—评审）
│   │   ├── setup.md                   # /setup 引导（documents 文件夹、简历导入或访谈）
│   │   ├── expand.md                  # /expand 从文档和线上足迹补充能力
│   │   ├── add-template.md            # /add-template 注册自定义模板（LaTeX、Typst 等）
│   │   ├── add-portal.md              # /add-portal 为你所在市场生成招聘网站搜索 skill
│   │   ├── rank.md                    # /rank 把抓到的职位筛选成排序清单
│   │   ├── outcome.md                 # /outcome 记录申请结果、归档材料
│   │   ├── gmail-sync.md              # /gmail-sync 从 Gmail 自动识别申请状态
│   │   ├── interview.md               # /interview 分阶段准备包 + 模拟面试
│   │   ├── html-report.md             # /html-report 生成申请跟踪仪表盘
│   │   ├── notion-sync.md             # /notion-sync 在 Notion 数据库中生成单向流水线视图
│   │   └── reset.md                   # /reset 清空资料数据或 documents 文件夹
│   ├── skills/
│   │   ├── job-application-assistant/  # 核心申请 skill
│   │   │   ├── SKILL.md               # skill 定义
│   │   │   ├── 01-candidate-profile.md # 你的教育、经历、技能
│   │   │   ├── 02-behavioral-profile.md# PI/DISC/性格测评
│   │   │   ├── 03-writing-style.md    # 语气、结构、该做与不该做
│   │   │   ├── 04-job-evaluation.md   # 职位匹配度评分框架
│   │   │   ├── 05-cv-templates.md     # LaTeX 简历结构 + 定制规则
│   │   │   ├── 06-cover-letter-templates.md # LaTeX 求职信模板
│   │   │   └── 07-interview-prep.md   # STAR 案例 + 面试框架
│   │   ├── job-scraper/               # 职位搜索编排
│   │   └── upskill/                   # /upskill 技能差距分析与学习计划
│   └── settings.json                  # Claude Code 权限（共享、限定范围）
├── .agents/skills/                    # 招聘网站 CLI 工具
│   ├── jobbank-search/                # Akademikernes Jobbank（丹麦）
│   ├── jobdanmark-search/             # Jobdanmark.dk（丹麦）
│   ├── jobindex-search/               # Jobindex.dk（丹麦）
│   ├── jobnet-search/                 # Jobnet.dk（丹麦，政府门户）
│   ├── linkedin-search/               # LinkedIn 公开职位列表（不限国家）
│   └── freehire-search/               # freehire.me 技术职位聚合（多市场，REST API）
├── cv/
│   └── main_example.tex               # moderncv LaTeX 模板
├── cover_letters/
│   ├── cover.cls                      # 自定义求职信 LaTeX 类
│   ├── cover_example.tex              # 示例求职信（结构参考 + CI 冒烟测试）
│   └── OpenFonts/                     # Lato + Raleway 字体
├── templates/                         # 经 /add-template 注册的自定义模板
│   └── README.md                      # 文件夹结构说明
├── documents/                         # 供 /setup 路径 A 和 /expand 使用的职业素材
│   ├── README.md                      # 文件夹结构说明
│   ├── cv/                            # 主简历（PDF 或 .tex）
│   ├── linkedin/                      # LinkedIn 资料导出（PDF）
│   ├── diplomas/                      # 学位证与成绩单
│   ├── references/                    # 推荐信
│   └── applications/                  # 过往申请记录（<company>_<role>/）
├── .github/workflows/ci.yml           # CI：LaTeX 冒烟编译、skill 检查、CLI 类型检查
├── salary_lookup.py                   # 薪资对标工具（自带数据）
├── tools/
│   ├── check_framework_version.py     # CI 检查：skill 文件变更时需提升 framework_version
│   ├── check_upstream_updates.py      # 预览上游更新会动到哪些个性化文件
│   ├── convert_salary_excel.py        # 把薪资 Excel 转成 JSON
│   ├── fetch_page.py                  # 针对 403 的 GET 专用浏览器头重试（遵守 robots.txt）
│   ├── install_portal_clis.py         # 安装所有已发现的招聘网站 CLI（bun install）
│   ├── lint_skills.py                 # CI 检查：skills、commands、settings.json
│   ├── posting_guard.py               # 完整性检查：剥离隐藏文本、标记形似注入的行
│   ├── robots_check.py                # 用 robots.txt 把关浏览器头重试
│   ├── security_guards.py             # CI 防护：权限 allow/ask/deny、skill 授权、gitignore、清单
│   ├── upstream_triage.py             # 把上游提交分成「值得看」与「大概可跳过」
│   ├── verify_layout.py               # 测量编译后 PDF 的版面（空洞、孤行、页脚碰撞）
│   ├── verify_pdf.py                  # 校验编译后 PDF 的页数与可提取文本
│   └── README_SALARY_TOOL.md          # 薪资工具配置说明
├── job_scraper/                       # 抓取状态（已见职位、结果）
├── gmail_sync/                        # /gmail-sync 状态（已处理消息 ID、上次同步日期）
├── upskill/                           # /upskill 报告输出（每次运行一份 markdown 报告）
├── job_search_tracker.csv             # 申请跟踪表
└── SETUP.md                           # 详细配置指南
```

## `/apply` 是怎么工作的

`/apply` 执行一套**撰写—评审流程**，并强制编译 PDF：

1. **解析**职位描述（URL 或文本）
2. **评估匹配度**：对照你的资料（技能、经验、文化、地点、职业方向）
3. **起草**量身定制的 LaTeX 简历与求职信
4. **启动评审 agent**，由它调研公司并对草稿挑刺
5. **修订**：根据评审意见修改
6. **编译并检查**两份 PDF：简历用 lualatex，求职信用 xelatex。Claude 会读取渲染后的页面并在 LaTeX 上反复调整，直到简历正好 2 页且没有孤立的条目标题、求职信正好 1 页且签名可见、字体一致。
7. **简历 ATS 检查**：提取 PDF 的文本层（`pdftotext`，可选依赖），按 ATS 解析器看到的样子核验——联系方式是否为字面文本、有没有乱码字形、阅读顺序是否正常——再把职位描述的关键词覆盖率跟提取结果对照打分。资料真正支持的关键词会补进去；真实的短板保持可见，绝不硬塞。
8. **交付**最终成品，附一份核验清单

简历和求职信里的每一项陈述，都对照你的真实资料核验过。系统从不编造技能或经历。

### 这套流程的不同之处

- **PDF 验证闭环。** 大多数 LaTeX 简历模板产出的东西在 `.tex` 里「看着没问题」，到 PDF 里就崩：职位标题孤零零掉到下一页、求职信溢到第 2 页、项目符号字体悄悄退回正文字体。`/apply` 会编译并逐页目视检查每一份 PDF，并施加针对性修复（`\needspace`、`\enlargethispage`、给列表项套字体匹配包装），直到版面干净为止。每次申请都会自动执行。
- **针对 PDF 文本层的 ATS 验证。** ATS 读的是 PDF 内嵌的文本，不是渲染后的页面——而 LaTeX 可能悄无声息地产出文本层是乱码的 PDF（本该是邮箱的位置变成图标字形、多栏版式导致行交错）。`/apply` 会用 `pdftotext` 提取编译后简历的文本层，按解析器真正看到的内容核验联系方式、阅读顺序，以及职位关键词的覆盖率。诚实原则是硬性的：资料不支持的关键词会被承认为短板，绝不硬塞。
- **按相关性加权的简历删减。** 简历超过 2 页时，流程不会机械地从「最老」的段落开始砍。它给每个候选行按三项打分：(a) 与目标职位的相关性，(b) 在文档中的独特性，(c) 求职信是否依赖它，然后先砍总分最低的那一行。一条命中职位关键词的早期岗位要点，会排在一条没命中的近期岗位要点前面留下来。
- **撰写与评审分离。** 撰写者负责写；另一个带全新上下文的 Claude agent 负责调研公司并对草稿挑刺，之后撰写者再修订。这能抓住漏掉的关键词、偏弱的表述和泛泛的套话——这些单跑一遍往往就留下了。
- **省 token 的评审调度。** 评审 agent 直接以行内形式收到草稿，不必重新读取；核验清单在流程末尾只跑一次，而不是两个 agent 各跑一遍。注意：第 5 步新增的编译检查会花掉其中一部分节省——流程用一些端到端 token 成本，换来真正减少坏 PDF 递交到用户手里的情况。

## 自定义

### 哪些文件需要手动修改

如果你更愿意直接改文件、不用 `/setup`：

| 文件 | 改什么 |
|------|--------|
| `CLAUDE.md` | 你的完整资料（姓名、教育、经历、技能、目标） |
| `01-candidate-profile.md` | 你简历数据的结构化版本 |
| `02-behavioral-profile.md` | 你的行为风格测评或自我评估 |
| `04-job-evaluation.md` | 技能匹配范围、职业目标、动机筛选条件 |
| `05-cv-templates.md` | 针对不同岗位类型的自我陈述模板 |
| `07-interview-prep.md` | 来自真实经历的 STAR 案例 |
| `search-queries.md` | 针对你的技能和地点设置的求职搜索查询 |

### 更新搜索条件

随着你的优先级变化，你可以只重配职位搜索，而不用重跑整套资料配置：

```
/setup --section search
```

这会重跑搜索配置访谈：目标哪些岗位、搜哪些技能、哪些地点、哪些招聘网站。它还会根据你的资料，推荐一些你可能没考虑过的岗位类型。

### 自定义模板

简历用的是 [moderncv](https://ctan.org/pkg/moderncv)（banking 样式），求职信用自定义的 `cover.cls` 加 Lato/Raleway 字体。两者都是 LaTeX——这是本仓库自带并维护的参考引擎。

想换成自己的模板——LaTeX、[Typst](https://typst.app/)，或任何能在命令行编译出 PDF 的工具链——运行：

```
/add-template
```

指向你的源文件（一个 `.tex` 文件加上相关的 `.cls`/`.sty` 文件和捆绑字体；或一个 `.typ` 文件加上本地包；或其他工具链的对应物）。命令会问你模板的相关信息——源文件扩展名、编译命令、字体及其位置、需要保留的样式规则、硬性页数上限——把一切存到 `templates/` 下，跑一次强制的测试编译，然后启用该模板，让 `/apply` 基于它起草和编译。模板用 `[PLACEHOLDER]` 占位符代替个人数据存储，所以可以放心提交和分享。

- `/add-template --list` 列出已注册的模板
- `/add-template --use <name>` 在它们之间切换
- `/add-template --use default` 回到内置的 moderncv / cover.cls 模板

更喜欢手动操作？手动路线照样可行：直接改 `05-cv-templates.md` 和 `06-cover-letter-templates.md` 里的说明。

### 求职搜索工具

`.agents/skills/` 里的四个丹麦 CLI 工具（Jobbank、Jobdanmark、Jobindex、Jobnet）演示了如何为特定市场构建招聘网站集成。如果你在别的国家，运行：

```
/add-portal
```

给它你本地招聘网站的 URL。命令会研究该网站（搜索 URL 规律、结果页结构、robots.txt/访问规则），用相同的结构、命令和输出约定搭出 CLI skill 骨架，并在注册任何东西之前跑一次真实查询做测试。需要登录的网站会被拒绝；条款严格的网站，会在生成的 skill 里加上醒目的「仅供个人使用」警告。生成的 skill 与你所在市场相关，放在你自己的 fork 里；生成器本身才是通用的那部分。

维护了一个适配你所在市场或语言的 fork？把它加到[社区 fork 与适配](https://github.com/ai-job-search-dev/ai-job-search/discussions/78)帖子里，方便别人找到。

除了丹麦的演示，仓库还附带两个**不依赖国家**的入门网站 skill：

- **`linkedin-search`** —— 基于 LinkedIn 公开、免登录的 `jobs-guest` 接口。不限领域，**零运行时依赖**（只要有 `bun` 就能跑），搜索地点用显式参数传入，所以开箱即可用于任何市场（`-l "Berlin, Germany"`、`-l "Mumbai, Maharashtra, India"`、`-l "Remote"`……）。**仅供个人使用**——自动化访问违反 LinkedIn 的服务条款，请控制频率。见 `.agents/skills/linkedin-search/SKILL.md`。
- **`freehire-search`** —— 查询 [freehire.me](https://freehire.me) 聚合器的公开 REST API（JSON，无需 API key）。偏技术岗（软件、数据、工程、DevOps、远程），通过维度参数支持多市场（`--region`、`--country`、`--remote`），**零运行时依赖**。与抓 HTML 的丹麦网站不同，它返回结构化结果（技能、资历、类别）。后端是 MIT 许可的，[可自托管](https://github.com/strelov1/freehire)——如果你愿意，把 `FREEHIRE_API_URL` 指向自己的实例即可。见 `.agents/skills/freehire-search/SKILL.md`。

### 扩展这套框架：网站、模板、评估标准——以及从别的 fork 借用

上面这些汇总成一个扩展模型，这里直说：框架有三个扩展点，都不需要碰上游代码：

1. **网站 skill** —— 招聘网站层面的模块系统。每个 `*-search` skill 都是 `.agents/skills/` 下一个自包含的文件夹，遵守同一套约定（`search`/`detail` CLI、`--format json|table|plain` 输出、`SKILL.md` 里的 `enabled:` 开关、自带测试）。`/scrape` 会自动发现任何遵守该约定的已安装 skill——不用注册，不用接线。`/add-portal` 负责生成新的；[社区网站索引](https://github.com/ai-job-search-dev/ai-job-search/discussions/78)收录了其他 fork 做的那些。
2. **文档模板** —— `/add-template` 可以注册任何能在命令行编译出 PDF 的简历或求职信工具链，LaTeX 也好，别的也罢。
3. **评估标准** —— 资料里的硬性排除项和偏好都是自由文本，评估量表就按你写的东西打分。「产假条件要好」「薪资不低于我工会的 X 档」「不接受 on-call」——每条只是资料里的一行字，不用写代码，而且在 `/rank` 和 `/apply` 的匹配度评估里权重实打实。语言是唯一有专门结构化处理的排除项类型：`/setup` 会把你工作中用到的每种语言和对应水平（直接询问，或从简历/LinkedIn 导出中推断）记录到一张 `Languages` 表里，语言门槛（`04-job-evaluation.md`）会硬性拒绝要求一门你完全没申报过的语言的职位，而对「你确实会说、但它要求的水平高于你申报水平」的情况只做标记、不自动拒绝，这样边界情况（比如对方卡死「流利」，而你是 B1/B2）交给你自己判断，而不是被悄悄丢掉。

**从别的 fork 借用一个网站 skill**，是获得上游没有的招聘网站的正规做法：在[网站索引](https://github.com/ai-job-search-dev/ai-job-search/discussions/78)里找到它，打开那个 fork，把那一个文件夹拷进自己的 `.agents/skills/`。运行之前：

- **读代码。** 全部读完——这些 CLI 在你机器上是预授权的（`.claude/settings.json` 里放行了），直接对着你的职业数据跑。检查网络请求是否只发往它声称要搜的招聘网站，`package.json` 里是否没有 `dependencies`、没有生命周期脚本（`postinstall` 之类），以及有没有读写自己文件夹以外的位置。
- **离线跑它的测试**（在 skill 的 `cli/` 目录里 `bun test`）——写得好的 skill，测试不联网也能通过。
- 看一下 `enabled:` 开关和 skill 自带的条款说明。

拷贝步骤故意设计成手动的。你的设置已经允许已安装的网站 skill 免确认运行——所以一个替你从第三方仓库拉取它们的安装器，恰好会跳过最关键的那道检查：由你先读代码。没有这样的安装器，这是一个安全决策，而不是缺了个功能。

与市场相关的**数据源**（某个国家的薪资数据库、地方性薪酬档位表）和招聘网站遵循同样的模式：它们属于对应市场的 fork，通过 [#78](https://github.com/ai-job-search-dev/ai-job-search/discussions/78) 分享，而不是放进上游。

### 薪资对标

薪资工具可以搭配你提供的任何薪资数据使用（工会统计、Glassdoor 导出、自己调研的结果等）。格式要求和配置方式见 `tools/README_SALARY_TOOL.md`。没有薪资数据也没关系，这一步会直接跳过。

### 重新开始

要清空你的资料数据重新开始：

```
/reset profile    # 清空 skill 文件，保留框架规则
/reset documents  # 删除 documents/ 文件夹里的文件
/reset all        # 两者都清空
```

`/reset` 会明确列出将要删除什么，并要求你输入 `RESET` 确认。在你输入之前，什么都不会删。

### 保持更新

上游迭代很快。与其直接拉 master 碰运气，不如把 fork 更新到一个打上标签的 [release](../../releases)——那是经过审核的检查点，[CHANGELOG.md](CHANGELOG.md) 里有说明。`python3 tools/check_upstream_updates.py` 会在合并前精确预览一次更新会动到你哪些个性化文件，`python3 tools/upstream_triage.py` 则把你落后于上游的那些提交分成「值得看」和「大概可跳过」（每周跑一次，可以把结果发到一个滚动 issue 里）。完整操作见 [SETUP.md 第 8 节](SETUP.md#8-pulling-upstream-updates-into-your-fork)。

## 提升效果的小建议

### 资料详实度决定成败

输出质量最大的单一影响因素，是你在资料里写了多少细节。资料单薄，申请就泛泛；资料翔实，才能真正量身定制。

- **岗位描述：** 别只列职位名。写清楚你在每个岗位上实际做了什么：具体项目、用到的工具、职责、可量化的成绩。你给的素材越多，系统就越能针对不同岗位精确地重构你的经历。
- **让技能带上场景：** 与其干巴巴写「Python」或「项目管理」，不如描述你是怎么用、在哪用的。「用 Python 和 scikit-learn 搭过客户流失预测的 ML 流水线」比「Python、机器学习」能提供多得多的工作素材。
- **三条引导路径殊途同归：** 不管你是把 `/setup` 指向 `documents/` 文件夹、粘贴一份简历，还是一路走访谈，原则都一样：输入越丰富，输出越锋利。

### 职业路径探索

这套框架支持两种不同的求职模式：

- **明确目标型：** 你清楚想做什么岗位或行业。系统帮你按匹配度细化和排序。
- **潜在机会发现型：** 通过分析你的完整履历（不只是职位名，还有你实际做的工作），系统能挖出你没想过的职业路径。可迁移的技能能对应到意想不到的行业；你享受过或擅长的事情背后有规律；还有结合你领域专长与新技术的新兴岗位。

想把这部分用足，就在 `/setup` 里花点时间，不光描述经历，还要说清什么让你有劲、什么让你心累、你希望多做什么。这些背景直接决定了系统怎么评估匹配度，以及在 `/scrape` 时给你端出哪些岗位。

## 参与贡献

想提 PR？先读 [CONTRIBUTING.md](CONTRIBUTING.md)——它讲清了什么会被合并、什么应该留在 fork 里，以及为什么。

## 致谢

- [Mikkel Krogholm](https://github.com/mikkelkrogsholm)（[skills 仓库](https://github.com/mikkelkrogsholm/skills)）提供了求职搜索 CLI skill
- 由 [Anthropic](https://anthropic.com) 的 [Claude Code](https://claude.com/claude-code) 构建

## 许可

MIT
