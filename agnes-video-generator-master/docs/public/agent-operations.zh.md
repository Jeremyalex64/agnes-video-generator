# 🤖 让 AI Agent 来操作本项目

> 本文是"用编程 Agent 驱动 Agnes Video Generator"的**权威说明**。官网的相关指南与 wiki 页面都由本文摘要而来——若两者与本文不一致，以本文为准。

Agnes Video Generator 从设计之初就是给 AI 编程助手用的。接入方式有三档，你可以停在任意一档：

| 档位 | 你能得到什么 | 需要做什么 |
|------|--------------|------------|
| **1. 把 Agent 指向本仓库** | Agent 能部署项目、创建任务、观察进度、排查失败 | 什么都不用做——仓库自带 `AGENTS.md` |
| **2. 装上伴随技能库** | Agent 不必先"现学"这个项目：三个现成技能覆盖使用、排障、贡献 | 一条命令（见下文） |
| **3. 用 flint 分发** | 你在用的每个 Agent 都装上同一套技能，且始终同步 | 一条命令（见下文） |

---

## 第一档 —— 把 Agent 指向本仓库

仓库自带为 Agent 而非为人写的 [`AGENTS.md`](../../AGENTS.md)，内容包括：环境要求、四层部署验证清单、
共享约定（日志前缀、重试模型、i18n 规范），以及修 Bug、跑回归、发版的工作流。

把项目 clone 下来，然后告诉你的 Agent：

> 阅读这个项目的 `AGENTS.md`，安装依赖，配置 API Key `<your-key>`，然后启动服务。

接着确认部署是真的可用，而不只是"进程起来了"：

> 跑一下部署验证检查。

Agent 会按四层验证清单逐项执行（连通性 → 静态分析 → 端点测试 → 功能检查）并汇报每一项结果。

**这已经很好用，但有一个限制**：`AGENTS.md` 住在仓库里，所以 Agent 必须能读到仓库才能用；
而且它既讲怎么*改*这个项目，也讲怎么*操作*它。

---

## 第二档 —— 装上伴随技能库

伴随技能库把日常操作打包成三个技能，Agent 不需要先读 30KB 的文档就能上手：

> **仓库：[lcy362/agnes-video-generator-skills](https://github.com/lcy362/agnes-video-generator-skills)**

| 技能 | 什么时候用 |
|------|------------|
| **`agnes-video-usage`** | 从零到成片：选部署方式、配 Key、在六种任务类型里选型、提交任务并观察进度，以及用手动模式检查点让 Agent 直接改写分镜、旁白、字幕或某个场景视频后再继续 |
| **`agnes-video-troubleshoot`** | 查清一次失败的真正原因：队列饱和、`401` 域名不匹配、超时、分辨率不对、竖屏画面躺倒、续传后字幕错乱——并判断它属于配置问题、上游容量问题，还是值得动手修的缺陷 |
| **`agnes-video-contrib`** | 改本项目的代码并让 PR 通过 CI：英文 commit 铁律、改前端必须重构建、i18n、测试与大版本回归 |

每个技能都是"决策树 + 小脚本"（`agnes_task.py` 建任务/轮询/查产物、`verify_deploy.sh` 部署验证清单、
`diagnose_task.py` 取证、`check_media.py` 校验分辨率与方向、`preflight.sh` 本地 CI 门禁）。
它们只有文档与工具，**不含服务代码，也不经手任何密钥**。

**推荐：用 flint 安装。** [**flint**](https://github.com/lcy362/flint) 是一个本地优先的开源技能管理器：
把你的技能目录集中在一处，再通过 symlink 或复制投放到任意 Agent 目录与项目目录。只需把这份技能库**登记为来源一次**，
三个技能就会出现在你用的每个 Agent 面前——新增 Agent 不用再拷一遍，换个项目也不用重来。

先把仓库 clone 下来。flint 只认**本地**源——它不会替你联网拉任何东西；所谓"刷新"就是读你指定的那份
git 检出，且只在你主动点的时候才做：

```bash
git clone https://github.com/lcy362/agnes-video-generator-skills.git
npx flint-skills-hub          # 打开本地界面 http://localhost:8787
# Sources → 添加刚 clone 下来的目录；flint 会从它的 skills/ 子目录里找到这三个技能
```

它内置支持数十个 Agent（Claude Code、Cursor、Codex、TRAE 等），并且完全本地运行：无云端、无账号、无上报。
技能始终是你磁盘上的普通目录，**即使以后不用 flint 了也照常可用**。**[去 GitHub 给 flint 点个 Star ⭐](https://github.com/lcy362/flint)**

**替代方案：手动拷贝。** 不用 flint 也行——同一套技能就是普通目录。代价是每个项目都要来一次，
每上一个新 Agent 也要再配一遍：

```bash
git clone https://github.com/lcy362/agnes-video-generator-skills.git /tmp/agnes-skills
cp -R /tmp/agnes-skills/skills/* .agents/skills/     # 任何读 .agents/skills 的 Agent 都能用
```

装好之后，向 Agent 描述**结果**而不是命令：

> 我的任务 4de1bcd26fda 失败了——查清原因，并告诉我值不值得修。

> 创意视频出来是无声的。我要旁白和字幕——配好并跑一遍。

> 拿我那个已完成的任务，把分镜提示词优化一下，然后只重跑受影响的部分。

---

## 第三档 —— 一套技能，所有 Agent

第二档解决了"装上"；这一档说的是它为什么持续划算：

- **你会换 Agent。** 今天 Cursor、明天 Claude Code、下周 TRAE——flint 把同一套技能投放到各自目录，不用手工再拷。
- **你有多个项目。** 项目级技能放在 `<项目>/.agents/skills/`，可以提交进 git，全组都能用上。
- **你的技能会变多。** 技能可以打标签、组成预设，"agnes 那几个"就成了一个预设，新 Agent 一键应用。

---

## Agent 最能帮上忙的地方：手动模式

手动模式会在检查点暂停流水线，让 Agent 先改产物再继续。Agent 读任务的 `manifest.json`、就地编辑文件、
预演哪些下游步骤会重跑，然后确认继续——于是"润色旁白""修字幕"变成一次交代，而不是来回复制粘贴。

详见：[手动模式指南](manual_mode_guide.md) · 产品说明：[使用指南](usage.zh.md)。

---

## 边界

| 载体 | 用途 |
|------|------|
| `AGENTS.md` | 用于**修改本项目代码**——约定、验证、发版流程 |
| 伴随技能库 | 用于**操作本项目**——部署、生成、改产物、排障、贡献 |
| 本文 | 两者之上的权威说明，官网与 wiki 均引用本文 |

用 Web UI 或在线 Studio 不需要以上任何一样——Agent 路线是给"宁愿描述结果、不想敲命令"的人准备的。

---

## 相关文档

- [部署指南](getting-started.zh.md) —— 其中的"方式 D"是本文的最短形式，与本文保持同步
- [使用指南](usage.zh.md) —— 模式、检查点、配置
- [手动模式指南](manual_mode_guide.md) —— 检查点与外部 Agent 协作协议
- [API 参考](api.zh.md) —— Agent 所驱动的那些端点
- [常见问题](faq.zh.md) —— 含"多个 Agent 之间技能怎么保持同步"
