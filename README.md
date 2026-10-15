# 报警距离–目标速度 指标规则（summary.json 的线性度与区分度）

> 版本：1.2　　日期：2026-10-14
> 适用对象：autotest 跑完后产生的 `summary.json`（含雷达真值列）
> 算例文件：批次 444 实测（见 §8），示例图 `2026-10-14-lidar-speed-distance-rule-example.png`
> 绘图脚本：`docs/plot_lidar_fit.py`（源码见附录 D）

> **v1.2 变更**：① 口径收敛为**两个**——全速域（剔除离群值）与 ≤30km/h（剔除离群值，§3.4），
> 不再单独输出高速段口径；② 一致性检查/对比规则同步改为"双口径并行"（§4.5/§6）；
> ③ 算例更新为双口径；④ 附录 D 增加绘图脚本源码。
>
> **v1.1 变更**（保留）：离群点（离散值）剔除由"可选"改为**必选步骤**（§3.2），并新增异常清单披露要求；
> 新增分段（现收敛为双口径）一致性检查与对比规则。

---

## 1. 目的

把"**报警发生的位置（距离）随目标速度的变化行为**"量化成可比较的数字，回答两个问题：

1. **单次运行**：报警距离和速度的关系有多"线性"（线性度数值），以及报警位置对速度的**区分能力**有多强（区分度数值）；并给出可视化图表。
2. **两次运行**：把两个 `summary.json` 的区分度做**对比**，得出"两次运行的报警行为是否可区分/区分度变化多大"的结论。

本规则**只使用** `summary.json` 中由雷达（lidar）真值链路提供的两列数据（`雷达距离`、`雷达速度(m/s)`），不依赖评测内部中间产物，可在任意两次运行之间复现。

---

## 2. 输入数据与字段

### 2.1 数据来源

`summary.json` 由 `summary_by_table.py` 在每次 autotest 后生成（与 `summary.xlsx`/`summary.txt` 同目录）。其中雷达列来自：

- `lidar_truth.csv` / `lidar_truth.json` 产物（`lidar_truth_mapping.py` 生成于每个 case 的 `*_output/` 目录）；
- 评测写入 `adas_result.json` 的 `first_trigger_frame`（报警帧）。

### 2.2 本规则使用的字段

| 字段 | 含义 | 本规则中的角色 |
|---|---|---|
| `雷达速度(m/s)` | 目标**稳定段速度真值**（独占整段轨迹、按"匀速目标"假设取最平稳段均值） | 自变量 **x** |
| `雷达距离` | **报警帧**目标的真实距离（lidar 真值；外插帧为斜率外推） | 因变量 **y** |
| `雷达真值来源` | `内插` / `外插`（报警帧定位方式；外插=速度不可靠、位置可靠） | 可选过滤口径 |
| `结果` | `正触发` / `过早` / `过晚` / `漏报` / `无触发` 等 | 分组着色 / 可选过滤口径 |
| `base_id`、`topic`、`relpath`、`视频名称` | 用例标识与场景信息 | 排查定位用（**离群点清单必须带这些字段**） |
| `触发距离` | 模型自报的触发距离（`position.y`） | **不用于本规则**；仅作可选对照 |

### 2.3 为什么用"雷达距离"而不是"触发距离"

本规则评估的是**报警时机本身的真实水平**。若用模型自报距离，会把测距误差（例如 BEV 系统性偏差）混进"报警时机"的评估里，导致同一报警行为在不同测距精度下得出不同结论。雷达距离是外部真值，衡量的是"报警发生在目标多远处"这一客观事实。

> 可选对照分析（不在本规则主口径内）：用 `触发距离` 重算一遍，与雷达距离口径对比，用于分离"报警时机差异"与"测距精度差异"。

### 2.4 前置校验

- `summary.json` 必须含 `雷达距离`、`雷达速度(m/s)` 两列，且**至少 5 行有效数据**；否则**中止**并输出明确提示（如：`雷达列缺失或为空：请确认该批次已跑 lidar_truth_mapping 且产物为新格式`）。
- 若全部行都缺其中任一列（典型原因：mapping 步骤未执行、或产物为旧格式），视为"无数据"，不给任何数值结论。

---

## 3. 样本集定义

### 3.1 纳入 / 剔除（第一层：缺失）

- **纳入**：`雷达速度(m/s)` 与 `雷达距离` **均非空**的行。
- **剔除并计数**：缺任一字段的行（多为漏报/无触发、或该 case 无雷达产物）。报告里必须写明 `总数 / 纳入 / 剔除`。
- 同一文件内**重复速度值不做去重**（不同 case 允许同速多点，回归按点计权）。

### 3.2 离群点（离散值）剔除——**必选步骤**

