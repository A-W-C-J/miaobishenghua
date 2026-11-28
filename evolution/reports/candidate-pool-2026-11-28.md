# 朱雀候选池数据优化报告（2026-11-28）

- 运行环境：生产库 wordflower-cn（cn-hangzhou），14 天窗口 11-14 → 11-28
- 判据来源：`.agents/skills/zhuque-optimization` v2.0（PR-C/D/F + 新铁律）
- **本期结论：green 臂双条件命中熔断（已自动退役）——v2.0 规则下首次自动熔断；玄幻实验发现基线臂缺失 bug（genres 截断），修复 PR #3258 已开；新候选发起暂缓。v2 red 层继续观察。**

## 一、配置现状

updated_at 2026-11-23 11:30 UTC（自上期未变动，无人工干预）。**本期动作**：1107g 熔断退役（enabled=false、weight=0，条目保留记账）。

| 候选 | weight | enabled | tiers | genres | 状态 |
|---|---|---|---|---|---|
| v_current | 0.3 | true | [] | 16 词 | 基线臂（客户端截断 bug 修复中） |
| exp_voice_urban_1031 | 0 | false | green, yellow | urban 8 | 退役存档 |
| exp_voice_urban_soft_1107 | 0.7 | true | red | urban 8 | red 常驻臂 |
| exp_voice_urban_soft_1107g | **0** | **false** | green | urban 8 | **本期熔断退役** |
| exp_xianxia_texture_1123 | 0.3 | true | [] | 玄幻 8 | 玄幻实验臂（判定暂挂起） |

## 二、采用信号

texture 信号 N/A。1107g / 1123 两个新臂 11-23 后即刻起量，档位/题材分流正常（但见第四节 bug）。

## 三、分层统计（14 天累计窗口，urban+玄幻关键词组）

| 候选 | 层 | runs/books | avg_best | Δbest | avg_drop | Δdrop | 门槛 |
|---|---|---|---|---|---|---|---|
| **1107** | **red** | **941 / 131** | 0.825 | **−0.030** | 0.065 | **+0.012** | 双过 ✓ |
| **1107g** | **green** | **72 / 27** | 0.609 | **+0.065** | 0.178 | **−0.040** | 双过 ✓ |
| 1123 | yellow | 68 / 11 | 0.682 | +0.018 | 0.127 | +0.009 | 双过（但无同群基线，见四） |
| 1123 | null | 72 / 34 | 0.772 | +0.007 | 0.064 | +0.016 | 双过（同上） |
| 1123 | red/green | 46/12、16/6 | — | — | — | — | 未过 |
| v_current | green/yellow/red/null | 605/93、1232/136、910/126、1124/523 | 0.544/0.664/0.855/0.765 | — | 0.218/0.136/0.053/0.080 | — | 参照臂 |

## 四、判定与动作

### 1107g green——双条件命中，自动熔断（v2.0 首次）

- Δbest +0.065 ≥ +0.05 且 Δdrop −0.040 ≤ −0.03——**两条熔断线同时越线**（另 pass02 5 vs 67 佐证）
- 动作：enabled=false + weight=0（一次 RPC 完成，已读回验证；urban 8 词被 1107 与 v_current 共用，不摘除）
- 判读：都市声口软叠加在 green 档**有害**——绿档书本就过线，附加约束只添 churn 不添收益；与 red 档有效形成鲜明分层。教训：题材叠加候选默认应限定 red/yellow，green 是「不折腾」层

### 1107 red——第七期，仍未越线

- Δbest −0.030（七期轨迹：+0.012 → +0.026 → +0.004 → −0.005 → −0.003 → −0.025 → **−0.030**）；Δdrop +0.012（连续七期为正但本期收窄）
- 轮换线 −0.05 / +0.03 均未达（差 0.020 / 0.018）→ **保持观察，0.7 不动**

### 1123 玄幻——判定挂起：发现基线臂缺失 bug

- **实证**：11-23 配置后玄幻书（纯玄幻关键词、不含 urban 词）**零 v_current run**，仅 1123 的 137 run / 36 本——实验组 100%、无对照
- **根因**：客户端 `parseCandidate` 与 admin 保存路径把 genres **截断到 8 个**；v_current 现有 16 词（urban 8 + 玄幻 8）被砍成只剩 urban 8，玄幻书匹配不到基线臂。即「v_current 扩 genres 做对照臂」的方案被静默击败
- **修复**：PR #3258（上限 8 → 32 + 回归测试，桌面/admin 门禁全绿；business-specs 既有失败与本次无关，干净 main 同样挂）。随发版生效后 v_current 16 词恢复全量匹配，玄幻实验自动恢复 50/50 对照，无需改配置
- **暂定处置**：1123 暂不退役（无伤害且修复后自愈）、本期不判定、已跑数据（137 run）标记为「无对照期数据」存档

