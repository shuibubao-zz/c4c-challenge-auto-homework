# C4C —— 作业自动求解与排版（卢怡然）

> 从 Claude 基线出发，迁移到国产大模型，并从「微积分极限」扩展到「线性代数 + 常微分方程」。
> 目标级别：**Level 3**（并覆盖 L4 的端到端演示与架构贡献）

**English summary:** This repo takes the Claude-based starter kit and migrates it to Chinese LLMs (DeepSeek / Qwen / Kimi) while adding two new subject domains. On a real 12-problem linear algebra homework set it auto-solves **all 12 (including 2 proof/concept problems)** with 43/43 symbolic verification checks passing, solves a 10-problem mixed linear-algebra+ODE worksheet at 10/10 (27/27 checks), and keeps the Claude baseline intact at 17/18 (94.4%).

---

## 目录一览（即 GitHub 提交全集）

```
C4C/
├── README.md                          ← 你在这里
├── homework-solver/                   ← 可运行技能包（完整流水线）
│   ├── docs/
│   │   ├── demo.md                    ← 端到端演示（工作表 → PDF，真实控制台留档）
│   │   └── trace_problem5.md           ← 第 5 题五阶段全程追迹（真实中间值）
│   ├── domain_skills/                ← T-box：calculus_limits / linear_algebra / differential_equations
│   ├── scripts/                       ← pipeline / ingest / parse / classify / solve / render / compile
│   ├── test_cases/                    ← 混合域工作表（线代 + ODE + 跨域题）
│   └── tests/golden/                  ← ★ Claude 基线黄金（pristine starter 实跑固化 + sha256）
├── evidence/
│   ├── run/                           ← 主跑留档：三阶段 JSON / tex / pdf / log / run_report
│   │   └── 关于_llm_calls.jsonl的说明.md   ← 为什么没有真模型调用记录
│   ├── demo/                           ← 演示留档：12 题主卷 + 混合域卷，含控制台输出
│   ├── selftest_run.txt               ← 自检 45/45 完整输出
│   ├── final_check.txt                ← 提交前核对 55/55 完整输出
│   └── skill_manifest.json            ← .skill 包内每个文件的 sha256
│
├── pack_skill.py                      ← 重新打包 .skill（含包内哈希自校验）
├── final_check.py                     ← 提交前核对：必交文档 / 包完整性 / 结果一致性（55 项）
│
├── 卢怡然_C4C_方案设计.md              ← 【必交】架构、模型选型理由、求解策略
├── 卢怡然_C4C_验证报告.md              ← 【必交】12 题答案对比 + Claude 基线对比
├── 卢怡然_C4C_教学说明.md              ← 【必交】安装、使用、支持哪些课程
├── 卢怡然_C4C_拿来说明.md              ← 【必交】拿了什么、Claude 版 vs 国产版差异
├── 卢怡然_C4C_AI日志.md                ← 【必交·无此项无法评审】开发全过程 AI 使用记录
├── 卢怡然_C4C_作业原件.md              ← 【必交】真实作业输入（线性代数，12 题）
├── 卢怡然_C4C_output.pdf              ← 【必交】真实作业输出（8 页中文 PDF）
├── 卢怡然_C4C_homework-solver.skill   ← 【必交】打包的技能包
│
├── c4c-homework-solver-starter.zip    ← Claude 基线原件（sha256 与下发包一致）
│
# 以下三项本地存在但不提交（见 .gitignore）
#   _starter/   基线对照源码，用于重新固化基线黄金
#   _archive/   早期调试草稿，已被 tests/golden/ 取代
```

---

## 结果速览

