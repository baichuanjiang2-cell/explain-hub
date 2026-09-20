# Playbook — 实战规则与踩坑清单 / Field-tested rules

本文件是 explain-hub 的经验库：每条规则都来自真实批量讲解中踩过的坑。
按需查阅；画图相关的内容在 [diagram-spec.md](diagram-spec.md)。

## 1. 数据获取降级链 / Getting the stars list

1. `gh` CLI 常常不存在——不要假设它可用。
2. 首选公开 API：`https://api.github.com/users/<name>/starred?per_page=100&page=N`（未认证限额 60 次/小时，够拉列表；翻页直到空数组）。先 `GET /users/<name>` 校验账号存在。
3. Windows 注意：用 python 解析 JSON 时临时文件要放在工作区相对路径，`/tmp` 对 Windows 原生 python 不可见。
4. API 不可达 → 请用户手动导出（GitHub → Your stars 页面复制），不要卡死。
5. 拉完把临时 JSON 删掉，别留在工作区。

## 2. 子代理编排与限流 / Sub-agents & rate limits

- 大仓库分析派子代理（保住主上下文），prompt 必须自带完整模板路径、输出目录、`file:line` 锚点要求和"不确定就明说"的指令，并要求**最终回复保持简短**。
- 并行子代理会撞模型限流（表现为 `[1302]` 速率错误、任务静默失败）。**其他代理跑完后重派失败的即可**，不是致命错误。
- 给子代理的克隆指令：`--depth 1`，失败重试 2 次，仍失败降级为 GitHub API（`api.github.com` + `raw.githubusercontent.com`）分析并在 README 注明。
- 克隆目录在**视觉验收全部通过之前不要删**——修复图表时还要对照代码。

## 3. 门面仓库识别 / Facade repos

星数不等于内容。动手分析前先花一分钟验证：

- `main` 分支是否只有个位数提交、几十个文件？（某 90k★ 仓库 main 实为从零重写的空脚手架，真实代码在被归档的 sibling 仓库）
- README 宣传的功能在目录里有没有对应实现目录？
- 发现门面时：找到真实代码库（archived 分支 / `-classic` 后缀兄弟仓库 / 官方链接），**讲真实代码**，并在 README 和索引里写明"主仓库为脚手架，讲解基于 X"。

## 4. 文档与代码漂移核查 / Doc-vs-code drift

README 和 SKILL.md 会撒谎（过时、宣传口径）。对要写进讲解的关键声明：

- 声称的 CLI 子命令 → `grep argparse` / 入口文件确认真的存在。
- 声称的数量（模板数/预设数/支持工具数）→ 按目录或枚举实测，两个口径都写。
- 声称的版本/星数 → 能用 API 核实就核实（`api.github.com/repos/<o>/<r>` 返回 `stargazers_count`），不能就写"沿用任务给定值，未独立核实"。
- 漂移本身就是讲解素材：单独写进"注意事项"。

## 5. A/B 分级启发式 / Tiering heuristics

给 A 级（全量三图）的信号，满足越多越优先：

- 是本机已装技能/插件的上游仓库（两边讲解互相引用，价值翻倍）；
- 星数高（工具生态位明确）；
- 本地已有副本（省一次克隆）；
- 用户实际会用的核心工具（代理客户端、编辑器、生产力工作台）。

B 级（一页纸）兜底其余全部。分级是**临时**的：批量清单里标好 A/B，用户随时可升级单个 B。上限默认 25 个 A，超了就让用户砍或提高上限。

## 6. 批量确认节奏 / Batch cadence

- 先给全量清单让用户对**范围和深度**拍板（一次问清，不逐项骚扰）。
- 执行期每批 5–8 项：出清单（✅建议/⬜可选 + 推荐理由）→ 用户勾选 → 连续做完 → 下一批。
- 两个任务线（技能/stars）可混勾；用户说"全做/按推荐来"就整批执行，不再打断。

## 7. 索引与交叉引用 / Index & cross-references

- 每个输出目录一份 `README.md` 总索引：篇目表 + 状态列（⬜待做/🖊️写作中/✅完成/❌失效），完成一篇改一篇。
- skill 同源仓库 ↔ 本机技能讲解**双向链接**；同一赛道项目在"注意事项"里一句话对比（如"X 走本地 SDK，Y 走 HTTP 服务化"）。
- 失效技能记录成表（名称/症状：空目录或断链/修复指引），这是用户最意外的收获之一。

## 8. 模式 B 专项 / Installed-skills mode specifics

- 技能目录是**软链接农场**：`~/.zcode/skills` 与 `~/.agents/skills` 大量互指，真实副本在 `~/.skills-manager/skills`、`~/.codex/skills`（含 `.system` 内置）和插件缓存。用 `readlink` 逐个解析，按真实路径去重。
- `ls -F` 带 `@` 是软链；`*/` 通配会**跳过断链**——用 `readlink -e` 判断目标是否存在。
- 目标目录存在但为空 = 安装损坏（空目录）；readlink 为空且目标不存在 = 断链。两类都记入失效表。
- 有的 SKILL.md 不在加载列表里（frontmatter 缺失/禁用），照样值得讲——以磁盘内容为准。
- 成系列的去重合并（同前缀），一篇总览讲清成员分工与配合链。

## 9. 渲染与视觉验收 / Rendering & visual QA

- 截图链与常见坑（Chromium 下载失败、中文路径）见 [diagram-spec.md](diagram-spec.md) §渲染。
- 验收代理 prompt 要素：逐页 JSON 裁决（pass/fail + 带位置的问题列表）、设计体系清单（正交连线/蒙版标签/≤2 珊瑚/图例置底）、"minor: 前缀=不阻塞"、"fail 只判用户肉眼可见的破损"。
- **只复检改过的页**，把修复内容写进复检 prompt 让代理有的放矢。
- 常见 fail 模式速查：箭头标签压线（蒙版与线 6-10px 隙）、虚线跨实线无半圆桥、悬空箭头（终点没落在盒子边上）、两条线共用同一 attach 点、导出格式标签装反、流程图末节点脱节。

## 10. B 级一页纸模板 / Lightweight one-pager

- 资料来源：`raw.githubusercontent.com/<o>/<r>/main/README.md`（main 不行试 master/dev，或 `api.github.com/repos/<o>/<r>/readme` + `Accept: application/vnd.github.raw`）；全失败就用仓库元数据（description/topics/language）写并注明。
- 结构：`# 仓库名 — 一句话定位`；`## 它能做什么`（3-6 条，宣传语转述为事实）；`## 怎么上手`；`## 与你的关联`（与其他讲解的串引，没有硬扯就写适用场景）；`## 链接`。
- 25-45 行；文件名 `<owner>__<repo>.md`；语气克制，未提及的不写。
