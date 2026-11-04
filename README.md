# 《Rust 程序设计（第 2 版）》中文版 · LaTeX 工程

由 EPUB 转换、并按 O'Reilly 英文原版 *Programming Rust: Fast, Safe Systems Development,
Second Edition* 的 PDF 版式重排。

---

## 1. 目录结构

```
latex_source/
├── main.tex                    编译入口（10 行，只做 \input + subfile）
├── build.bat                   一键编译（三遍 + 清 minted 缓存）
├── README.md                   本文件
├── main.pdf                    产物
├── book/
│   ├── ccs.tex                 全书导言区：页面/字体/标题/代码/图表/提示框/页脚/章节命令
│   ├── oreilly-pygments.tex    从英文 PDF 采样的 Pygments 配色（70 个 \PYG 宏）
│   ├── index.tex               装配清单：封面 → 目录 → 前言 → 23 章 → 附录
│   └── content/
│       ├── front/              00-cover / 00-title / 00-copyright / 01-notice /
│       │                       02-oreilly / 03-endorsements / 04-translator /
│       │                       05-preface / 06-reviewers
│       ├── main/               chapter01 … chapter23（约 2.6 MB TeX 源码）
│       └── back/               90-authors / 91-cover
├── images/
│   ├── cover.jpg               封面
│   ├── icon-note.png           页边距里的 NOTE 图标（从原版 PDF 第 55 页抠出）
│   ├── ch01 … ch23/            190 张插图，按「图 x-y」自动改名 fig-x-y.jpeg
│   └── misc/                   其余零散图片
└── tools/
    ├── extract.py              EPUB → work/src/*.html + structure.json + 插图
    └── convert.py              XHTML → LaTeX（含 Pygments 预着色、表格网格、脚注排序）
```

上游数据在工程外：`../work/{epub,src,img,structure.json}`。

---

## 2. 怎么编

```bat
build.bat
```

或手动：

```bat
xelatex -interaction=nonstopmode main.tex     :: 需要跑三遍
```

依赖 **TeX Live 2026 + XeLaTeX**、**Python 3.9+（Pygments、lxml）**、字体
（见第 5 节）。默认路线不需要 `-shell-escape`；`build.bat` 里带着它只是为了
随时能切回 minted，留着无害。

三遍不是可选项：第一遍写 `.toc/.aux`，第二遍目录才有页码、书签才成形，
第三遍收敛 `\myChapter` 手工设定的章号与页脚中的章名。

---

> 如果环境配置遇到问题或者编译失败可以直接到[releases](https://github.com/jnulzl/Programming-Rust-2th-CN/releases)下载最新的pdf文件
