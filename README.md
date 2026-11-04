# 《Rust 程序设计（第 2 版）》中文版 · LaTeX 工程

由 EPUB 转换、并按 O'Reilly 英文原版 *Programming Rust: Fast, Safe Systems Development,
Second Edition* 的 PDF 版式重排。工程骨架沿用 `../CMake-Cookbook/latex_source`
的三层结构（`main.tex` → `book/ccs.tex` → `book/index.tex` → `book/content/**`）。

当前状态：**719 页，三遍 xelatex 零错误、零缺字**
（`Overfull \hbox` 94 条，其中只有 2 处超过 15bp，见第 9 节；
`Overfull \vbox` 1 处 9.8bp —— 第 20 章一个脚注页的脚注底比版心低约 3.5mm，
仍高于页脚 17bp，不裁字）。

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

## 3. 版式对照（原书实测 → 本工程实现）

| 项目 | 英文原版实测 | 本工程实现位置 |
|---|---|---|
| 纸张 | 504 × 661.5 bp（7 × 9.19 in） | `geometry`：`paperwidth/paperheight` |
| 版心 | 360 bp 宽（左右边距各 72 bp），版心高 528 bp | `textwidth=360bp`、`top=53bp`、`bottom=78.5bp` |
| 正文 | Minion Pro 10.5 pt / 行距 13.2 pt | 字号 10.5 bp + `\linespread{1.0476}` |
| 章号 | MyriadPro-SemiboldCond 16.8 bp | `\@makeschapterhead` 内 `16.8bp` |
| 章名 | 25.2 bp，右对齐，上方 0.5 bp 通栏细线 | 同上 |
| 节 / 小节 | 18.9 bp / 15.8 bp，编号在行内 | `\titleformat{\section}` / `{subsection}` |
| 代码 | UbuntuMono 8.5 bp，无框无底色无行号，左缩进 17 bp | `\codefont` + `\fvset{xleftmargin=17bp}` |
| 代码配色 | 关键字 #006699 粗、类型 #007788 粗、字符串 #cc3300、数字 #ff6600、注释 #336666 斜 | `book/oreilly-pygments.tex` |
| 脚注 | 8 bp / 10 bp，正文内上标 7 bp | `\footnotesize` 重定义 |
| 图 | 通栏 0.4 bp 灰框，题注 9.8 bp 粗体左对齐 | `\myFigure` |
| 表 | 无竖线，表头黑底白字，9 bp | `\myTableBegin`/`\myTableEnd` |
| NOTE 框 | 图标在版心外 17 bp、宽 38 bp，正文缩进 65 bp、9.6 bp | `myNote` 环境 |
| 页脚 | 「页码 \| 章名」/「节名 \| 页码」+ 0.3 bp 细线，9 bp | `fancyhdr` |
| 链接色 | #990000 | `linkcolor=linkred`（目录内改黑） |

---

## 4. 模版文件各自管什么

- **`main.tex`** — 只负责 `\input{book/ccs.tex}` 和 `\subfile{book/index.tex}`。
  要换纸张、换字体、改标题样式，都不在这里动。
- **`book/ccs.tex`** — 全书唯一需要手改的版式文件，503 行，按注释分块：
  页面尺寸 / 字体 / 段落 / 代码高亮开关 / 标题 / 插图 / 表格 / 提示框 /
  引文 / 目录书签 / 长串断行 / 页脚 / Unicode 兜底 / 章节命令。
- **`book/index.tex`** — 由 `convert.py` 生成，勿手改。它只调用四种命令装配全书。
- **`book/oreilly-pygments.tex`** — 70 个 `\PYG{...}` 定义，minted 与预着色两条路线共用。
- **`tools/convert.py`** — 570 行，转换规则都在这儿，改转换行为只改它。

## 5. 自定义命令速查

| 命令 / 环境 | 用途 |
|---|---|
| `\myChapter{n}{标题}{文件}` | 编正文章（输出「CHAPTER n」+ 细线 + 标题 + 指定后空白） |
| `\myFront{标题}{文件}` | 前言类：进目录、无编号、小节也不编号 |
| `\myFrontNoToc{文件}` | 封面/版权页：不进目录、不印页眉 |
| `\chapmark{...}` | 设定页脚里的章名 |
| `\myFigure{路径}{图 x-y：题注}` | 通栏插图 + 细边框 + 粗体题注 |
| `\myCaption{...}` | 单独的图/表题注段 |
| `\myTableBegin` … `tabularx` … `\myTableEnd` | 表格外层（**不能**包成环境，见第 8 节） |
| `myNote` / `mySidenote` / `myEpigraph{出处}` | 提示框 / 旁注 / 章首引文 |
| `myCopyright` | 版权页小号排版 |
| `\texthref{url}` | 显示文字就是网址的链接 |