| 指标 | 结果 |
|---|---|
| 真实作业求解率 | **12 / 12（100%）**，含 2 道证明/概念题 |
| 符号回验（主跑） | 12/12 题通过，共 **43 项断言全绿** |
| 混合域工作表（线代 + ODE + 跨域题） | **10 / 10 求解**，27 项断言全绿 |
| Claude 基线保持 | **17 / 18 = 94.4%**，三个阶段的 JSON 与基线**逐字段一致** |
| 新增学科 | 线性代数（10 计算求解器 + 7 概念模板）+ 常微分方程（4 求解器） |
| Stage 5 编译验证 | **7 / 7 通过**（含缺字检查 V7），8 页中文 PDF，195,197 字节 |
| 自检套件 | **45 / 45** 通过 |
| 提交前核对 | **55 / 55** 通过 |
| 干净克隆复跑 | git clone 后直接跑：流水线退出码 0，自检 45/45，核对 55/55 |
| **未完成** | 无真实国产模型调用（无 API key）；#11/#12 由确定性概念模板求解，非 LLM |

---

## 快速开始

```bash
pip install -r homework-solver/requirements.txt

# 一键跑通（无需 API key 也能出 PDF；--no-llm 关掉 LLM 兜底）
python homework-solver/scripts/pipeline.py \
    卢怡然_C4C_作业原件.md output \
    --no-llm --compile \
    --course "线性代数（工科）" --student "卢怡然" --lang zh

# 混合域（线代 + ODE）演示
python homework-solver/scripts/pipeline.py \
    homework-solver/test_cases/test3_mixed_linalg_ode.md output_mixed \
    --no-llm --compile \
    --course "数学综合测验" --student "卢怡然" --lang zh

# 自检 / 提交前核对
python homework-solver/tests/selftest.py
python final_check.py
```

完整演示（含真实控制台输出与产物清单）见 `homework-solver/docs/demo.md`。

---

## 核心设计：一条铁律

> **凡是 SymPy 能确定性算出来的，绝不让 LLM 猜；LLM 只做推理、证明与讲解，且不参与答案正确性判定。**

由此产生一个反直觉的收益：**换 DeepSeek / Qwen / Kimi 跑同一个包，确定性求解的题答案逐字节不变。** 答案不依赖模型，学科才有可复用性。

```
① 域 T-box + 确定性 SymPy solver   ← 答案的权威来源（43+27 项独立回验）
② legacy starter kit solve.py      ← 保住 Claude 基线（字节未改）
③ 国产 LLM                         ← 推理/证明/讲解，永不当答案裁判
```

非计算题（证明、几何含义、举例）走**概念模板 + SymPy 实例验证**：
模板给骨架，SymPy 真算一个具体实例佐证，回验照跑——不是空文本填空。

---

## 加一门新学科要改几行框架代码？

**零行。** 两步：

1. 往 `homework-solver/domain_skills/` 丢一个 yaml（概念 / 解法 / 分类规则）
2. 在 `homework-solver/scripts/solvers_<学科>.py` 里注册同名 solver

`MultiDomainClassifier` 会自动扫描 `domain_skills/` 下所有 yaml。
混合域工作表第 10 题（用特征值分解解 ODE 方程组）由注册在 ODE 域、
内部调用线代例程的跨域 solver 完成——**调度器零改动**。

---

## ⚠️ 诚实声明：本次未发生真实国产模型调用

- 开发环境中 `DEEPSEEK_API_KEY` / `DASHSCOPE_API_KEY` / `MOONSHOT_API_KEY` 均为空，常见配置目录扫描无命中。
- 第 11、12 题（证明题 / 概念说明题）由**确定性概念模板求解器**完成并通过符号回验，不是 LLM 生成；`--no-llm` 下全程零模型调用，**没有伪造任何答案**。
- 适配层本身经过验证（T4 组用本地 mock OpenAI 服务真发 HTTP，6/6 通过），但那只能证明代码正确，**不能证明 DeepSeek 真的被调过**。这两个区别我没有混为一谈。
- 详见 `evidence/run/关于_llm_calls.jsonl的说明.md` 与 `卢怡然_C4C_验证报告.md` §6。

拿到 key 后补齐这条证据只需一条命令（见上述说明文件）。

---

## 依赖

```
Python >= 3.10    SymPy  NumPy  PyYAML  requests(可选)  pypdf  python-docx
LaTeX：TeX Live / MiKTeX + ctex 宏包（中文 PDF 需要）
```

本包**默认不会**自动安装任何东西。缺依赖时会明确告诉你要装什么；确认要它自动装请加 `--allow-install`。
