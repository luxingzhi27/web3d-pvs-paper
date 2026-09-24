# 论文配图

图片由作者后续上传。本目录目前只提供文件命名与插图说明，不包含图片文件。

## 文件命名

| 图号 | 文件名 | 内容 |
|---|---|---|
| 图 1 | `fig01-teaser.png` | 同一传输预算下的应用效果概览 |
| 图 2 | `fig02-overall-framework.png` | 已制作的整体架构图 |
| 图 3 | `fig03-survival-field.png` | 已制作的方向生存场架构图 |
| 图 4 | `fig04-visibility-results.png` | 单元可见性定性比较与困难案例 |
| 图 5 | `fig05-runtime-scaling.png` | 候选规模与客户端查询延迟 |
| 图 6 | `fig06-streaming-coverage.png` | 渐进式可见贡献覆盖曲线 |

整体架构图和生存场架构图分别上传为 `fig02-overall-framework.png` 和 `fig03-survival-field.png`。采样与后退视锥示意图暂不纳入正文，不占用上述编号。

## 在大纲中显示

[总体大纲](../../docs/00-overall-outline.md)已包含文字占位、图面内容和图注，并为每幅图预留注释中的 Markdown 图片标签。图片上传后，取消对应标签的 HTML 注释即可显示。

例如，图 2 上传后的有效标签为：

```markdown
![图2 GCOF-PVS整体架构](../paper/figures/fig02-overall-framework.png)
```

该相对路径用于 `docs/00-overall-outline.md`。采用其他图片格式时，同步修改文件扩展名与标签路径。

## 图稿约定

图号与标题由正文图注统一管理。符号、连线语义、配色与各图内容见[图表目录](../../docs/09-figures-tables.md)。现有架构图作为图稿，上传时以最新目录中的数学符号和数据流约定为准。

实验图由正式数据产生，不以示例数值填充。用于英文稿的英文图注在正文撰写阶段独立整理。