想改「CHAPTER 3」为「第 3 章」，只改 `ccs.tex` 里 `\@makeschapterhead` 的那一行
（`CHAPTER~\mychaptergiant`）。目录与页脚的中文章名已由 `\myChapter` 生成，不会跟着变。

---

## 6. 转换流水线

```
EPUB ──extract.py──▶ work/src/*.html + work/structure.json + images/chNN/*.jpeg
                          │
                          └──convert.py──▶ book/content/**/*.tex + book/index.tex
                                                  │
                                            xelatex ×3 ──▶ main.pdf
```

`convert.py` 的主要规则：

- **脚注**：EPUB 把注释集中在每章末尾的 `zhu_shi_nei_rong` 块里。先整体收集，
  再按正文引用顺序发 `\footnote{}`，保证编号顺序正确。
- **代码**：`<pre>` 的 class 映射到 rust / shell / toml / c / text 五种 lexer，
  构建期由 Pygments 预着色成 `\PYG{...}`，再包进 `fvextra` 的 `Verbatim`。
- **图**：只认「以『图 x-y』开头的紧邻段落」为图注。EPUB 里「如表 17-2 所示」
  这种句子会被误判成图注并生成不存在的图片路径，这条规则就是挡它的。
- **表**：`rowspan`/`colspan` 建「placed / row_gap / col_skip」三套网格
  （详见第 8 节 multirow 一条）。列宽按内容长度加权。
- **引文块**：含 `calibre8` 类的 → `myEpigraph`（带出处）；其余 → `myNote` / `mySidenote`。
- **长路径**：正文里形如 `a/b/c` 的长 token 在 `/` 后补 `\allowbreak{}`。

---

## 7. 字体

```
中文正文   Source Han Serif SC（思源宋体）
中文标题 / 表头 / 代码里的中文   Microsoft YaHei
西文正文   Cambria
西文标题   Arial Narrow
代码       Consolas（C:/Windows/Fonts/consola*.ttf）
希伯来文   Arial          （第 17 章 "ערב טוב" 示例）
卡纳达文   Nirmala UI     （第 3 章 "ಠ_ಠ" 示例）
```

原书用的是 Minion Pro / MyriadPro-SemiboldCond / UbuntuMono，三者都是商用字体，
本机以 Cambria / Arial Narrow / Consolas 替代：Cambria 数字等高、x 高度接近
Minion，替换后行宽变化最小；Arial Narrow 是最接近 MyriadPro-SemiboldCond 的
字宽密度；Consolas 自带全套框线字符（目录树 `─├└│` 直接可显示）。

**换平台要改的只有 `ccs.tex` 第 69–78 行这几条 `\set*font`。** macOS / Linux 上把
Cambria 换成 `TeX Gyre Pagella`、Arial Narrow 换成 `TeX Gyre Heros Condensed`、
Consolas 换成 `JetBrains Mono` 或 `Iosevka`，中文换成 `Noto Serif CJK SC` 即可。

---

## 8. 兼容性与格式问题（都是实际踩过的）

**1. minted v3 在 Windows 下缓存目录必须是绝对路径。**
`latexminted` 只允许把高亮结果写进 `TEXMFOUTPUT` / `TEXMF_OUTPUT_DIRECTORY`
指定的绝对路径内，而 minted 默认 `\minted@cachedir = "_minted"` 是相对路径，
于是 1400 多个代码块全部写入失败，报错还极具误导性（"Cannot locate file ..."）。
`ccs.tex` 里保留了补丁：用 `kpsewhich --var-value TEXMF_OUTPUT_DIRECTORY` 读出
绝对路径再改 `\minted@cachedir`。`build.bat` 已设好这个变量。

**2. 默认走「构建期预着色」，不走 minted。**
两条路线观感一致，但 minted 在这台机器上单遍编译要 **19 分钟**，预着色只要
**3 分钟**（全书 1400+ 代码块）。切换开关是 `ccs.tex` 里的
`\newif\ifmintedcode`；走 minted 需要 `-shell-escape` 且 Pygments 在 PATH 里。

