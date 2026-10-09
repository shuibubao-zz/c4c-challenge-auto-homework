# 单题五阶段全程追迹：第 5 题（线性方程组）

> 目的：拿一道具体的题，把流水线五个阶段的**真实中间产物**逐一摆出来，
> 证明每个阶段都不是"黑箱"。全部数值取自实际运行（见文末复跑命令），
> 不是手写示例。

**题目原文（作业原件第 5 题）：**

```
解下列线性方程组：

$$
\begin{cases}
x + y + z = 6 \\
2x - y + 3z = 9 \\
x + 2y - z = 2
\end{cases}
$$
```

---

## 阶段 1：摄取（Ingest）

`scripts/ingest.py` 读入 Markdown 源文件，产出 `1_ingested.json`：

| 字段 | 值 |
|------|-----|
| `source_file` | `卢怡然_C4C_作业原件.md` |
| `format` | `markdown` |
| `sections` | 按 `## 第 N 题` 切出的 12 个小节 |
| `raw_text` | 原文全文（UTF-8，无乱码） |

**本阶段 #5 的中间产物**：原文被完整切为一个独立 section，标题行
`## 第 5 题`，正文含题干 `解下列线性方程组：` 与一个 display 公式块。

---

## 阶段 2：解析（Parse）

`scripts/parse_problems.py` 把每个 section 结构化为题目对象，
产出 `2_parsed.json`。**#5 的解析结果（原文摘录）**：

```json
{
  "id": "5",
  "text": "解下列线性方程组：\n\n$$\n\\begin{cases}\nx + y + z = 6 \\\\\n2x - y + 3z = 9 \\\\\nx + 2y - z = 2\n\\end{cases}\n$$",
  "math_expressions": [
    {
      "type": "display",
      "latex": "\\begin{cases}\nx + y + z = 6 \\\\\n2x - y + 3z = 9 \\\\\nx + 2y - z = 2\n\\end{cases}"
    }
  ],
  "type": "conceptual",
  "sub_problems": []
}
```

要点：公式块被独立抽出（`math_expressions`），矩阵环境
`\\begin{cases}...\\end{cases}` 完整保留，方程未被截断。

---

## 阶段 3：分类 → 求解 → 回验（Classify + Solve + Validate）

`scripts/classify.py` 按 `domain_skills/linear_algebra.yaml` 的
`classification_rules` 逐条匹配。**#5 的分类决策**：

| 项 | 值 |
|------|-----|
| 命中规则 | `text_contains: ["方程组", "通解"]`（priority 命中） |
| `domain` | `linear_algebra` |
| `concept_id` | `linear_system` |
| `solver` | `solvers_linalg.linear_system_solver` |
| `classify_reason` | `题干出现『方程组 / 通解』` |

`solvers_linalg.py::linear_system_solver`（SymPy 引擎）求解，
**#5 的求解结果（原文摘录）**：

| 字段 | 值 |
|------|-----|
| `solved` | `true` |
| `steps` | ① 写成矩阵形式 $Ax=b$，用高斯消元求解 → ② 未知量为 $x,y,z$ → ③ 解得 $x=1, y=2, z=3$ |
| `answer_latex` | `x = 1, y = 2, z = 3` |
| `engine` | `sympy` |

**符号回验（3/3 全过，摘自 `validation.checks`）**——把解代回三个原方程，逐个算残差：

| 检查项 | 结果 | 依据 |
|--------|------|------|
| 代回方程 $x+y+z=6$ | ✅ | 残差 = 0 |
| 代回方程 $2x-y+3z=9$ | ✅ | 残差 = 0 |
| 代回方程 $x+2y-z=2$ | ✅ | 残差 = 0 |

手工复核：$1+2+3=6$ ✓；$2(1)-2+3(3)=2-2+9=9$ ✓；$1+2(2)-3=2$ ✓。

---

## 阶段 4：LaTeX 渲染（Render）

`scripts/render_latex.py` 把解答排版进 `homework.tex`。
**#5 的 LaTeX 输出（原文摘录）**：

```latex
\problem{5}

解下列线性方程组：

$$
\begin{cases}
x + y + z = 6 \\
2x - y + 3z = 9 \\
x + 2y - z = 2
\end{cases}
$$

\solution

将方程组写成矩阵形式 $Ax = b$，用高斯消元（或克拉默法则）求解。

未知量为 $x, y, z$。

解得 $x = 1, y = 2, z = 3$。

\textbf{答案：}
\[\boxed{x = 1, y = 2, z = 3}\]

\textcolor{okgreen}{【符号回验 3/3 项全部通过】}
```

注意题干公式是**从解析结果原样回写**的，不是重新生成——保证
"输入什么题目，卷面就呈现什么题目"。

---

## 阶段 5：编译（Compile）

加 `--compile` 时调用本机 xelatex 生成 `homework.pdf`。
#5 在 PDF 中的呈现：题号 5 → 题干 → `\solution` 解答步骤 →
boxed 答案 → 绿色回验标记。中文字体走 ctex 方案，
本页无 "Missing character" 告警（详见 `evidence/run/homework.log`）。

---

## 一次复跑命令

```bash
python homework-solver/scripts/pipeline.py 卢怡然_C4C_作业原件.md \
    <输出目录> --no-llm --course "线性代数（工科）" --student "卢怡然" --lang zh
```

最近一次全量运行结果（`run_report.json`）：
**12 题 / 求解 12 / 回验通过 12 / 耗时 1.52s / LLM 回退 0 次**。

| 阶段 | #5 的落盘文件 | 关键中间值 |
|------|--------------|-----------|
| 1 摄取 | `1_ingested.json` | section 切分正确 |
| 2 解析 | `2_parsed.json` | 公式块完整抽出 |
| 3 求解 | `3_solutions.json` | solver / answer / validation 3/3 |
| 4 渲染 | `homework.tex` | 题干原样回写 + boxed 答案 |
| 5 编译 | `homework.pdf`（`--compile`） | 无缺字告警 |
