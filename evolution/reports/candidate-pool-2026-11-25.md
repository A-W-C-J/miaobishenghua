# 朱雀候选池数据优化报告（2026-11-25）

- 运行环境：生产库 wordflower-cn（cn-hangzhou），14 天窗口 11-11 → 11-25
- 判据来源：`.agents/skills/zhuque-optimization` v2.0（PR-C/D/F + 新铁律）
- **本期结论：v2 red 层第六期无触发（Δbest −0.025 / Δdrop +0.016，均未越线）；green 臂与玄幻候选样本未过门槛；全局活跃实验已达 3 个上限，本期不新发起。不写库。**

## 一、配置现状

updated_at 2026-11-23 11:30 UTC（自上期未变动，无人工干预；五候选 instruction md5 全部完好）。

| 候选 | weight | enabled | tiers | genres | 状态 |
|---|---|---|---|---|---|
| v_current | 0.3 | true | [] | 16 词（urban 8 + 玄幻 8） | 基线臂 |
| exp_voice_urban_1031 | 0 | false | green, yellow | urban 8 | 退役存档 |
| exp_voice_urban_soft_1107 | 0.7 | true | red | urban 8 | red 常驻臂（收割胜者） |
| exp_voice_urban_soft_1107g | 0.3 | true | green | urban 8 | green 实验臂 |
| exp_xianxia_texture_1123 | 0.3 | true | [] | 玄幻 8 | 玄幻实验臂 |

## 二、采用信号

texture 信号 N/A（对照臂已退役）。**tiers 采用**：1107g / 1123 两个新臂的 run 均从 11-23 配置后即刻起量（1107g green 28 run、1123 玄幻 49 run），档位/题材分流在新客户端正常工作。

## 三、分层统计（14 天累计窗口，都市+玄幻关键词组）

| 候选 | 层 | runs/books | avg_best | Δbest | avg_drop | Δdrop | worse_after | 门槛 |
|---|---|---|---|---|---|---|---|---|
| **1107** | **red** | **820 / 121** | 0.828 | **−0.025** | 0.067 | **+0.016** | 0.230 vs 0.198 | 双过 ✓ |
| 1107g | green | 28 / 9 | 0.612 | +0.066 | 0.177 | −0.038 | 0.286 | runs 未过（28<50） |
| 1123 | yellow | 26 / 4 | 0.691 | +0.035 | 0.122 | −0.018 | 0.269 | 未过（books 4<8） |
| 1123 | red | 5 / 2 | 0.757 | −0.096 | −0.071 | −0.122 | 0.600 | 未过 |
| 1123 | green/null | 3/2、15/8 | — | — | — | — | — | 未过 |
| v_current | green/yellow/red/null | 652/103、1147/136、998/124、1150/541 | 0.546/0.656/0.853/0.767 | — | 0.215/0.140/0.051/0.082 | — | 参照臂 |

## 四、判定

### 1107 red——第六期，双指标连续为正但轮换线未达

- Δbest −0.025（基线 0.853，六期轨迹：+0.012 → +0.026 → +0.004 → −0.005 → −0.003 → **−0.025**，本期为历史最优）；
- Δdrop +0.016（连续六期为正，本期略收窄）；
- 熔断：不满足（距两线各 0.025 / 0.046）；轮换：净效应需 ≤−0.05 或 ≥+0.03，差 0.025 / 0.014 → **保持观察，权重 0.7 不动**

### 1107g green / 1123 玄幻——样本积累中

- 1107g：28 run / 9 本（差 22 run 过线）；1123：最多一层 26 run / 4 本——均只报告不动。按当前流速两臂各需 1–2 期过线

### 退役 / 收割——本期无

- 1107 净效应 |Δ| 超 0.02（非平局）且为收割胜者常驻臂，不退役；两新臂样本不足，不触发退役判定
- 无候选净效应越轮换线，不开收割 PR

### 新候选发起——触发条件满足但全局封顶

- 候选格子（题材组 avg_best ≥0.70 且无活跃实验）：**历史组（书级中位 0.776）、科幻组（0.765）** 符合；玄幻组已有 1123 覆盖
- 但全局活跃实验已达 3 个（1107 / 1107g / 1123），触顶「全局 ≤3」封顶 → **本期不新发起**。待任一实验结论成立（退役）后释放额度，下期候选目标：历史组

