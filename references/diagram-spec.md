# Diagram Spec — 三图规范与验收 / The three diagrams

A 级讲解固定三张图。画法以 vendored diagram-design 技能为权威（先读它的
SKILL.md §6 连线规则、§7 复杂度预算，再按图型读对应 `references/type-*.md`）；
本文件是加速版摘要 + 本引擎的验收标准。发生冲突时以 diagram-design 原文为准。

路径：安装后为 `~/.agents/skills/explain-hub/third-party/diagram-design/`，
vendor 源码目录为 `third-party/diagram-design/`。

## 三张图画什么 / Content briefs

| 图 | 图型（diagram-design 术语） | 回答的问题 | 内容要点 |
|---|---|---|---|
| 功能全景图 | Architecture（分区+模块） | 它能做什么 | 按能力域分 ≤3 个 zone，共 ≤9 个节点；节点副标签列具体功能与 `文件:行号`；珊瑚焦点给招牌能力 |
| 模块架构图 | Architecture | 代码怎么分工 | 按 进程/层/信任边界 分区（如 渲染层/主进程/服务端 或 前端/后端/内核）；箭头标通信方式（HTTP/IPC/require） |
| 运行流程图 | Flowchart（可加泳道） | 核心功能怎么跑 | 端到端主链 ≤9 节点：oval 起点、rect 步骤、diamond 判定（≤3 出口，每条出口有标签）、merge dot 汇合；珊瑚只给主循环/最关键判定 |

选图型前先读 diagram-design SKILL.md §3 的视觉类型表；拿不准就 Architecture。

## 铁律速查 / Non-negotiables（详见 diagram-design §6）

1. 直角连线 `r=8` 圆角肘线；端点同 x/y 才可用直线；**对角线自动不合格**。
2. 箭头标签放在纸色蒙版 rect 上，蒙版与线留 **6-10px** 可见间隙；绝不压线、不竖排。
3. 不重叠、不共线；实线交叉处给**次要的一条**加 `a 8,8` 半圆桥；虚线跨实线必须桥。
4. 同一盒子同一边的多个 attach 点彼此错开 ≥12px。
5. 连线不穿非端点盒子（不可避免的横切场景：虚线+标签在可见端）。
6. 复杂度预算（§7）：≤9 节点 / ≤12 箭头 / ≤3 分区 / ≤2 珊瑚元素；超了拆"总览+细节"两图。
7. 页面骨架：eyebrow + Instrument Serif 标题 + 副标题；`<svg role="img" aria-labelledby>` 带**前缀化** `<title>/<desc>`；图例横排在底部发丝线下（绝不飘在图里）；Google Fonts 引 Instrument Serif / Geist / Geist Mono / **Noto Sans SC**（中文标签回退）。
8. viewBox 顶部留白过大时直接裁剪（如 `viewBox="0 76 1180 614"`），比挪坐标安全。

## 页面输出 / Deliverable

- 每图一个**自包含 HTML**（内嵌 CSS+SVG，仅 Google Fonts 外链），命名
  `功能全景图.html` / `模块架构图.html` / `运行流程图.html`，与 README 同目录。
- 配色用 diagram-design 默认皮肤：paper `#f5f5f5`、ink `#2d3142`、muted `#4f5d75`、accent `#eb6c36`（≤2 个焦点元素）、rule `rgba(45,49,66,.10)`。
- figcaption 注明分析基线（commit / 本机副本路径），footer 注明
  `GENERATED WITH DIAGRAM-DESIGN · 日期`。

## 渲染 / Rendering（Windows/Git Bash 实战链）

1. **URL 必须百分号编码**（中文路径 + 空格会让直连失败）：
   `python -c "from urllib.parse import quote; print(quote('功能全景图.html'))"`
2. **优先 msedge 通道**（Chromium 官方下载在受限网络常失败，且本机 Edge 必在）：
   ```
   npx playwright screenshot --channel msedge --viewport-size=1280,900 --full-page "<file-url>" _panorama.png
   ```
3. msedge 不可用再 `npx playwright install chromium`（可能需要代理）。
4. 三张自检图命名 `_panorama.png` / `_architecture.png` / `_flow.png`，留在交付目录里当预览。

## 验收 / Acceptance rubric（自检与视觉代理共用）

逐页检查，fail 只判用户肉眼可见的破损：

- 文字：无截断/重叠/tofu（CJK 乱码），标签未溢出盒子
- 连线：无对角线；标签不压线（蒙版留隙可见）；虚线跨实线有半圆桥；无悬空箭头（终点必须落在盒子边缘）；无两条线共用 attach 点
- 构图：分区完整包住节点；图例横排底部；珊瑚 ≤2；无阴影/紫青霓虹
- 一致性：页眉/页脚/figcaption 齐全；多页之间骨架一致

修复后**只复检改过的页**，并在复检 prompt 里写明每处修复内容。