**目的**：让线性度/区分度反映"报警策略的主流行为"，不被个别散乱点（离散值）主导；**剔除不是免责**——异常清单本身是重要产出（见下方披露要求）。

**算法（两轮 MAD 准则，每个口径内部独立执行）**：

```
第 1 轮：对当前口径的样本做最小二乘拟合 y = k·x + c
        残差 r_i = y_i − (k·x_i + c)
        med = median(r),  MAD = median(|r_i − med|)
        稳健尺度 scale = 1.4826 × MAD
第 2 轮：剔除 |r_i − med| > 3 × scale 的点，用剩余样本复拟合 → 最终指标
```

- 固定一轮剔除即可（3σ_MAD 足够保守）；若剔除后 |残差| 仍出现新的 >3×scale 点，报告中注明并保留（避免迭代过拟合）。
- **每个输出口径独立执行**：全速域剔除集合与分段剔除集合可以不同（残差相对各自拟合）。
- **删除后样本量门槛**：若剔除后 n < 5，回退"不剔除"并标注。

**披露要求（硬性）**：

1. 报告中必须列出被剔除点清单（`base_id / 速度 / 距离 / 结果 / 场景`），并给出剔除数量。
2. 满足以下任一条件时，结论中必须**显式点出**（不得静默）：
   - 剔除比例 > 5%；
   - 被剔除点集中在同一 `结果` 或同一 `场景(relpath)`；
   - 剔除使 k 或 R² 变化超过 20%（说明离群点对结论影响显著）。
3. 图形上被剔除点**必须画出**（× 标记），不得从图中消失（见 §5）。

### 3.3 结果 / 来源口径

- **默认（主口径）**：
  - 结果口径：**全部**（`正触发/过早/过晚` 都纳入）。理由：本规则衡量的是"实际报警行为"；只看正触发会人为截断分布。
  - 来源口径：**全部**（含 `外插` 帧）。理由：外插帧仅有距离可靠（已由"位置可靠"机制标注），位置仍参与；速度不受报警帧影响。
- **可选对照口径**：仅 `结果=正触发`；仅 `雷达真值来源=内插`。对照口径必须标注，且**不参与**默认结论。

### 3.4 速度口径——**两个，必报**

本规则固定输出**两个口径**，均先剔除离群值（§3.2，各口径独立执行）：

| 口径 | 定义 |
|---|---|
| 全速域（剔离群） | 全部样本 |
| ≤30km/h（剔离群） | `v ≤ 30 km/h`（= 8.3333 m/s，可配置） |

- 30 km/h 为默认分界（低速段数据密集、行为稳定，是"线性度"的主要参考区间）；可通过参数配置。
- **不再单独输出高速段口径**；低速段以外区间的行为差异，通过"全速域 vs ≤30km/h"的双口径一致性检查（§4.5）与离群点清单（§3.2）体现。
- 两口径的 n 门槛同 §3.5。

### 3.5 样本量门槛

| 门槛 | 值（默认，可配置） | 含义 |
|---|---|---|
| 可计算指标 | n ≥ 5 且 速度方差 > 0（剔除后） | 低于此值只给描述统计并标注"样本不足" |
| 可给出对比结论 | 双方 n ≥ 10 且双方速度跨度均 ≥ 4 m/s | 低于此值对比结论一律标注"样本不足，仅供观察" |

---

## 4. 单文件指标定义

记样本为 `(x_i, y_i)`，x = 雷达速度(m/s)，y = 雷达距离(m)；以下指标均在**离群剔除后**的样本上计算（§3.2），每个口径（全速域 / ≤30km/h）各出一套。

### 4.1 线性拟合（最小二乘）

```
x̄ = Σx_i / n,   ȳ = Σy_i / n
Sxx = Σ(x_i − x̄)²,   Sxy = Σ(x_i − x̄)(y_i − ȳ)
k = Sxy / Sxx          （斜率，单位：秒）
c = ȳ − k·x̄           （截距，单位：米）
```

拟合式：**y = k·x + c**（"报警距离 = k × 速度 + c"）。

### 4.2 线性度指标 L（主指标之一）

```
RSS = Σ(y_i − (k·x_i + c))²
TSS = Σ(y_i − ȳ)²
L = R² = 1 − RSS / TSS
```

- 越接近 1 → 报警距离随速度越呈线性。描述性参考分级：

| R² | 解读 |
|---|---|
| ≥ 0.8 | 强线性：报警距离基本由速度线性决定 |
| 0.5 – 0.8 | 中等：总体趋势线性，存在可解释的离散 |
| 0.2 – 0.5 | 弱：趋势可见但噪声/分场景差异大 |
| < 0.2 | 基本无线性：报警位置与速度无稳定关系 |

### 4.3 区分度指标 S（主指标之二，速度敏感性）

**定义：S = k，单位"秒"。**

