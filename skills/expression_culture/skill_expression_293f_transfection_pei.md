---
skill_id: "skill_expression_293f_transfection_pei"
name: "HEK-293F 悬浮细胞高通量 PEI 瞬时转染与表达"
version: "1.0.0"
category: "表达培养/细胞转染"
target_platform: "细胞工作站平台"
author: "湿实验自动化组"
tags: ["293F", "瞬时转染", "PEI转染", "抗体表达", "细胞工作站"]
timeout_seconds: 3600
---

### 1. 意图与适用场景 (Agent Intent & Context)
*   **功能描述**：利用 PEI（聚乙烯亚胺）介导的化学转染法，将构建纯化后的抗体/重组蛋白表达质粒转染至悬浮培养的 HEK-293F 细胞中进行瞬时表达[cite: 1, 2]。
*   **适用场景**：抗体候选分子转染验证、抗体上清表达制备及高通量蛋白小试生产链路[cite: 1, 2]。
*   **限制条件**：仅适用于处于对数生长期的悬浮 293F 细胞[cite: 2]；转染体系所用培养基必须为无双抗（PS-free）的 293F 培养基[cite: 2]。

### 2. 输入契约 (Input Schema)
| 参数名                     | 类型    | 单位                    | 默认值 | 约束范围       | 描述说明                             |
| :------------------------- | :------ | :---------------------- | :----- | :------------- | :----------------------------------- |
| `input_plasmid_plate_id`   | String  | -                       | -      | 必须在LIMS登记 | 待转染抗体质粒板/管 ID[cite: 1]      |
| `culture_volume_ml`        | Float   | mL                      | 30.0   | 2.0 - 400.0    | 单孔/单瓶转染表达培养总体积[cite: 2] |
| `target_cell_density`      | Float   | $10^6\ \text{cells/mL}$ | 2.5    | 2.0 - 3.0      | 转染起始细胞密度[cite: 2]            |
| `plasmid_dosage_ug_per_ml` | Float   | $\mu\text{g/mL}$        | 1.0    | 0.8 - 1.5      | 培养体系单位体积质粒用量[cite: 2]    |
| `pei_to_dna_ratio`         | Float   | -                       | 3.5    | 3.0 - 4.0      | PEI 与质粒的质量比[cite: 2]          |
| `complex_incubation_min`   | Integer | min                     | 20     | 15 - 20        | 转染复合物室温静置孵育时长[cite: 2]  |
| `expression_duration_days` | Integer | day                     | 5      | 5 - 7          | 转染后细胞连续培养表达天数[cite: 2]  |

### 3. 平台与物料依赖 (Hardware & Consumables)
*   **自动化平台工具**：
    *   细胞工作站平台（配置多通道移液臂、微孔板/离心管夹爪）[cite: 1]
    *   培养扩增平台（配置板载/外置 $\text{CO}_2$ 振荡摇床与培养箱）[cite: 1]
    *   自动化细胞计数仪[cite: 5]
*   **预置试剂与耗材**：
    *   处于对数生长期的 HEK-293F 悬浮细胞液[cite: 2]
    *   无抗生素的 293F 基础培养基[cite: 2]
    *   PEI 转染试剂（$1\ \text{mg/mL}$ 溶液）[cite: 2]
    *   无菌深孔板 / 50 mL 离心管 / 细胞摇瓶[cite: 2, 3]
    *   宽口无菌移液吸头（防止剪切力损伤细胞）

### 4. 标准执行动作序列 (Execution Steps)
1. **细胞状态质检与体系调整**：
   - 机械臂抓取 293F 细胞原液，移液模块取样至细胞计数仪测定活率与密度[cite: 2, 5]。
   - 判定细胞活率 $\ge 90\%$ 后，细胞工作站加入新鲜培养基，将细胞密度精准稀释调整为 `target_cell_density`（$2.0 \sim 3.0 \times 10^6\ \text{cells/mL}$）[cite: 2]。
2. **转染复合物配制（双管/双槽法）**：
   - **管 1（质粒稀释液）**：吸取占培养总体积 $5\%$ 的无抗生素 293F 培养基，加入计算总量的质粒 DNA（按 `plasmid_dosage_ug_per_ml` 计算），吹吸 2 次混匀[cite: 2]。
   - **管 2（PEI 稀释液）**：吸取占培养总体积 $5\%$ 的无抗生素 293F 培养基，按 `pei_to_dna_ratio`（3~4 倍质粒质量）加入对应体积的 PEI 试剂，吹吸 2 次混匀[cite: 2]。
   - **混合孵育**：机械臂将管 1 的质粒体系转移注入管 2 的 PEI 体系中，微孔板振荡器以 600 rpm 振荡混匀 10 秒，室温静置 `complex_incubation_min`（15~20 min）形成转染复合物[cite: 2, 5]。
3. **加样转染与培养启动**：
   - 移液臂将孵育完成的转染复合物逐孔/逐瓶均匀滴加至待转染的 293F 细胞悬液中，轻柔吹吸混匀[cite: 2]。
   - 机械臂将转染容器转移至培养扩增平台（$37^\circ\text{C}$、$8\%\ \text{CO}_2$、设定转速的振荡摇床）中启动培养[cite: 1, 2]。
4. **表达维持与状态标记**：
   - 细胞持续培养 `expression_duration_days`（5~7 天），中控调度器标记任务计时器并锁定收获窗口[cite: 2]。

### 5. 输出契约 (Output Schema & Deliverables)
*   **物理产物**：
    *   `output_culture_vessel_id`: 处于转染表达状态的 293F 培养容器/孔板 ID[cite: 1, 2]。
    *   `scheduled_harvest_timestamp`: 目标收获时间戳（启动后第 5~7 天）[cite: 2]。
*   **数据产出**：
    *   转染前细胞密度与活率报告（JSON/CSV）[cite: 2]。
    *   质粒与 PEI 消耗量对应矩阵清单[cite: 2]。

### 6. 质控门禁与异常处理 (QC Gates & Fallback)
*   **Pre-Check（转染前门禁）**：
    *   若测得细胞活率 $<90\%$ 或细胞结团严重，触发 `ERR_CELL_VIABILITY_LOW` 终止转染，告警人工复核[cite: 2]。
    *   若实测质粒纯度 $A_{260}/A_{280} \notin [1.8, 2.0]$，标记质粒纯度预警[cite: 11]。
*   **Post-Check（转染后流转门禁）**：
    *   培养周期届满后，系统状态自动流转为 `READY_FOR_HARVEST`，自动唤起下游上清离心（$4000\ \text{rpm},\ 4^\circ\text{C},\ 30\ \text{min}$ 离心过滤）或 Protein A 磁珠纯化 Skill[cite: 1, 2]。