### 新候选发起——触发但暂缓

- 候选格子：历史组（书级中位 0.776）、科幻组（0.765）符合「高残量无实验」；1107g 退役后全局活跃降至 2，额度已释放
- **暂缓原因**：当前客户端 genres 截断 bug 未修复（#3258 待发版），此刻下发任何新题材候选都会重蹈「100% 实验组」覆辙。修复发版后下一期自动发起，目标历史组

### 收割（PR-F）——无

无候选净效应越轮换线（1107 red 差 0.020），不开收割 PR。

## 五、写库验证

- 本期一次 RPC 写入（1107g 熔断），读回验证：weight 0 / enabled false / v_current 指令仍空 / genres 16 词保留 / 遥测透传，全部一致

## 六、待办与观察点（下期 12-01）

1. **#3258 发版**：生效后玄幻实验恢复对照进入判定；新候选（历史组）届时自动发起
2. **1107 red 逼近轮换线**：Δbest −0.030 距 −0.05 差 0.020，若下期延续可能触发 weight 0.7→0.9（需连续两期确认）
3. **green 层结论入库**：题材叠加候选不适用于 green 档——写入候选起草管线的默认纪律（新候选 tiers 默认 red/yellow）
4. 例行数据（11-28 09:10 自动产物）：chapters clean 21453、episodes clean 5860、fewshot 2639 篇
5. 全池回退：无需

## 机读数据

```json
{
  "reportDate": "2026-11-28",
  "pool": [
    { "id": "v_current", "weight": 0.3, "enabled": true },
    { "id": "exp_voice_urban_1031", "weight": 0, "enabled": false },
    { "id": "exp_voice_urban_soft_1107", "weight": 0.7, "enabled": true },
    { "id": "exp_voice_urban_soft_1107g", "weight": 0, "enabled": false },
    { "id": "exp_xianxia_texture_1123", "weight": 0.3, "enabled": true }
  ],
  "telemetry": [
    { "candidate": "exp_voice_urban_1031", "tier": "green", "runs": 45, "avgBest": 0.56, "avgDrop": 0.214 },
    { "candidate": "exp_voice_urban_1031", "tier": "yellow", "runs": 47, "avgBest": 0.59, "avgDrop": 0.136 },
    { "candidate": "exp_voice_urban_1031", "tier": "red", "runs": 5, "avgBest": 0.755, "avgDrop": 0.243 },
    { "candidate": "exp_voice_urban_1031", "tier": null, "runs": 5, "avgBest": 0.848, "avgDrop": 0.067 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "green", "runs": 24, "avgBest": 0.409, "avgDrop": 0.348 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "yellow", "runs": 37, "avgBest": 0.608, "avgDrop": 0.165 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "red", "runs": 941, "avgBest": 0.825, "avgDrop": 0.065 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": null, "runs": 6, "avgBest": 0.896, "avgDrop": 0 },
    { "candidate": "exp_voice_urban_soft_1107g", "tier": "green", "runs": 72, "avgBest": 0.609, "avgDrop": 0.178 },
    { "candidate": "exp_voice_urban_soft_1107g", "tier": "red", "runs": 6, "avgBest": 1, "avgDrop": 0 },
    { "candidate": "exp_xianxia_texture_1123", "tier": "green", "runs": 16, "avgBest": 0.844, "avgDrop": 0.038 },
    { "candidate": "exp_xianxia_texture_1123", "tier": "yellow", "runs": 68, "avgBest": 0.682, "avgDrop": 0.127 },
    { "candidate": "exp_xianxia_texture_1123", "tier": "red", "runs": 46, "avgBest": 0.79, "avgDrop": 0.059 },
    { "candidate": "exp_xianxia_texture_1123", "tier": null, "runs": 72, "avgBest": 0.772, "avgDrop": 0.064 },
    { "candidate": "v_current", "tier": "green", "runs": 605, "avgBest": 0.544, "avgDrop": 0.218 },
    { "candidate": "v_current", "tier": "yellow", "runs": 1232, "avgBest": 0.664, "avgDrop": 0.136 },
    { "candidate": "v_current", "tier": "red", "runs": 910, "avgBest": 0.855, "avgDrop": 0.053 },
    { "candidate": "v_current", "tier": null, "runs": 1124, "avgBest": 0.765, "avgDrop": 0.08 }
  ]
}
```