物理含义：**目标速度每增加 1 m/s，报警时刻目标真实距离平均增加 k 米**；数值上等于"**等效 TTC**"——若报警严格按 TTC = T 触发，则理想情况下 `k ≈ T`、`c ≈ 0`。

| S = k | 解读 |
|---|---|
| \|k\| < 0.5 s | 基本无区分度：快慢目标都在同一距离报警（平） |
| 0.5 – 2 s | 弱：速度对报警位置有影响但远低于 TTC 线性预期 |
| 2 – 4 s | 中：接近/处于常见 TTC 策略量级（参考 TTC=4s） |
| > 4 s | 强：报警位置对速度的区分明显（视策略定义可超过参考线） |
| k < 0 | **异常**：越快越近（反直觉），必须排查并显著标注 |

不确定度与显著性（单文件报出）：

```
σ² = RSS / (n − 2)
SE(k) = sqrt(σ² / Sxx)
k 的 95% 置信区间 = k ± t(0.975, n−2) · SE(k)
```

### 4.4 辅助输出

| 指标 | 公式/说明 |
|---|---|
| RMSE | `sqrt(RSS/(n−2))`，单位米；越小越好 |
| 截距 c | 米；c 明显为负或过大提示策略非线性/场景混杂 |
| Spearman 秩相关 ρ | 对异常值稳健的单调性核对；与 k 明显矛盾时检查离群点 |
| **Theil–Sen 稳健斜率** | 所有点对斜率的中位数（抗离群）；与最小二乘 k 差异大 → 说明离群影响仍显著，必须与披露清单对照解读 |
| 分速度档统计 | 档宽默认 2 m/s；每档 n、中位距离、均值距离——用于图表与"哪一段不线性"的定位 |

### 4.5 双口径一致性检查（必做）

对两个口径（全速域 / ≤30km/h）的拟合结果做一致性检查：

```
相对差异 = |k_full − k_low| / max(|k_full|, |k_low|)
```

| 情形 | 处理 |
|---|---|
| 两口径 k 同号且相对差异 ≤ 20% | 正常：两口径结论一致，可直接汇总陈述 |
| 两口径 k 符号相反，或相对差异 > 20% | **必须并列陈述两口径**；标明"低速段以外区间（>分界）的报警行为与低速段不一致"，全速域数字不得单独使用 |
| 两口径 R² 差异大（如 ≤30km/h ≥0.8、全速域 <0.3） | 结论以 ≤30km/h 口径为主描述，并注明以外区间的离散原因 |

### 4.6 边界情形处理

| 情形 | 处理 |
|---|---|
| n < 5（任一环节） | 不拟合；仅描述统计 + "样本不足" |
| 速度全同（Sxx=0） | 不拟合；输出该速度下的距离均值/中位/极差 |
| 剔除后 n < 5 | 回退不剔除并标注 |
| k < 0 | 输出并显著标注异常（结合离群清单与双口径差异定位） |
| 剔除比例 > 5% 或集中同场景 | 披露（§3.2），结论必须包含异常清单摘要 |

---

## 5. 图表规范

### 5.1 图内容（单文件）

- **散点**：x = 雷达速度(m/s)，y = 雷达距离(m)；按 `结果` 着色（`正触发` 绿 / `过早` 橙 / `过晚` 红 / 其它灰），图例附各分类样本数。
- **离群点**：以 **× 标记画出**（所有口径剔除点的并集），图例标注剔除数量（§3.2 披露要求）。
- **拟合线**：
  - 全速域（剔除后）一条实线；
  - 低速段（剔除后）一条实线（不同颜色）；两线各自标注 `d = k·v + c（R²=…）`。
- **分界线**：30 km/h 处一条竖直虚线，标注"30 km/h 分界"。
- **参考线**（可选）：虚线 `d = T_ref · v`，`T_ref` 默认取该文件 `需求TTC` 的中位数（缺省 4.0s）。
- **标注框**（左上）：全速域（n/剔除数/R²/k）与低速段（剔除数/R²/k）各一行。
- 坐标轴带单位；标题含批次/文件标识与总样本数。

### 5.2 输出约定

- 文件名：`{summary 文件名主干}_lidar_fit.png`，与 `summary.json` 同目录（或统一输出目录）。
- 对比模式额外出一张双图：A/B 两组散点 + 各自拟合线同图（或分 panel），图例标注 A/B。

### 5.3 示例

批次 444 实测图见 `2026-10-14-lidar-speed-distance-rule-example.png`（按本节规范生成：离群 ×、双拟合线、30 km/h 分界、TTC=4s 参考线）。

---

## 6. 两份 summary.json 的区分度对比

设文件 A、B，各自先按 §4 得到**同口径**下的 `S_A=k_A, S_B=k_B`（及其 `SE`、`n`、`R²`）：全速域一套、≤30km/h 一套——**两个口径都必须对比**。

