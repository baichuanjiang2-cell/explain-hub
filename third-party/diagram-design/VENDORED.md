# Vendored: diagram-design

本目录是 **diagram-design** 技能的完整 vendored 副本（v2.6，MIT 许可），
作为 explain-hub 的图表绘制引擎。

## 来源

本副本取自 skills 市场安装的本地副本（安装时未携带 git 元数据，无法确定
上游 commit）。未做任何修改。查找上游更新：搜索 "diagram-design agent
skill editorial SVG" 或在技能市场检索 diagram-design。

## 为什么 vendor

explain-hub 的深度讲解强制要求三张 diagram-design 风格图表（功能全景 /
模块架构 / 运行流程）。打包副本保证：

1. 离线/受限网络环境下也能安装（运行时从本目录复制到用户技能目录）；
2. 图表质量基线一致（diagram-design 的连线规则与复杂度预算是验收标准）。

## 更新

 prefer 使用你机器上已安装的更新版本——explain-hub 运行时会优先探测
本地已装的 diagram-design，找到即用那份；本目录只是兜底安装源。

## 许可

diagram-design 以 MIT 许可发布，版权归其原作者所有。本 vendored 副本
遵循同一许可；explain-hub 对该目录的修改（如有）同样以 MIT 开源。
