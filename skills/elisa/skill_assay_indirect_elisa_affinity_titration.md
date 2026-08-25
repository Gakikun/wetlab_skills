---
skill_id: "skill_assay_indirect_elisa_affinity_titration"
name: "抗体结合亲和力测定与梯度滴定自动化间接 ELISA (96孔板)"
version: "1.0.0"
category: "检测分析/亲和力表征"
target_platform: "自动化移液工作站平台 / 检测分析平台"
author: "湿实验自动化组"
tags: ["ELISA", "抗体亲和力", "EC50", "Sirp-alpha", "96孔板", "酶标仪", "洗板机"]
timeout_seconds: 18000
---

### 1. 意图与适用场景 (Agent Intent & Context)
*   **功能描述**：用于高通量测定候选抗体分子（如5514）对靶抗原（如 wt_ RBD 或 BA.1.1_RBD）的体外结合活性。通过自动化梯度稀释、多步洗涤与显色终止，采集 450 nm 吸光度读数并拟合四参数（4PL）剂量反应曲线，计算 EC50 值。
*   **适用场景**：抗体工程突变株亲和力初筛、结合特异性交叉验证、候选分子批次活性一致性评价。
*   **限制条件**：仅适用于吸光度终点法定量分析；如需测定超高精度结合解离动力学常数（$k_{on}$ / $k_{off}$ / $K_D$），应优先调用 SPR 或 BLI 动力学检测 Skill。

### 2. 输入契约 (Input Schema)
| 参数名                    | 类型    | 单位             | 默认值    | 约束范围                  | 描述说明                                                |
| :------------------------ | :------ | :--------------- | :-------- | :------------------------ | :------------------------------------------------------ |
| `input_antibody_plate_id` | String  | -                | -         | 必须在LIMS登记            | 待测一抗样品板 ID（初始浓度 $\ge 9.0\ \mu\text{g/mL}$） |
| `coating_antigen_name`    | String  | -                | "wt_ RBD" | ["wt_ RBD", "BA.1.1_RBD"] | 包被抗原种类名称                                        |
| `coating_antigen_conc`    | Float   | $\mu\text{g/mL}$ | 1.0       | 0.5 - 5.0                 | 抗原包被终浓度                                          |
| `coating_volume_per_well` | Float   | $\mu\text{L}$    | 50.0      | 50.0 - 100.0              | 每孔抗原包被体积                                        |
| `blocking_duration_hours` | Float   | h                | 2.0       | 1.0 - 2.0                 | 37°C 恒温封闭时长                                       |
| `primary_dilution_ratio`  | Float   | -                | 3.0       | 2.0 - 5.0                 | 一抗梯度稀释倍数（默认 1/3 梯度稀释）                   |
| `primary_dilution_points` | Integer | -                | 12        | 8 - 12                    | 稀释浓度梯度点数（含 0 浓度对照孔）                     |
| `secondary_ab_dilution`   | Integer | -                | 2500      | 1000 - 5000               | HRP 标记二抗稀释倍数（1:2500）                          |
| `tmb_incubation_seconds`  | Integer | s                | 180       | 120 - 300                 | TMB 底物显色反应时长（避光）                            |

### 3. 平台与物料依赖 (Hardware & Consumables)
*   **自动化平台工具**：
    *   自动化移液工作站平台（配置多通道移液头与 96 孔吸头阵列）
    *   自动洗板机（如 HydroSpeed 96 高速洗板机）
    *   微孔板恒温孵育箱 / 板载温控振荡模块（37°C）
    *   多功能微孔板检测仪 / 酶标仪（配置 450 nm 吸收光检测光路）
    *   机械臂及孔板传动轨道
*   **预置试剂与耗材**：
    *   96 孔高结合酶标板（如 Jetbiofil #FEP-101-896）
    *   包被缓冲液（1× CBS，Solarbio #C1055）
    *   洗涤液（1× PBST：含 0.5% Tween-20 的 1× PBS 溶液）
    *   封闭液（含 1% BSA 的 1× PBST 溶液）
    *   样品稀释液（含 1% BSA 的 1× PBS 溶液）
    *   检测二抗：HRP 偶联 Anti-Human IgG（Promega #W4031）或 HRP 偶联 Anti-His（BIOEASY #BE2062）
    *   TMB 显色底物液（康为世纪）
    *   终止液（$1\ \text{mol/L}\ \text{H}_2\text{SO}_4$，Solarbio #C1058）
    *   标准对照品：阳性对照抗体（5514，各 2 复孔）、阴性对照抗体（N6，1 复孔）

### 4. 标准执行动作序列 (Execution Steps)
1. **抗原包被（Coating）**：
   - 移液工作站将靶抗原用 1× CBS 稀释至 `coating_antigen_conc`（$1\ \mu\text{g/mL}$）。
   - 向 96 孔酶标板每孔精准分液 $50\ \mu\text{L}$ 抗原稀释液，机械臂将板送入 4°C 冷藏模块孵育过夜（12–16 h）。