### 6.1 对比量

| 对比量 | 定义 | 说明 |
|---|---|---|
| **ΔS（主）** | `ΔS = S_B − S_A`（秒） | >0 = B 的区分度更强；相对变化 `ΔS_rel = ΔS / max(|S_A|,|S_B|)` |
| ΔR² | `R²_B − R²_A` | **描述性**，不做显著性检验 |
| Δc | 截距差（米） | 可选，辅助解释 |

对比必须**逐口径**进行：全速域结论 + ≤30km/h 结论分别给出——常见情形是"低速段一致、全速域（低速段以外区间）变化"。

### 6.2 显著性检验（区分度对比）

两文件独立，斜率差的检验统计量（每个口径各自计算）：

```
SE_A = σ_A / sqrt(Sxx_A)        （同 §4.3，均为剔除后口径）
SE_B = σ_B / sqrt(Sxx_B)
z = ΔS / sqrt(SE_A² + SE_B²)
p = 2 · (1 − Φ(|z|))             （双侧，正态近似）
```

- n 较小（单侧 n < 30）时改用 Welch–Satterthwaite 近似自由度的 t 检验，并在报告中注明所用方法。
- p 值含义：**在"两次运行的区分度相同"的假设下，观察到当前或更大差异的概率**。

### 6.3 可比性检查（任何一项不满足 → 不得下"有/无变化"结论）

| 检查项 | 默认门槛 | 不满足时的处理 |
|---|---|---|
| 口径一致 | 结果口径、来源口径、**离群剔除准则（3×MAD）与速度分界（30 km/h）完全一致** | 标注"口径不一致，不可比" |
| 样本量 | 双方（对应口径）n ≥ 10 | 标注"样本不足" |
| 速度范围重叠 | 同口径内 重叠率 = `交集长度 / 并集长度` ≥ 0.5 | 标注"速度范围不可比" |

### 6.4 结论判定表（每个口径各判一次）

| 显著性 | 幅度 | 结论 |
|---|---|---|
| p < 0.05 | \|ΔS\| ≥ max(0.5 s, 10% × max(S_A, S_B)) | **"区分度显著变化"**：B 相对 A 变强/变弱 ΔS = …s（相对 …%），p = … |
| p < 0.05 | 幅度小于阈值 | **"统计显著、幅度小"**：方向明确但实际影响有限，谨慎解读 |
| p ≥ 0.05 | — | **"无显著变化"**：该口径下区分度一致 |
| 可比性不满足 | — | **"不可比"**：仅列数值，不给结论 |

**双口径汇总结论模板**：

```
≤30km/h：A k=3.55s (n=85, R²=0.96)　B k=3.66s (n=83, R²=0.97)　Δk=+0.11s，p=0.31 → 无显著变化
全速域 ：A k=…　B k=…　Δk=…，p=… → …
综合结论：低速段行为一致；全速域差异由低速段以外区间/离群清单揭示（…）
```

（数字为格式示例，真实结论以实际计算为准。）

### 6.5 逐速度档差异表（可选加强）

对两文件在同一档宽（默认 2 m/s）下各档输出：`n_A / 中位_A`、`n_B / 中位_B`、`Δ中位 = 中位_B − 中位_A`。用于回答"差异发生在哪个速度段"，是"显著变化"结论的定位补充。

---

## 7. 输出物约定

### 7.1 单文件指标 JSON（`{summary 主干}_lidar_metrics.json`）

```json
{
  "source": "summary.json 路径",
  "generated_at": "YYYY-MM-DDThh:mm:ss+08:00",
  "population": {
    "total_records": 117, "used": 116, "excluded_missing": 1,
    "result_filter": "全部", "source_filter": "全部",
    "speed_range_ms": [2.73, 13.19], "distance_range_m": [13.47, 45.34]
  },
  "outliers": {
    "criterion": "MAD-3σ (1.4826×MAD)",
    "excluded_counts": { "full": 6, "low": 2 },
    "excluded_list": [
      { "base_id": 708, "speed_ms": 12.88, "distance_m": 13.47,
        "result": "过晚", "relpath": "车辆静止遇左后方行驶二轮车" }
    ]
  },
  "segments": {
    "full": {
      "n": 110, "k_s": 2.2351, "k_ci95_s": [1.915, 2.556], "intercept_m": 14.02,
      "r2": 0.6392, "rmse_m": 4.69, "spearman_rho": 0.835,
      "theil_sen_s": 3.0195
    },
    "low": { "boundary_kmh": 30, "n": 83, "k_s": 3.6590, "k_ci95_s": [3.527, 3.790],
             "r2": 0.9741, "rmse_m": 1.12, "spearman_rho": 0.967, "theil_sen_s": 3.7523 }
  },
  "speed_bins": [
    { "from_ms": 2, "to_ms": 4, "n": 26, "median_m": 17.66, "mean_m": 17.39 }
  ],
  "consistency": { "relative_k_diff": 0.39, "verdict": "两口径分化（>20%），须并列陈述" },
  "conclusion": "≤30km/h 线性度优秀（R²=0.97，k=3.66s≈TTC4s）；全速域（R²=0.64，k=2.24s）与低速段分化，须并列解读"
}
```

