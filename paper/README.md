# Paper Manuscript Workspace

当前阶段**先不写英文完整初稿**。

本仓库的工作顺序是：

1. 先用中文把 `docs/*.md` 中每个章节的论证逻辑收敛；
2. 在 `docs/12-evidence-status.md` 中确认哪些 claim 已经有正式证据；
3. 等 Method / Related Work / Evaluation 稳定后，再在这里创建英文 LaTeX / Markdown 初稿；
4. 英文正文与中文规划文档分开维护。

## 推荐的英文初稿撰写顺序

不要先写 Abstract。

建议：

1. Related Work
2. Problem Formulation
3. Method
4. Evaluation
5. Introduction
6. Discussion
7. Conclusion
8. Abstract

原因：

> Introduction 和 Abstract 都依赖最终稳定的 novelty boundary 与实验结论。如果过早写，后面最容易反复返工。

## 未来建议目录

```text
paper/
├── README.md
├── main.tex
├── sections/
│   ├── introduction.tex
│   ├── related_work.tex
│   ├── formulation.tex
│   ├── method.tex
│   ├── system.tex
│   ├── evaluation.tex
│   └── discussion.tex
├── figures/
└── references.bib
```

在中文论证文档没有稳定前，暂不创建占位英文正文。
