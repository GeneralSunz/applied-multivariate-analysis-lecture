# 实用多元统计分析（第6版）· 电子讲义版 LaTeX 源码

把《实用多元统计分析（第6版）》全书重新排版的 **LaTeX 工程**，输出为适合屏幕阅读的**电子讲义**版式。

> 原书：*Applied Multivariate Statistical Analysis, 6th Edition*，Richard A. Johnson / Dean W. Wichern 著
> 中译：陆璇、叶俊 译，清华大学出版社

---

## 一、这个工程做了什么

原书是**纸书版式**（双面印刷、左右不对称页边距、页码分列外缘），直接在屏幕上读很别扭。
本工程把全书正文重新排成**电子讲义版式**：

| 项 | 纸书版 | 本讲义版 |
|---|---|---|
| 开本 | 185 × 260 mm（非标准） | **A4**（210 × 297 mm） |
| 页边距 | 内侧 18 mm / 外侧 26 mm（**版心偏**） | 左右各 30 mm（**版心居中**） |
| 页眉 | 奇偶页交替，左右不对称 | 统一**居中** |
| 页码 | 奇偶页分列外缘 | 统一**居中页脚** |
| 章首页装饰带 | 左对齐 + 右侧"第 X 章" | 整条**居中** |
| 目录 | 手工排，页码是原书页码 | **自动生成**，页码正确、可点击 |
| 正文行距 | 1.08 | **1.15**（屏幕更易读） |
| 超链接 | 不可见 | 低调**彩色**，全书交叉引用可点击 |

**正文内容与原书一致**，只改版面。

---

## 二、编译方法

需要 **TeX Live 2024**（或更新）与 **XeLaTeX**。**不需要任何脚本**，`book_body.tex` 已是汇编好的正文。

```bash
xelatex -interaction=nonstopmode main_lecture.tex
xelatex -interaction=nonstopmode main_lecture.tex   # 第二遍生成目录与交叉引用
```

产出 `main_lecture.pdf`（约 519 页）。

> 两遍是必须的：第一遍写 `.aux`（目录、交叉引用、书签），第二遍才排定页号和超链接。

### 中文字体

导言区用 `fontset=windows`，调用系统里的**中易宋体 / 黑体**（`SimSun` / `SimHei`）等。
在 Linux / macOS 上编译，把 `main_lecture.tex` 里的

```latex
\documentclass[UTF8,zihao=5,fontset=windows,oneside,openany]{ctexbook}
```

改成 `fontset=fandol`（TeX Live 自带）或 `fontset=macnew` 即可。

---

## 三、目录结构

```
.
├── main_lecture.tex              主文件
├── preamble_lecture.tex          导言区（版式、宏包、自定义命令）
├── book_body.tex                 已汇编的正文（595 页拼接结果）
├── main_lecture.pdf              编译好的成品
├── tex/
│   ├── part_frontmatter_lecture.tex   前置部分：封面/扉页/版权页/译者序/原序/致谢
│   └── pg/                            595 个逐页源文件（pg-013 … pg-607）
└── figs/                         69 张插图
```

`tex/pg/` 是逐页独立的源文件，便于定位与修改；`book_body.tex` 由它们按顺序拼接而成。
只改 `tex/pg/` 里的文件**不会**影响编译结果 —— 需要同步重新拼接，或者直接改 `book_body.tex`。

---

## 四、自定义排版命令

工程在 `preamble_lecture.tex` 里定义了一套宏，用来处理"从扫描件重排"遇到的具体问题：

| 命令 | 用途 |
|---|---|
| `\mvinc{pg-XXX}{l,t,r,b}` | 按归一化坐标从源页裁剪插图（编译前已展开为 `\includegraphics`） |
| `\mvcap{...}` / `\mvtabcap{...}` | 图题 / 表题（小号黑体、居中，且禁止与图表分离） |
| `\mvfigure` / `\mvtable` | 图 / 表容器（用 `\centering` 而非 `center` 环境，避免不可分页造成大片空白） |
| `\mvbox` / `\mvframe` | 加框的命题 / 定义 |
| `\mvthm` / `\mvsub` | 结论 / 小标题 |
| `\mvbold{...}` | 黑体强调 |
| `\mvfit{...}` | 仅在超宽时缩放到版心宽度 |
| `\mvnomalign` | 复位段落对齐（防止 `\centering` 泄漏到全文） |

---

## 五、排版上踩过的坑（供同类项目参考）

1. **`\raggedright` 会让标题后第一段不提行**
   LaTeX 自带的 `\raggedright` 会把 `\parindent` 置 0，而 `titlesec` 的 title format 不是完全分组的，
   于是这个 0 泄漏到标题后第一段，造成"节标题下第一段不缩进、与后面段落不对齐"。
   **改用 `ragged2e` 的 `\RaggedRight`**（保留 `\parindent`），并显式 `afterindent=true`。

2. **`center` 环境导致整块不可分页**
   图表用 `center`（trivlist）会变成不可分页的整体，放不下就整块跳到下一页，留下大片空白。
   改用 `\centering` 即可分页，空白带页数大幅下降。

3. **`\centering` 未分组会泄漏到全书**
   一个没放进分组的 `\centering` 会让后续所有页面居中。用 `\mvnomalign` 兜底复位。

4. **`hyperref` 与 `\boldsymbol` 冲突**
   `\section{$\boldsymbol{\mu}_0$ …}` 会触发 `! Illegal parameter number in definition of \Hy@tempa`。
   用 `\texorpdfstring{$\boldsymbol{\mu}_0$}{mu0}` 解决，并加载 `bookmark` 宏包。

5. **`\w` 在 Python 正则里匹配汉字**
   批量给"式 (X-Y)""图 X.Y"加超链接时，用 `(?<![\w{\\])` 作后顾断言会**永远匹配不上汉字紧邻的引用**
   （如"和式 (2-29)""如图 4.4"）。应改为只排除 ASCII 字母数字。

6. **页末脚注若手工排成普通正文，会被分页推到下一页**
   应使用真正的 `\footnote`，让 LaTeX 的脚注机制把它钉在引用所在页；
   同时 `\renewcommand{\thefootnote}{\textcircled{\arabic{footnote}}}` 可保留原书的 ①②③ 编号外观。

---

## 六、版权声明

**本仓库不主张对原书内容的任何权利。**

- 原书文字、图表、例题、习题的著作权属于 **原作者 Richard A. Johnson、Dean W. Wichern**；
- 中译本的著作权与专有出版权属于 **译者陆璇、叶俊** 与 **清华大学出版社**；
- 本仓库仅为**个人学习用途**的排版工程，供研究 LaTeX 中文科技书排版技术参考，**禁止任何商业使用**；
- 如权利人认为本仓库不当，请提 Issue 或联系仓库所有者，将**立即删除**。

如果你是来找排版技术的，建议重点关注 `preamble_lecture.tex` 与上面第五节列出的坑，
那部分是独立于原书内容的工程经验。