### 7.2 对比结果 JSON（`{A主干}_vs_{B主干}_lidar_compare.json`）

```json
{
  "a": { "source": "…", "segments": { "full": {"n": 98, "k_s": 1.62, "r2": 0.44}, "low": {"n": 80, "k_s": 3.55, "r2": 0.96} } },
  "b": { "source": "…", "segments": { "full": {"n": 116, "k_s": 1.27, "r2": 0.23}, "low": {"n": 83, "k_s": 3.66, "r2": 0.97} } },
  "comparability": { "scope_consistent": true, "n_ok": true, "speed_overlap_ratio": 0.87 },
  "per_segment": [
    { "segment": "low",  "delta_k_s": 0.11, "delta_k_relative_pct": 3.1,  "p": 0.31, "conclusion": "无显著变化" },
    { "segment": "full", "delta_k_s": -0.35, "delta_k_relative_pct": -21.5, "p": 0.016, "conclusion": "显著下降" }
  ],
  "conclusion": "≤30km/h 无显著变化；全速域差异主要由低速段以外区间变化驱动"
}
```

> （字段名与结构为准；以上示例数字为示意，真实结论以实际计算为准。）

### 7.3 数值精度约定

R²、ρ 保留 3 位小数；k、CI、c 保留 2–3 位小数；距离类保留 2 位小数。

---

## 8. 算例：批次 444（真实数据，双口径）

**数据概况**（直接读取 444 的 `summary.json`）：

| 项 | 值 |
|---|---|
| 记录数 / 纳入 | 117 / **116**（1 行缺雷达字段，剔除） |
| 速度范围 | 2.73 – 13.19 m/s（9.8 – 47.5 km/h） |
| 距离范围 | 13.47 – 45.34 m |
| 结果分布 | 过早 61 / 正触发 45 / 过晚 10 |
| 来源分布 | 内插 63 / 外插 53 |

**离群点剔除（MAD-3σ）**：

| 口径 | 剔除数 | 明细 |
|---|---|---|
| 全速域 | 6 | base_id 701/703/706/707/708/711 —— **全部为 46–48 km/h 的"过晚"**，全部来自"车辆静止遇左后方行驶二轮车"场景 |
| ≤30km/h | 2 | base_id 684/685（正触发，距离偏高） |

**指标结果（两个口径）**：

| 口径 | n | k（区分度） | c | R²（线性度） | RMSE |
|---|---|---|---|---|---|
| **全速域（剔 6 离群）** | 110 | **2.24 s** | 14.02 m | **0.639** | 4.69 m |
| **≤30km/h（剔 2 离群）** | 83 | **3.66 s** | 7.17 m | **0.974** | 1.12 m |

辅助：Spearman ρ——全速域 0.835、≤30km/h 0.967；Theil–Sen 稳健斜率——全速域 3.02 s（与最小二乘 2.24 s 差异大 → 剔除后离群影响仍不可忽略）；≤30km/h 3.75 s（与最小二乘 3.66 s 接近 → 低速段稳定）。

**解读**：

1. **≤30km/h 口径线性度优秀**：R² = 0.974，k = 3.66 s（≈ 参考线 TTC=4s，图中绿色拟合线与紫色参考线几乎贴合）——低速段报警策略接近"按 TTC≈3.7s 触发"的理想行为。
2. **双口径分化（触发并列陈述规则）**：全速域 k=2.24 s 与 ≤30km/h 的 3.66 s 相对差 39%（>20%，§4.5）——说明低速段以外区间的报警行为与低速段不一致；全速域数字必须与低速段并列解读、不得单独使用。（离群剔除本身也在改善全速域结果：R² 从 0.227 提升到 0.639，但仍低于低速段。）
3. **被剔除的点不是免责项**：全速域剔除的 6 个点全部是 46–48 km/h 的"过晚"、集中在"车辆静止遇左后方行驶二轮车"场景（base_id 701–711 区间，报警时距离仅 13.5–16.1 m，TTC≈1.0–1.3s）——这是本批次最需要排查的问题；≤30km/h 口径另剔除 2 点（684/685，正触发、距离偏高）。
4. 图：`2026-10-14-lidar-speed-distance-rule-example.png`（离群 ×、全速域/≤30km/h 双拟合线、30 km/h 分界、TTC=4s 参考线）。

---

