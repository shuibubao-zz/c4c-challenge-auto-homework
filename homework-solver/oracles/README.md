# Oracle 来源文档与逐源蒸馏台账

本目录存放被蒸馏进 T-box 的 oracle 知识源。
T-box 运行时文件为 `domain_skills/*.yaml`，每个域一份。

## 蒸馏机制

```
Oracle 文档 → 通读并抽取 → 蒸馏为 T-box 条目：
  - 概念节点（定义、定理、前置关系）
  - 解法方法（计算型 solver 的领域依据）
  - 概念模板（非计算题的证明/举例骨架）
  - 分类规则（题面特征 → 概念的映射）
  - 回验规则（答案怎么算"对"）
  - 常见错误（学生最常踩的坑 → 反向用于出回验）
```

## 逐源蒸馏台账（Per-Source Distillation Ledger）

下表逐源登记：每个 oracle 文件被蒸馏进了哪个 T-box 的哪些章节。
「落点」列的行号区间以当前仓库文件为准。

### calculus_limits.yaml（C4B 原有主域）

| # | Oracle 文件 | 类型 / 权威级 | 蒸馏落点（T-box 具体条目） |
|---|------------|--------------|--------------------------|
| 1 | `01_textbook_stewart.md` | 教科书 / Primary | `concepts` 概念图：绝对值、不等式、极限定义（ε-δ 形式）、单侧极限、极限四则定律、连续性三条件、切线定义（`calculus_limits.yaml` L72 起） |
| 2 | `02_study_guide.md` | 学习指南 / Primary | `oracle_sources.distilled_knowledge`（L20–29）：四关键词法、δ 不依赖 x、连续三条件、L'Hôpital 适用前提、Newton 法、隐函数求导；常见错误清单进入分类规则的反例关键词 |
| 3 | `03_instructor_guide.md` | 讲师指南 / Primary | `oracle_sources.distilled_knowledge`（L33–44）：GCSC 证明骨架、非线性 ε-δ 先界变量因子再取 min、五类极限情景、代数工具箱、增长阶、不动点证明套路、三大考试陷阱 |
| 4 | `04_pedagogy_guide.md` | 教学法指南 / Primary | `oracle_sources.distilled_knowledge`（L48–58）：范式转变与博弈比喻（概念题模板的话术来源）、预分析 vs 正式证明之别、去心邻域细节、min 记号、共轭有理化、错误分类学、评分权重 40/60 |
| 5 | `05_exemplars.md` | 例题集 / Secondary | `classification_rules`（L559 起）的调参测试集 + `validation_rules`（L686 起）的对照答案：Worksheet 3（8 题 8/8）、Worksheet 4（10 题 10/10），全部作为 golden test 固化在 `tests/selftest.py` |

### linear_algebra.yaml（C4C 扩展 · 主域）

| # | Oracle 来源 | 类型 / 权威级 | 蒸馏落点 |
|---|------------|--------------|--------------------------|
| 1 | 同济大学《线性代数》第七版（工科） | 教科书 / Primary | `concepts` 全部概念节点（行列式、逆、秩、特征值、对角化、正交化、二次型、SVD）+ 每个概念的关键性质（如 Σλᵢ=tr(A)、实对称必可正交对角化）+ 非计算概念模板的理论表述（文档版，非本目录文件） |
| 2 | Strang, *Introduction to Linear Algebra*, 5th ed. | 教科书 / Primary | 秩的"像空间维数"几何视角、SVD 的几何解释（`solvers_linalg.py` 概念模板的话术来源） |
| 3 | SymPy 1.14 Matrix API 文档 | API 参考 / Primary | `solution_methods` 的例程选择（`det()`/`berkowitz_det()` 双实现互检、`eigenvects()` 返回结构、`diagonalize()` 抛错语义、`GramSchmidt(orthonormal=True)`）+ 回验规则的双通道检查设计 |

### differential_equations.yaml（C4C 扩展 · 第二域）

| # | Oracle 来源 | 类型 / 权威级 | 蒸馏落点 |
|---|------------|--------------|--------------------------|
| 1 | 同济大学《高等数学》第七版 第十二章 | 教科书 / Primary | `concepts`（通解/初值问题/拉普拉斯变换/方程组）+ 解法通式（一阶线性积分因子、二阶常系数特征方程三种根型） |
| 2 | Boyce & DiPrima, *Elementary Differential Equations* | 教科书 / Primary | 方程组特征值解法的理论依据（`solvers_ode.py::linear_ode_system_solver` 跨域模板） |
| 3 | SymPy dsolve / laplace_transform 文档 | API 参考 / Primary | `solution_methods` 例程（`dsolve(ics=)` 语义、`laplace_transform(noconds=True)`）+ 初值回验的代回检查 |

> **登记规则**：往后每新增一个 oracle 源，必须在本台账加一行，
> 写明蒸馏到了 T-box 的哪些条目；没进 T-box 的知识不算完成蒸馏。

## 给同学的扩展指引

给求解器扩展新域（例如概率论、离散数学）：

1. **收集 oracle 源**——教材章节、讲义、例题卷
2. **抽取 T-box 条目**——概念、解法、分类关键词、回验规则
3. **写域 YAML**——参照 `calculus_limits.yaml` 的结构
4. **实现确定性 solver**——参数化代码，回验内置
5. **拿例题卷实测**——已知答案的题全过才算域覆盖达标
6. **登记台账**——回到本文件，逐源登记蒸馏落点

T-box 的质量直接决定求解率。oracle 源越多，域覆盖越全。