2. **洗板与封闭（Washing & Blocking）**：
   - 机械臂将包被板转移至自动洗板机，甩去孔内液体，使用 1× PBST 按 $300\ \mu\text{L/孔}$ 洗涤 3 次并彻底抽干残留。
   - 移液站向每孔加入 $200\ \mu\text{L}$ 封闭液（1% BSA in PBST）。
   - 机械臂转移孔板至 37°C 恒温孵育箱静置封闭 `blocking_duration_hours`（2 小时）。
   - 封闭完成后再次送入洗板机，以 $200\ \mu\text{L/孔}$ 1× PBST 循环洗涤 3 次并抽干。
3. **一抗梯度稀释与加样孵育（Primary Antibody Incubation）**：
   - 移液站在稀释板中用 1% BSA-PBS 配制 $9\ \mu\text{g/mL}$ 的待测抗体与对照品原液（各 $200\ \mu\text{L} \times 2$）。
   - **梯度稀释流水线**：96 孔酶标板首列（第 1 孔）加入 $150\ \mu\text{L}$ 抗体溶液（初始浓度 $9\ \mu\text{g/mL}$ 或 $3\ \mu\text{g/mL}$），第 2 至第 11 孔预先加入 $100\ \mu\text{L}$ PBS 稀释液，第 12 孔（0 浓度对照）仅加入 $100\ \mu\text{L}$ PBS。
   - 移液头从上一浓度孔吸取 $50\ \mu\text{L}$ 加入下一孔，吹吸 3 次混匀后再吸取 $50\ \mu\text{L}$ 传递，形成 1:3 梯度稀释链，确保各孔最终反应体积恒定为 $100\ \mu\text{L}$。
   - 机械臂将板送至 37°C 孵育模块温育 60 分钟。
   - 温育结束后，洗板机以 $200\ \mu\text{L/孔}$ 1× PBST 洗涤 3 次并抽干。
4. **二抗加样与孵育（Secondary Antibody Incubation）**：
   - 移液工作站将 HRP-Anti-Human IgG 按 1:2500 比例在稀释液中现配配制。
   - 向酶标板每孔加入 $50\ \mu\text{L}$ 二抗工作液，送入 37°C 恒温箱孵育 40 分钟。
   - 孵育结束后，洗板机以 $200\ \mu\text{L/孔}$ 1× PBST 洗涤 3 次并彻底拍干/抽干。
5. **显色与反应终止（Color Development & Quenching）**：
   - 移液站向每孔快速分液 $50\ \mu\text{L}$ TMB 显色液，在室温避光环境下精准孵育 `tmb_incubation_seconds`（3 分钟）。
   - 计时截止时，移液站立即向每孔加入 $50\ \mu\text{L}$ $1\ \text{mol/L}\ \text{H}_2\text{SO}_4$ 终止液，终止显色反应。
6. **上机读数与 4PL 数据拟合（Detection & Analysis）**：
   - 机械臂迅速将酶标板转移至多功能微孔板检测仪。
   - 在 450 nm 波长下测定各孔吸光度（$\text{OD}_{450}$ 值）。
   - 数据处理模块自动计算复孔平均值，并以浓度为自变量 $X$、$\text{OD}_{450}$ 为因变量 $Y$ 执行四参数逻辑斯蒂拟合（4PL）：
     $$Y = \frac{A - D}{1 + \left(\frac{X}{C}\right)^B} + D$$
     *(式中 $A$ 为曲线下渐近线估值，$B$ 为 Hill 斜率，$C$ 为 $\text{EC}_{50}$ 值，$D$ 为曲线上渐近线估值)*。

### 5. 输出契约 (Output Schema & Deliverables)
*   **物理产物**：
    *   `output_waste_plate_id`: 测定完成的 96 孔酶标废液板（状态流转为 `DISPOSAL_READY`）。
*   **数据产出**：
    *   `raw_od450_matrix.csv`: 96 孔原始 $\text{OD}_{450}$ 吸光度矩阵数据文件。
    *   `ec50_fit_results.json`: 包含各抗体样品拟合参数（$A$、$B$、$\text{EC}_{50}$、$D$ 及拟合优度 $R^2$）的结构化报告。

### 6. 质控门禁与异常处理 (QC Gates & Fallback)
*   **Pre-Check（前置检查）**：
    *   洗板机 PBST 洗液储量 $< 500\ \text{mL}$ 或废液桶容量 $> 80\%$ 时，触发 `ERR_WASHER_FLUID_ANOMALY` 暂停执行。
    *   一抗起始样品质检浓度 $< 9.0\ \mu\text{g/mL}$ 时，触发 `ERR_ANTIBODY_CONC_INSUFFICIENT` 终止加样流程。
*   **Post-Check（后置质量门禁）**：
    *   **阴性对照基线检查**：阴性对照孔（N6 / 0 浓度空白孔）$\text{OD}_{450} > 0.15$ 时，判定为背景过高（非特异性吸附或洗板不彻底），系统标记 `WARN_HIGH_BACKGROUND` 并提示复核封闭效果。
    *   **阳性对照有效性检查**：阳性对照（5514）最高浓度孔 $\text{OD}_{450} < 1.5$ 时，判定为显色或抗原包被活性失效，抛出 `ERR_POSITIVE_CTRL_FAILED` 终止后续批次决策。
    *   **拟合优度门禁**：若 4PL 拟合模型决定系数 $R^2 < 0.90$，标记该样品为 `STATUS_POOR_FIT`，触发 Agent 备选策略（增加高浓度点或改用非线性样条拟合评估）。