## 9. 边界与异常清单

| # | 情形 | 处理 |
|---|---|---|
| 1 | 雷达列不存在 / 全空 | 中止，提示检查 mapping 产物（新格式） |
| 2 | 部分行缺字段 | 剔除并计数，正常计算 |
| 3 | 剔除后 n < 5 | 回退不剔除并标注 |
| 4 | 速度全同 | 不拟合 + 说明 |
| 5 | k < 0（任一口径） | 输出并显著标注异常（结合离群清单与双口径差异定位） |
| 6 | 剔除比例 > 5% 或集中同场景/同结果 | 必须披露异常清单（§3.2） |
| 7 | 两文件口径不一致（含剔除准则/速度分界） | 不可比，仅列数值 |
| 8 | 速度范围重叠率 < 0.5 | 不可比（或显著标注） |
| 9 | 单侧 n < 10 | 对比结论标注"样本不足" |
| 10 | 最小二乘与 Theil–Sen 斜率差异大 | 说明离群影响显著，与披露清单对照解读 |

---

## 附录 A：公式汇总

```
k  = Sxy / Sxx                       Sxx = Σ(x−x̄)²,  Sxy = Σ(x−x̄)(y−ȳ)
c  = ȳ − k·x̄
R² = 1 − RSS / TSS
RMSE = sqrt(RSS / (n − 2))
SE(k) = sqrt((RSS/(n−2)) / Sxx)
CI95(k) = k ± t(0.975, n−2) · SE(k)
ρ  = Pearson(rank(x), rank(y))
MAD 剔除：med = median(r)，scale = 1.4826 × median(|r − med|)，剔除 |r − med| > 3·scale
Theil–Sen 斜率 = median{ (y_j − y_i) / (x_j − x_i) | x_j > x_i }
z  = ΔS / sqrt(SE_A² + SE_B²)，p = 2(1 − Φ(|z|))
速度重叠率 = |[min_A,max_A] ∩ [min_B,max_B]| / |[min_A,max_A] ∪ [min_B,max_B]|
```

## 附录 B：实现伪代码

```
输入: summary.json 路径（对比模式：两个文件）；参数: 速度分界=30km/h, MAD=3, 档宽=2m/s
1. 读取 JSON；校验含"雷达距离""雷达速度(m/s)"列（否则中止）
2. 第一层筛选（缺字段剔除）；统计 总数/纳入/剔除
3. 对每个口径（全速域 / ≤分界）：
   3a. 若 n<5 或 Sxx==0 → 描述统计 + 标注，跳过
   3b. 两轮 MAD 剔除（§3.2）；记录剔除清单
   3c. 复拟合 → k, c, R², RMSE, SE(k), CI95, ρ, Theil–Sen, 分速度档表
4. 双口径一致性检查（§4.5）；生成结论文本（含披露要求）
5. 输出 {summary主干}_lidar_metrics.json + {summary主干}_lidar_fit.png（离群 × 标记 + 双拟合线 + 分界 + 参考线）
6. 对比模式：对 A、B 分别执行 1–5；
   6a. 可比性检查（口径/剔除准则/速度分界/样本量/速度重叠率）
   6b. 逐口径 ΔS、z、p；按 §6.4 判定表产出结论 + 双口径汇总结论
   6c. 输出 {A主干}_vs_{B主干}_lidar_compare.json（含逐速度档差异表）
```

## 附录 C：术语表

| 术语 | 定义 |
|---|---|
| 线性度 L | 报警距离~速度最小二乘拟合的决定系数 R²（剔除离群后） |
| 区分度 S | 报警距离对速度的敏感性，数值上等于拟合斜率 k（秒） |
| 等效 TTC | k 的物理含义：速度每 +1 m/s 对应报警距离增量；严格 TTC=T 触发时 k≈T |
| 离群点 | 偏离拟合主流趋势超过 3×稳健尺度（1.4826×MAD）的点；剔除但必须披露 |
| 速度分界 | 全速域 / ≤30km/h 两个口径的分界速度，默认 30 km/h（8.3333 m/s） |
| 报警帧 | 首次触发告警的采样帧（评测写入 `adas_result.json` 的 `first_trigger_frame`） |
| 稳定段速度 | 独占整段轨迹、速度最平稳一段的均值（假定目标匀速） |

## 附录 D：绘图脚本源码（`docs/plot_lidar_fit.py`）

依赖 matplotlib（中文字体默认 Microsoft YaHei，非 Windows 环境请替换 `font.sans-serif`）。
用法：`python plot_lidar_fit.py <summary.json 路径> [--boundary-kmh 30] [--out <png>] [--ttc-ref 4.0]`

