# 朱雀候选池数据优化报告（2026-11-19）

- 运行环境：生产库 wordflower-cn（cn-hangzhou），启动后窗口 2026-11-07 06:52 UTC → 11-19 01:35 UTC，约 11.8 天
- 判据来源：`.agents/skills/zhuque-optimization` PR-C/PR-D；口径铁律见 `zhuque-system`
- **本期结论：v2 red 层连续第四期无触发——Δbest 首次转负（−0.005，优于基线）、Δdrop 连续三期为正；效应小而稳，轮换线（+0.03）未达 → 不动作，不写库。**

## 一、配置现状

updated_at 2026-11-13 11:21:43 UTC（人工退役 1031 后未再变动，无新干预；两候选指令 md5 完好）：

| 候选 | weight | enabled | tiers | 角色 |
|---|---|---|---|---|
| v_current | 0.5 | true | [] | 基线臂（全档位） |
| exp_voice_urban_1031 | 0 | false | green, yellow | 人工退役态（数据冻结于 11-13 11:06） |
| exp_voice_urban_soft_1107 | 0.5 | true | red | red 实验臂 |

## 二、采用信号

- texture 信号 N/A（对照臂已退役）
- tiers 采用：v2 red 臂无旧客户端污染（1031 退役后残尾自然终止）

## 三、分层统计（启动后累计窗口，都市层）

| 候选 | 层 | runs/books | avg_best | Δbest | avg_drop | Δdrop | worse_after | 门槛 |
|---|---|---|---|---|---|---|---|---|
| **v2** | **red** | **586 / 110** | 0.805 | **−0.005** | 0.083 | **+0.010** | 0.242 vs 0.233 | **双过 ✓** |
| v2 | green/yellow/null（残尾泄漏） | 24/32/19 | — | — | — | — | — | 未过 |
| 1031（已退役存档） | green/yellow | 107/35、168/45 | 0.527/0.674 | −0.013/+0.009 | 0.185/0.124 | +0.033/+0.017 | — | 存档 |
| 1031（已退役存档） | red/null | 27/8、20/18 | 0.863/0.752 | — | 0.102/0.098 | — | — | 未过 |
| v_current | green/yellow/red/null | 450/76、713/97、635/99、870/408 | 0.525/0.663/0.810/0.751 | — | 0.234/0.141/0.073/0.085 | — | 0.247/0.286/0.233/0.260 | 参照臂 |

## 四、判定

### v2 red 层——四期轨迹：小而稳的正向，未达轮换线

| 期 | Δavg_best | Δavg_drop | 第 2 轮超出基线 |
|---|---|---|---|
| 11-10 | +0.012 | +0.019 | +0.024 |
| 11-13 | +0.026 | +0.011 | +0.005 |
| 11-16 | +0.004 | +0.011 | +0.019 |
| **11-19** | **−0.005** | **+0.010** | **+0.009** |

- 熔断：Δbest −0.005（优于基线方向）、Δdrop +0.010 → 不满足
- 轮换：净效应 −0.005/+0.010 → Δdrop 需 ≥+0.03，差 0.020 → 不满足
- 逐轮：v2 R1 +0.043 vs 基线 +0.046；v2 R2 +0.033 vs 基线 +0.024——第 2 轮保持 +0.009 优势
- **判读**：效应形态极其稳定——best_conf 在 ±0.005 内波动（本期首次转负）、drop 连续三期 +0.010~+0.019、第 2 轮持续微优。这是一个**小而真实的叠加项效应**，但量级（~0.01）只有轮换线（+0.03）的三分之一。按规则继续观察

### 1031——退役态，数据冻结

无新 run（末次 11-13 11:06）。历史存档读数不变。

## 五、写库验证

本期无配置变更，未写库。

## 六、待办与观察点（下期 11-22）

1. **轮换线对小效应候选的可达性（政策问题，提请人工裁断）**：v2 连续四期 Δdrop +0.010~+0.019 稳定为正、Δbest ≈ 0，但轮换线 +0.03 是按纹理候选（效应 ~0.1）校准的，对叠加项候选可能永远够不着。现状成本：red 层一半流量持续拿不到这个小收益。选项：① 维持规则不动（保守，继续 50/50）；② 人工裁断将 v2 权重上调（如 0.7，基线仍保留对照）——属人工动作，任务不自动执行
2. **v2 继续观察**：若 Δbest 转差并逼近 +0.05 触发熔断，若 Δdrop 站上 +0.03 触发轮换；两线均远，单期波动不动作
3. **例行任务 11-18 档未跑**（analysis_summary 仍为 11-16 09:08 的数据，evolution-sync 最新为 11-15 期）——遥测类任务仍在跑（候选池本期正常），疑似例行复跑任务漏触发或失败，建议在交互会话核查 automation-fbaa4bcc 状态
4. **声口卡验收读数**（本期手动跑数，第四期）：red 13 本 0.803→0.850（6/13 改善）、yellow 9 本 0.647→0.583（5/9 改善）——GEPA4 靶子不变
5. 例行数据（11-16 09:08，本期无刷新）：chapters clean 14637、episodes clean 4066、fewshot 1709 篇
6. 全池回退：无需

## 七、上期对比

- 上期（11-16）：v2 漂移反转（+0.026→+0.004）、第 2 轮优势恢复；人工退役 1031 记录在案
- 本期：v2 Δbest 首次转负（−0.005）、drop 连续三期为正——小而稳的叠加效应确认；轮换线未达 → 继续观察

## 机读数据

```json
{
  "reportDate": "2026-11-19",
  "pool": [
    { "id": "v_current", "weight": 0.5, "enabled": true },
    { "id": "exp_voice_urban_1031", "weight": 0, "enabled": false },
    { "id": "exp_voice_urban_soft_1107", "weight": 0.5, "enabled": true }
  ],
  "telemetry": [
    { "candidate": "exp_voice_urban_1031", "tier": "green", "runs": 107, "avgBest": 0.527, "avgDrop": 0.185 },
    { "candidate": "exp_voice_urban_1031", "tier": "yellow", "runs": 168, "avgBest": 0.674, "avgDrop": 0.124 },
    { "candidate": "exp_voice_urban_1031", "tier": "red", "runs": 27, "avgBest": 0.863, "avgDrop": 0.102 },
    { "candidate": "exp_voice_urban_1031", "tier": null, "runs": 20, "avgBest": 0.752, "avgDrop": 0.098 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "green", "runs": 24, "avgBest": 0.479, "avgDrop": 0.221 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "yellow", "runs": 32, "avgBest": 0.55, "avgDrop": 0.213 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "red", "runs": 586, "avgBest": 0.805, "avgDrop": 0.083 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": null, "runs": 19, "avgBest": 0.776, "avgDrop": 0.064 },
    { "candidate": "v_current", "tier": "green", "runs": 450, "avgBest": 0.525, "avgDrop": 0.234 },
    { "candidate": "v_current", "tier": "yellow", "runs": 713, "avgBest": 0.663, "avgDrop": 0.141 },
    { "candidate": "v_current", "tier": "red", "runs": 635, "avgBest": 0.81, "avgDrop": 0.073 },
    { "candidate": "v_current", "tier": null, "runs": 870, "avgBest": 0.751, "avgDrop": 0.085 }
  ]
}
```