## 五、写库验证

本期无配置变更，未写库。

## 六、专项：声口卡 A/B 首份数据（#2748 随发版生效）

- 已产数据：113 章带 cardVariant——**A 臂 3 本 / 4 章，B 臂 8 本 / 109 章**
- 分桶正确性验证：11 本书全部按 FNV-1a 哈希落桶（11/11 一致），偏斜是书级小样本（11 本里 3:8，A 臂书基本未生成新章），非分桶故障
- 读数门槛（≥3 章/本）：A 臂 0 本达标——继续积累，下期起每期报数

## 七、待办与观察点（下期 11-28）

1. **1107 red 逼近轮换线**：Δbest −0.025（距 −0.05 差 0.025）为六期最优，若下期延续跌势可能触发轮换（weight 0.7→0.9）；需连续两期确认
2. **green 臂 / 玄幻候选过线**：预计下期各有一层过线，进入正式判定
3. **历史组候选**：待全局活跃额度释放（任一实验退役）后发起，证据已就绪（0.776、无活跃实验）
4. **卡 A/B**：积累中，A 臂样本极薄，关注 B 臂重修订书的生成端改善
5. 例行任务已自愈：今早 09:07–09:08 导出 + analyze 全自动完成（流式修复生效），本期分析数字来自自动产物
6. 全池回退：无需

## 机读数据

```json
{
  "reportDate": "2026-11-25",
  "pool": [
    { "id": "v_current", "weight": 0.3, "enabled": true },
    { "id": "exp_voice_urban_1031", "weight": 0, "enabled": false },
    { "id": "exp_voice_urban_soft_1107", "weight": 0.7, "enabled": true },
    { "id": "exp_voice_urban_soft_1107g", "weight": 0.3, "enabled": true },
    { "id": "exp_xianxia_texture_1123", "weight": 0.3, "enabled": true }
  ],
  "telemetry": [
    { "candidate": "exp_voice_urban_1031", "tier": "green", "runs": 45, "avgBest": 0.56, "avgDrop": 0.214 },
    { "candidate": "exp_voice_urban_1031", "tier": "yellow", "runs": 47, "avgBest": 0.59, "avgDrop": 0.136 },
    { "candidate": "exp_voice_urban_1031", "tier": "red", "runs": 5, "avgBest": 0.755, "avgDrop": 0.243 },
    { "candidate": "exp_voice_urban_1031", "tier": null, "runs": 5, "avgBest": 0.848, "avgDrop": 0.067 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "green", "runs": 35, "avgBest": 0.426, "avgDrop": 0.296 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "yellow", "runs": 38, "avgBest": 0.603, "avgDrop": 0.165 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "red", "runs": 820, "avgBest": 0.828, "avgDrop": 0.067 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": null, "runs": 11, "avgBest": 0.727, "avgDrop": 0.106 },
    { "candidate": "exp_voice_urban_soft_1107g", "tier": "green", "runs": 28, "avgBest": 0.612, "avgDrop": 0.177 },
    { "candidate": "exp_xianxia_texture_1123", "tier": "green", "runs": 3, "avgBest": 0.654, "avgDrop": 0.146 },
    { "candidate": "exp_xianxia_texture_1123", "tier": "yellow", "runs": 26, "avgBest": 0.691, "avgDrop": 0.122 },
    { "candidate": "exp_xianxia_texture_1123", "tier": "red", "runs": 5, "avgBest": 0.757, "avgDrop": -0.071 },
    { "candidate": "exp_xianxia_texture_1123", "tier": null, "runs": 15, "avgBest": 0.766, "avgDrop": 0.025 },
    { "candidate": "v_current", "tier": "green", "runs": 652, "avgBest": 0.546, "avgDrop": 0.215 },
    { "candidate": "v_current", "tier": "yellow", "runs": 1147, "avgBest": 0.656, "avgDrop": 0.14 },
    { "candidate": "v_current", "tier": "red", "runs": 998, "avgBest": 0.853, "avgDrop": 0.051 },
    { "candidate": "v_current", "tier": null, "runs": 1150, "avgBest": 0.767, "avgDrop": 0.082 }
  ]
}
```