```python
# -*- coding: utf-8 -*-
"""报警距离 ~ 目标速度 绘图脚本（《lidar 速度-距离指标规则》v1.2 附录 D）。

用法:
    python plot_lidar_fit.py <summary.json 路径> [--boundary-kmh 30] [--out <png 路径>] [--ttc-ref 4.0]

口径（共两个，均为"剔除离群值后"）：
    - 全速域（剔除离群值）
    - ≤ 分界速度（默认 30 km/h，剔除离群值）

离群剔除：MAD-3σ 两轮法（首轮拟合 → |残差 − med| > 3 × 1.4826 × MAD 剔除 → 复拟合），
每个口径独立执行；被剔除点在图上以 × 标出（披露要求）。
"""
import argparse
import json
import os
from statistics import median

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt  # noqa: E402

plt.rcParams["font.sans-serif"] = ["Microsoft YaHei"]
plt.rcParams["axes.unicode_minus"] = False

RESULT_COLORS = {"正触发": "#2ca02c", "过早": "#ff7f0e", "过晚": "#d62728"}
OTHER_COLOR = "#7f7f7f"
MAD_K = 3.0


def load_points(summary_path):
    """从 summary.json 读入 (雷达速度(m/s), 雷达距离) 均非空的样本点。"""
    with open(summary_path, encoding="utf-8") as f:
        data = json.load(f)
    pts = []
    for r in data:
        v = r.get("雷达速度(m/s)")
        d = r.get("雷达距离")
        if v is None or d is None:
            continue
        pts.append({
            "v": float(v),
            "d": float(d),
            "res": r.get("结果"),
            "base_id": r.get("base_id"),
        })
    return pts


def fit(points):
    """最小二乘拟合 d = k·v + c，返回统计信息；样本不足返回 None。"""
    xs = [p["v"] for p in points]
    ys = [p["d"] for p in points]
    n = len(xs)
    if n < 2:
        return None
    xb, yb = sum(xs) / n, sum(ys) / n
    sxx = sum((x - xb) ** 2 for x in xs)
    if sxx == 0:
        return None
    sxy = sum((x - xb) * (y - yb) for x, y in zip(xs, ys))
    k = sxy / sxx
    c = yb - k * xb
    rss = sum((y - (k * x + c)) ** 2 for x, y in zip(xs, ys))
    tss = sum((y - yb) ** 2 for y in ys)
    return {"k": k, "c": c, "r2": 1 - rss / tss, "n": n, "rmse": (rss / (n - 2)) ** 0.5}


def mad_exclude(points, k_mad=MAD_K):
    """MAD-3σ 两轮剔离群；返回 (保留点, 被剔除点)。"""
    f = fit(points)
    if f is None:
        return list(points), []
    resid = [p["d"] - (f["k"] * p["v"] + f["c"]) for p in points]
    med = median(resid)
    scale = 1.4826 * median([abs(r - med) for r in resid])
    if scale <= 0:
        return list(points), []
    keep, drop = [], []
    for p, r in zip(points, resid):
        (keep if abs(r - med) <= k_mad * scale else drop).append(p)
    return keep, drop


def main():
    parser = argparse.ArgumentParser(
        description="报警距离~目标速度绘图（两个口径：全速域/≤分界，均为剔除离群值后）")
    parser.add_argument("summary", help="summary.json 路径")
    parser.add_argument("--boundary-kmh", type=float, default=30.0, help="低速段分界（km/h，默认 30）")
    parser.add_argument("--ttc-ref", type=float, default=4.0, help="TTC 参考线（秒，默认 4.0）")
    parser.add_argument("--out", default=None,
                        help="输出 png 路径；默认与 summary.json 同目录、同名加 _lidar_fit.png")
    args = parser.parse_args()

    boundary = args.boundary_kmh / 3.6
    pts = load_points(args.summary)
    if len(pts) < 5:
        raise SystemExit(f"有效样本不足（{len(pts)} 行）：请检查雷达列是否存在/非空")

    low = [p for p in pts if p["v"] <= boundary]
    keep_full, drop_full = mad_exclude(pts)
    keep_low, drop_low = mad_exclude(low)
    fit_full = fit(keep_full)
    fit_low = fit(keep_low)
    if fit_full is None or fit_low is None:
        raise SystemExit("拟合失败（样本不足或速度无变化）")

    excluded_ids = {p["base_id"] for p in drop_full} | {p["base_id"] for p in drop_low}
    print(f"全速域(剔{len(drop_full)}离群): n={fit_full['n']}, k={fit_full['k']:.4f}s, "
          f"c={fit_full['c']:.2f}m, R²={fit_full['r2']:.4f}, RMSE={fit_full['rmse']:.2f}m")
    print(f"≤{args.boundary_kmh:g}km/h(剔{len(drop_low)}离群): n={fit_low['n']}, k={fit_low['k']:.4f}s, "
          f"c={fit_low['c']:.2f}m, R²={fit_low['r2']:.4f}, RMSE={fit_low['rmse']:.2f}m")
    for scope, drops in (("全速域", drop_full), (f"≤{args.boundary_kmh:g}km/h", drop_low)):
        for p in drops:
            print(f"  剔除[{scope}] base_id={p['base_id']} v={p['v']:.2f}m/s "
                  f"d={p['d']:.2f}m 结果={p['res']}")

    fig, ax = plt.subplots(figsize=(9.5, 6.2), dpi=150)
    for res, color in RESULT_COLORS.items():
        sel = [p for p in pts if p["res"] == res and p["base_id"] not in excluded_ids]
        if sel:
            ax.scatter([p["v"] for p in sel], [p["d"] for p in sel], s=30, c=color,
                       alpha=0.8, edgecolors="white", linewidths=0.5, label=f"{res}（n={len(sel)}）")
    other = [p for p in pts if p["res"] not in RESULT_COLORS and p["base_id"] not in excluded_ids]
    if other:
        ax.scatter([p["v"] for p in other], [p["d"] for p in other], s=30, c=OTHER_COLOR,
                   alpha=0.8, edgecolors="white", linewidths=0.5, label=f"其他（n={len(other)}）")
    ex = [p for p in pts if p["base_id"] in excluded_ids]
    if ex:
        ax.scatter([p["v"] for p in ex], [p["d"] for p in ex], s=46, c="#555555",
                   marker="x", linewidths=1.4, label=f"离群点（剔除 n={len(ex)}）")

    xline = [min(p["v"] for p in pts), max(p["v"] for p in pts)]
    ax.plot(xline, [fit_full["k"] * x + fit_full["c"] for x in xline], color="#1f77b4", lw=2.2,
            label=f"全速域拟合: d={fit_full['k']:.2f}·v+{fit_full['c']:.2f}（R²={fit_full['r2']:.2f}）")
    xlow = [min(p["v"] for p in low), boundary]
    ax.plot(xlow, [fit_low["k"] * x + fit_low["c"] for x in xlow], color="#2ca02c", lw=2.4,
            label=f"≤{args.boundary_kmh:g}km/h 拟合: d={fit_low['k']:.2f}·v+{fit_low['c']:.2f}"
                  f"（R²={fit_low['r2']:.2f}）")
    ax.plot(xline, [args.ttc_ref * x for x in xline], color="#9467bd", lw=1.2, ls="--",
            label=f"参考: d = {args.ttc_ref:g}s × v（TTC={args.ttc_ref:g}s）")
    ax.axvline(boundary, color="#888888", lw=1.0, ls=":")
    ax.text(boundary + 0.15, ax.get_ylim()[1] * 0.95, f"{args.boundary_kmh:g} km/h 分界",
            fontsize=9.5, color="#666666")
    ax.annotate(
        f"全速域（剔{len(drop_full)}离群）: R²={fit_full['r2']:.3f}, k={fit_full['k']:.2f}s\n"
        f"≤{args.boundary_kmh:g}km/h（剔{len(drop_low)}离群）: R²={fit_low['r2']:.3f}, k={fit_low['k']:.2f}s\n"
        f"n={len(pts)}（纳入 {len(pts)} / 全速域剔除 {len(drop_full)}）",
        xy=(0.02, 0.97), xycoords="axes fraction", va="top", fontsize=10.5,
        bbox=dict(boxstyle="round", fc="#f7f7f7", ec="#cccccc"))
    ax.set_xlabel("雷达速度 (m/s)", fontsize=12)
    ax.set_ylabel("雷达距离（报警帧）(m)", fontsize=12)
    ax.set_title(f"报警距离 ~ 目标速度（n={len(pts)}，剔除离群值 + 双口径）", fontsize=13)
    ax.grid(True, ls=":", alpha=0.5)
    ax.legend(fontsize=8.5, loc="lower right")
    fig.tight_layout()

    out = args.out
    if out is None:
        stem, _ = os.path.splitext(os.path.abspath(args.summary))
        out = stem + "_lidar_fit.png"
    fig.savefig(out)
    print(f"图已保存: {out}")


if __name__ == "__main__":
    main()
```

## 修订记录

| 版本 | 日期 | 变更 |
|---|---|---|
| 1.0 | 2026-10-14 | 首版：单文件线性度+区分度、双文件区分度对比、图规范、444 算例 |
| 1.1 | 2026-10-14 | 离群点剔除改必选（MAD-3σ）+披露要求；新增分段口径（≤30km/h 必报）与分段一致性检查；对比规则逐口径化；算例/示例图更新 |
| 1.2 | 2026-10-14 | 口径收敛为两个（全速域/≤30km/h，均剔离群）；一致性检查与对比规则改双口径；附录 D 增加绘图脚本源码 |