**3. tabularx 不能被自定义环境包住。**
`tabularx` 要向前扫描自己的 `\end{tabularx}` 才能算列宽，包一层就会读到
`\end{myTable}`，报 `File ended while scanning use of \TX@get@body`。
所以表格用的是开关宏 `\myTableBegin` / `\myTableEnd`，`tabularx` 本体留在正文里。

**4. titlesec 的 `[display]` 形状不允许 label 里出现 `\par`。**
章首页要「CHAPTER N / 细线 / 章名」三行结构，用 `\titleformat{\chapter}[display]`
会报 `Paragraph ended before \ttl@format@ii`。改为直接重定义
`\@makeschapterhead`（全书只用 `\chapter*`，编号由 `\myChapter` 手工设定）。

**5. multirow 的三种宽度语义，用错会串行。**
- `\multirow{n}{*}{文本}`：自然宽度，短文本用它可以；
- `\multirow{n}{=}{文本}`：按列宽推断，在 `tabularx`+`X` 组合里算出来偏大，
  会把内容压到相邻行上（审读致谢表实测重叠）；
- `\multirow{n}{140bp}{文本}`：显式宽度，最稳，`convert.py` 现在用这个。

另外，被 rowspan 跨过的单元格**必须补一个空槽 `&`**，否则整行左移一位
（同一个表最初的错误就是「第 5 章 引用」跑到第一列去了）。被 colspan 跨过的
则不能再占列，由 `\multicolumn` 一并输出——`convert.py` 用 `row_gap` /
`col_skip` 两个集合分开记。

**6. 缺字要分两处兜。**
正文缺字用 `newunicodechar` 映射成数学符号（→ ≤ ≠ 等）或换字体；但
**verbatim/Verbatim 里的字符不展开字符含义**，`newunicodechar` 管不到。
代码块里的希伯来文与卡纳达文是 `convert.py` 生成 `\PYG` 序列时用
`{\hebrewfont …}` 包起来的——`Verbatim` 的 `commandchars=\\\{\}` 允许花括号和
反斜杠，所以这套写法在代码块内同样生效。

**7. 中文两端对齐 + 超长路径 = 无解的 Overfull。**
CJK 字间可断但 latin 长 token 不可断，`/` 也不是 TeX 默认断点。
`convert.py` 给形似路径的 token 在 `/` 后补 `\allowbreak{}`；仍有两处约 37 bp
的溢出（第 23 章两条 `libgit2.so` 路径），因为断完剩下的片段还是太长。
硬要消掉可以把 `\emergencystretch` 从 `1em` 调到 `3em`，代价是那两段字距明显发松。

**8. openright 自动插入的空白页会残留页脚。**
`book.cls` 的 `\cleardoublepage` 不给空白页设 `\pagestyle{empty}`，fancy 页脚会
在空白页留下「12 |」这种只有页码和竖线的残缺行。`ccs.tex` 重定义了
`\cleardoublepage` 为 `\clearpage{\pagestyle{empty}...}` 修掉。

**9. `\setcounter` 是全局赋值。**
`\myFront` 里把 `secnumdepth` 设成 -1 让前言的小节不编号（否则目录里出现
「0.1 致谢」「0.2 读者对象」），`\subfile` 返回后必须显式恢复成 2，
否则后面 23 章的小节会全部不编号。

**10. subfiles 的路径相对于 `book/`，不是工程根目录。**
`index.tex` 在 `book/` 下，它里面的 `\myChapter{...}{content/main/chapter01}`
解析成 `book/content/main/chapter01.tex`。自己写测试文件放根目录时，
参数要写成 `book/content/...`。

**11. EPUB 结构不可信，转换要有白名单。**
- 图注：只认「图 x-y」开头；
- 表格：EPUB 里 `colspan`/`rowspan` 常有多余声明；
- `<li>` 内的 `<pre>` 必须交给块级转换，否则整段代码被当行内文本塞进 `\texttt`；
- 脚注集中在章末，顺序要按正文引用重排。

**12. 中文版标题层级与原书不同。**
原书只有「章 / 节」，中文版多一级「小节」（如 2.6.1）。`secnumdepth=2`、
`tocdepth=2` 是照中文版设的；若要严格对齐英文原版，把两者改成 1。

**13. 图与题注必须打进同一个 `\vbox`。**
`center` 环境 + `\nopagebreak` 挡不住分页：`\end{center}` 后面的 `\vspace`
本身就是合法断点，图排到页底时题注会单独掉到下一页（第 3 章图 3-3 就是这么
掉的）。`\myFigure` 现在是一个垂直盒子，图与题注无从拆开。

