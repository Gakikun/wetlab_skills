---
skill_id: "skill_molecular_dna_amplification_and_clean"
name: "Oligo Pool 高通量 PCR 扩增与磁珠纯化"
version: "1.0.0"
category: "分子生物学/基因获取"
target_platform: "自动化移液工作站平台"
author: "湿实验自动化组"
tags: ["PCR扩增", "磁珠纯化", "抗体库构建", "DNA扩增"]
timeout_seconds: 7200
---

### 1. 意图与适用场景 (Agent Intent & Context)
*   **功能描述**：用于从头设计的合成寡核苷酸池（Oligo Pool）或候选片段的高通量 PCR 扩增，并通过固相顺磁珠（SPRI Beads）完成双向洗脱与纯化，去除引物二聚体及副产物。
*   **适用场景**：抗体文库构建前期片段扩增、二代测序库构建前 DNA 放大。
*   **限制条件**：仅适用于模板浓度 ≥ 0.1 ng/μL 的体系；若模板为完整质粒，应优先调用质粒提取相关 Skill。

### 2. 输入契约 (Input Schema)
| 参数名                    | 类型    | 单位 | 默认值 | 约束范围         | 描述说明                                   |
| :------------------------ | :------ | :--- | :----- | :--------------- | :----------------------------------------- |
| `input_template_plate_id` | String  | -    | -      | 必须在LIMS已登记 | 起始 Oligo Pool / DNA 样本 96 孔板 ID      |
| `pcr_annealing_temp`      | Float   | °C   | 58.0   | 50.0 - 68.0      | PCR 退火温度                               |
| `pcr_cycles`              | Integer | -    | 25     | 15 - 35          | PCR 扩增循环数                             |
| `beads_ratio`             | Float   | -    | 1.0    | 0.6 - 1.8        | 磁珠与反应液悬浮体积比（用于片段大小截留） |
| `elution_volume`          | Float   | µL   | 30.0   | 15.0 - 50.0      | 纯化产物洗脱体积（ddH2O 或 EB 缓冲液）     |

### 3. 平台与物料依赖 (Hardware & Consumables)
*   **自动化平台工具**：
    *   自动化移液工作站平台（配置 96 通道移液头、抓手机械臂）
    *   板载自动化 PCR 仪（带自动热盖）
    *   高通量 96 孔环形磁板模块
    *   板载振荡加热孵育器
*   **预置试剂与耗材**：
    *   96 孔 PCR 反应板（预加 2x 高保真 PCR Master Mix）
    *   SPRI 纯化磁珠储液槽（室温平衡 30 min）
    *   80% 现配乙醇储液槽
    *   无核酸酶水（EB Buffer）储液槽
    *   200 µL 无菌滤芯吸头 2 盒

### 4. 标准执行动作序列 (Execution Steps)
1. **体系分配**：移液工作站从 `input_template_plate_id` 吸取指定模板 DNA，注入 PCR 反应板对应孔位，吹吸 3 次混匀。
2. **热循环扩增**：机械臂将 PCR 板送入板载 PCR 仪，执行扩增程序：
   - 预变性：98°C, 3 min
   - 循环（`pcr_cycles` 次）：98°C 10s $\rightarrow$ `pcr_annealing_temp` 20s $\rightarrow$ 72°C 30s
   - 终延伸：72°C 2 min $\rightarrow$ 4°C 恒温保存
3. **磁珠结合**：抓手移出反应板，向每孔按 `beads_ratio` 加入磁珠悬液，振荡混匀 2 min，室温静置 5 min。
4. **磁分离与漂洗**：
   - 转移至磁板静置 3 min 直至液体澄清，移液头完全吸除上清废液。
   - 加入 150 µL 80% 乙醇漂洗 2 次（每次浸润 30s 后彻底抽干废液）。
5. **洗脱回收**：离开磁板，加入 `elution_volume` 洗脱液，振荡重悬 3 min，再次上磁板静置 3 min，将澄清产物转移至新 96 孔板。

### 5. 输出契约 (Output Schema & Deliverables)
*   **物理产物**：
    *   `output_purified_dna_plate_id`: 纯化后 DNA 96 孔板（处于 4°C 冷座）。
    *   `output_well_volume`: 实测剩余产物体积（目标值约为 `elution_volume` $\pm 2\ \mu\text{L}$）。
*   **数据产出**：
    *   下机浓度文件：集成 NanoDrop / 读板仪自动生成的 `concentration_ug_per_ml` 矩阵数据文件。

### 6. 质控门禁与异常处理 (QC Gates & Fallback)
*   **Pre-Check**：检查移液枪头库存及 80% 乙醇体积，任一项低于安全水位触发 `ERR_INSUFFICIENT_SUPPLIES` 暂停任务。
*   **Post-Check (门禁)**：
    *   产物浓度  ≥ 15 ng/μL 且 $A_{260}/A_{280} \in [1.8, 2.0]$：标记为 `STATUS_PASS`，自动流转至下游（如大肠杆菌转化或 Gibson 组装）。
    *   产物浓度 ＜ 15 ng/μL ：标记为 `STATUS_LOW_YIELD`，触发 Agent 重试分支（增加 3 个 PCR 循环或告警人工复核）。