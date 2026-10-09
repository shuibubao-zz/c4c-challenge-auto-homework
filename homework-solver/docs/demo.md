# 端到端演示：工作表进 → 答题 PDF 出

本文档是可复现的完整演示：**扫描工作表 → 自动求解 → 生成带回验标记的 PDF 答题卷**。
以下全部控制台输出与产物均为真实运行留档（`evidence/demo/`），
非手写示例。

---

## 演示一：整份线性代数作业（12 题）

**输入**：`卢怡然_C4C_作业原件.md`（12 题：计算题 10 道 + 证明/概念题 2 道）

**命令**（一行，无需任何 API Key）：

```bash
python homework-solver/scripts/pipeline.py 卢怡然_C4C_作业原件.md \
    evidence/demo --no-llm --compile \
    --course "线性代数（工科）" --student "卢怡然" --lang zh
```

**真实控制台输出**（完整版见 `evidence/demo/console_output.txt`）：

```
[Stage 1] 摄取
[Stage 2] 解析
   解析出 12 道题（概念 12 / 计算 0）
[Stage 3] 求解（引擎 v2 · LLM 兜底关闭）
   求解 12/12，通过符号回验 12，LLM 兜底 0，耗时 0.4s
      √ 第 1 题  [sympy]          -3
      √ 第 2 题  [sympy]          \begin{pmatrix} \frac{3}{4} & - \frac{1}{2} & ...
      √ 第 3 题  [sympy]          \begin{pmatrix} 2 & 1 \\ 4 & 3 \end{pmatrix}
      √ 第 4 题  [sympy]          2
      √ 第 5 题  [sympy]          x = 1, y = 2, z = 3
      √ 第 6 题  [sympy]          0(重数1), 3(重数2)
      √ 第 7 题  [sympy]          P = \begin{pmatrix} -1 & -1 & 1 \\ 1 & 0 & 1 ...
      √ 第 8 题  [sympy]          \begin{pmatrix} \frac{\sqrt{2}}{2} & ...
      √ 第 9 题  [sympy]          Q^{\mathrm{T}}AQ = \begin{pmatrix} 2 & 0 & 0 ...
      √ 第 10 题 [sympy]          U = \begin{bmatrix} -0.948683 & -0.316228 ...
      √ 第 11 题 [template+sympy]  \text{设 } \lambda_1 \neq \lambda_2,\ A\xi_i = ...
      √ 第 12 题 [template+sympy]  \mathrm{rank}(A) + \dim N(A) = n: ...
[Stage 4] LaTeX 生成
   evidence\demo\homework.tex (12,793 字节)
[Stage 5] 编译与验证
   本机可用引擎: ['xelatex', 'pdflatex', 'lualatex']
      [√] PDF 已生成且 > 1KB（461,492 字节）
      [√] PDF 文件完整（含 %%EOF）
      [√] 编译日志无致命错误
      [√] 页数 ≥ 1（实测 8 页）
      [√] 文本中无未解析引用（??）
      [√] 中文字符已正确嵌入（抽取到 1149 个汉字）
   PDF: evidence\demo\homework.pdf (461,492 字节)

================================================================
完成：12 题 / 求解 12 / 回验通过 12 / 耗时 9.97s
================================================================
```

**产物清单**（`evidence/demo/`）：

| 文件 | 说明 |
|------|------|
| `homework.pdf` | 最终答题卷，8 页，每题含步骤 + boxed 答案 + 绿色回验标记 |
| `homework.tex` | 渲染源码，可自行改样式重编 |
| `1_ingested.json` / `2_parsed.json` / `3_solutions.json` | 各阶段中间产物，可供追迹 |
| `run_report.json` | 汇总统计 |
| `console_output.txt` | 本演示完整控制台留档 |

亮点：

- **全程零 LLM 调用**（`--no-llm`），纯确定性管线；
- 第 11、12 题（证明题/概念题）由 `template+sympy` 求解器处理——
  模板骨架 + SymPy 实算验证，不是空模板；
- 12/12 全部通过**符号回验**（答案代回原条件重算残差）。

---

## 演示二：中文混合域工作表（线代 + 常微分方程，10 题）

**输入**：`homework-solver/test_cases/test3_mixed_linalg_ode.md`
（5 道线代 + 4 道常微分方程 + 1 道**跨域题**：用特征值分解解 ODE 方程组）

**命令**：

```bash
python homework-solver/scripts/pipeline.py \
    homework-solver/test_cases/test3_mixed_linalg_ode.md \
    evidence/demo/mixed --no-llm --compile \
    --course "数学综合测验" --student "卢怡然" --lang zh
```

**真实结果**（完整版见 `evidence/demo/console_output_mixed.txt`）：

```
完成：10 题 / 求解 10 / 回验通过 10 / 耗时 9.85s
PDF: evidence\demo\mixed\homework.pdf (401,599 字节，6 页)
中文字符已正确嵌入（抽取到 822 个汉字）
```

这道跨域题（工作表第 10 题）最能体现架构价值：
"求方程组 $x' = x - y,\ y' = x + 2y$（特征值为 $1 \pm i$ 型）的通解"
需要**同时调用两个域的知识**——线代的特征值分解 + ODE 的通解结构，
由 `linear_ode_system_solver`（注册在 ODE 域、内部调用线代例程）完成，
求解与回验全绿。

---

## 演示三（可选）：开启 LLM 兜底

去掉 `--no-llm` 后，未命中确定性求解器的题会降级到 LLM 兜底，
答案仍须过符号回验才标记通过。本演示中 12+10 题全部确定性求解，
LLM 兜底调用次数 = 0。

---

## 一图看懂五阶段

```
卢怡然_C4C_作业原件.md          ┐
        │ Stage 1 摄取          │
        ▼                      │
  1_ingested.json              │   每阶段落盘，
        │ Stage 2 解析         │   全程可追迹
        ▼                      │   （#5 题逐阶段
  2_parsed.json                │    中间值见
        │ Stage 3 求解         │    docs/trace_problem5.md）
   分类→T-box查表→solver→回验   │
        ▼                      │
  3_solutions.json             │
        │ Stage 4 渲染         │
        ▼                      │
  homework.tex                 │
        │ Stage 5 编译+验证    │
        ▼                      │
  homework.pdf（8页·12题·全绿）┘
```