**14. 页脚里的竖线要跟着书眉内容走。**
「页码 | 章名」的 `|` 若写死在 `\fancyfoot` 里，书眉为空的页（封面、扉页、
版权页）会留下孤零零的 `| 3`。改成把分隔符写进 `\markboth` 的内容
（`\chapmark` 与 `\my@sectionmark` 各带一半），空书眉自然什么都不印。
另注意：不能用 `etoolbox` 的 `\ifdefempty{\@leftmark}` 判空——LaTeX 在标记里
存了大小写包装，永远判不出"空"。

**15. 目录不要跟着链接染红。**
`\hypersetup{linkcolor=#990000}` 是给正文交叉引用用的，但目录条目也是
hyperref 链接，整页目录会变暗红，与原书（纯黑目录）不符。`ccs.tex` 里把
`\tableofcontents` 包了一层，只在目录内把 `linkcolor` 改回黑色，顺便把目录
标题后的空白从 96bp 收到 44bp（原书目录紧跟标题）。

**16. 比一页高的表必须用 xltabular，否则内容会被静默吃掉。**
`tabularx` 是一个不可分割的段落，表高超过版心时不会报错，而是直接从页脚
下方越过去——超出的行**在 PDF 里就没有了**（表 3-1「类型一览」实测越界
527bp，三条腿全丢）。`convert.py` 现在按行数分流：`>= 14` 行用 `xltabular`
（longtable + X 列，能跨页并 `\endfirsthead`/`\endhead` 重复表头），
短表仍用 `tabularx`（排版更紧凑）。`\myTableBegin` 里把 `\LTpre/\LTpost`
归零、`\LTleft/\LTright` 归零，保证跨页表与正文左缘对齐。
判断依据是 `Overfull \vbox` 而非 `\hbox`——后者只提示横向溢出，
前者才是"这一页装不下"的信号。

**17. 列表项里每个子元素后面的 `tail` 都不属于子元素。**
lxml 把一段文本切成 `text` / `tail` 两半：`<li>形如 <code>Ok(v)</code> 的值</li>`
里，`"形如 "` 是 `li.text`，`"Ok(v)"` 是 `code.text`，而 `" 的值"` 是 **`code.tail`**。
只遍历子元素而忘了 `ch.tail`，标记之后的文字会**整段消失**，且不留任何编译告警
（本书初版因此丢了 20 章、233 个列表项、9 009 字）。
第二个坑：`inline()` 是对「子节点」分派的，`inline(code)` 只吐裸文本、不套
`\texttt`，所以列表里要另走 `wrap(el)` 按元素自身标签包装。
自检办法：搜 `\item` 后紧跟空行的位置，以及列表项里是否有孤立的单词行。

**18. 章名/小节标题上的脚注要单独搬运。**
`\chapter*` 的标题由 `\myChapter` 输出，`block()` 遇到 `h1` 直接返回，标题上的
脚注标记就没人消费，随队列一起丢掉（第 4 章章名那条译者注）。做法是把它拆成两半：
`\myChapter` 的标题参数里放 `\mychapterfnmark`（标记），章文件开头放
`\footnotetext[1]{…}`（正文）。两个坑：
- 标题里的 `\footnotemark` 会被 hyperref 包成链接、染成暗红，要局部
  `\hypersetup{linkcolor=ink}`；
- `\@makefnmark` 是 7bp（按正文 10.5bp 定的），放进 25.2bp 的章名里几乎看不见，
  所以 `\mychapterfnmark` 里局部重定义成 11bp。
小节标题（`h2`/`h3`）上的脚注则走 `pending_fn`，顺延到下一段开头。

**19. 脚注必须按章重新编号。**
`book` 类默认全书连续编号，第 4 章第一条脚注会显示成 `[14]`（前三章累计了 13 条），
与原书「每章从 1 开始」不符。在 `\myChapter` / `\myFront` 里各加一句
`\setcounter{footnote}{0}` 即可。

---

## 9. 已知差异

- **字体**：Minion Pro / MyriadPro-SemiboldCond / UbuntuMono 为商用字体，已用
  近似字体替代（见第 7 节），字宽与字重有细微差别。
- **章首标签**：写作 `CHAPTER 3`（照英文原版）。中文版纸书排的是「第 3 章」，
  改法见第 5 节末。
- **两处 37 bp 溢出**：第 23 章 `libgit2.so` 路径所在段落，原因与调法见 8.7。
- **未做书末索引**：中文版有索引，EPUB 里没有，本轮未生成。
- **页眉**：原书无页眉、只有页脚，本工程同样不设页眉。
