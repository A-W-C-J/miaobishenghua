# 朱雀候选池数据优化报告（2026-11-22）

- 运行环境：生产库 wordflower-cn（cn-hangzhou），启动后窗口 2026-11-07 06:52 UTC → 11-22 01:00 UTC，约 14.7 天
- 判据来源：`.agents/skills/zhuque-optimization` PR-C/PR-D；口径铁律见 `zhuque-system`
- **本期结论：v2 red 层连续第五期无触发（效应稳在「best ≈ 平、drop 微正、第 2 轮微优」）→ 不动作，不写库。例行复跑任务的 analyze 步 OOM 已诊断并手工恢复（详见待办）。**

## 一、配置现状

updated_at 2026-11-13 11:21:43 UTC（人工退役 1031 后未再变动，无新干预；两候选指令 md5 完好）。

## 二、采用信号

- texture 信号 N/A（对照臂已退役）
- tiers 采用：v2 red 臂无旧客户端污染

## 三、分层统计（启动后累计窗口，都市层）

| 候选 | 层 | runs/books | avg_best | Δbest | avg_drop | Δdrop | worse_after | 门槛 |
|---|---|---|---|---|---|---|---|---|
| **v2** | **red** | **766 / 129** | 0.827 | **−0.003** | 0.069 | **+0.007** | 0.228 vs 0.226 | **双过 ✓** |
| v2 | green/yellow/null（残尾泄漏） | 33/41/23 | — | — | — | — | — | 未过 |
| 1031（已退役存档） | green/yellow | 107/35、168/45 | 0.527/0.674 | −0.013/+0.009 | 0.185/0.124 | +0.033/+0.017 | — | 存档 |
| v_current | green/yellow/red/null | 576/91、901/116、806/116、1044/491 | 0.532/0.661/0.830/0.752 | — | 0.222/0.139/0.062/0.087 | — | 0.233/0.287/0.226/0.256 | 参照臂 |

## 四、判定

### v2 red 层——五期轨迹：小而极稳的正向，轮换线仍未达

| 期 | Δavg_best | Δavg_drop | 第 2 轮超出基线 |
|---|---|---|---|
| 11-10 | +0.012 | +0.019 | +0.024 |
| 11-13 | +0.026 | +0.011 | +0.005 |
| 11-16 | +0.004 | +0.011 | +0.019 |
| 11-19 | −0.005 | +0.010 | +0.009 |
| **11-22** | **−0.003** | **+0.007** | **+0.009** |

- 熔断：Δbest −0.003、Δdrop +0.007 → 不满足
- 轮换：净效应 −0.003/+0.007 → Δdrop 需 ≥+0.03，差 0.023 → 不满足
- 逐轮：v2 R1 +0.034 vs 基线 +0.036；v2 R2 **+0.033** vs 基线 +0.024——第 2 轮超出 +0.009，与上期持平
- still80：v2 65.4% vs 基线 65.1%（平）；worse_after 0.228 vs 0.226（平）
- **判读**：五期 145 本配对证据下，v2 的效应形态高度稳定——best_conf 在 ±0.005 内、drop 连续五期为正（+0.007~+0.019）、第 2 轮持续 +0.005~+0.024 优势。真实但小（~0.01 量级），轮换线（+0.03）按纹理候选校准，对叠加项候选可达性存疑（政策问题已连续两期提请人工裁断，未见处置——本期继续挂起）

## 五、写库验证

本期无配置变更，未写库。

## 六、待办与观察点（下期 11-25）

1. **例行任务 analyze 步 OOM——已诊断并手工恢复，永久修复待交互会话**：11-22 09:00 例行复跑的导出成功（chapters.jsonl 6.2GB，流式没问题），但 analyze.mjs 整文件读入内存数组，超出 Node 默认 ~4GB 堆直接 OOM（日志实锤：JavaScript heap out of memory @ JSON.parse），导致 analysis_summary/fewshot 冻结在 11-16。本期已用 `node --max-old-space-size=16384 analyze.mjs` 手工恢复（机器 48GB）：chapters clean 18819（上期 14637）、episodes clean 5013、fewshot 2243 篇。**永久修复**：在交互会话把例行任务第 4 步改为 `node --max-old-space-size=16384 analyze.mjs`（或把 loadJsonl 改流式）——候选池任务无法在运行中自改其他任务
2. **轮换线可达性（政策问题，第三次提请）**：v2 五期 Δdrop +0.007~+0.019 稳定为正但够不着 +0.03；成本是 red 层半数流量持续拿不到这小收益。选项：维持规则 / 人工上调 v2 权重至 0.7。任务不自动执行
3. **机读 JSON 契约已上线**：新增的 11:00「朱雀迭代实录同步」任务（OSS 数据集 + GitHub 档案）消费本报告的「## 机读数据」节——本期 JSON 随报告落盘，11:00 同步任务取用；该节为刚性要求，每期必须带
4. 例行数据（11-22 10:05 恢复刷新）：chapters clean 18819、episodes clean 5013、fewshot 2243 篇
5. 全池回退：无需

## 七、上期对比

- 上期（11-19）：v2 Δbest 首次转负；例行任务漏跑疑云
- 本期：例行任务 09:00 实际跑过但 analyze OOM（已恢复）；v2 五期稳态确认；自动化家族重组（例行任务改写 + 实录同步任务新增）适配完成

## 机读数据

```json
{
  "reportDate": "2026-11-22",
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
    { "candidate": "exp_voice_urban_soft_1107", "tier": "green", "runs": 33, "avgBest": 0.487, "avgDrop": 0.239 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "yellow", "runs": 41, "avgBest": 0.557, "avgDrop": 0.198 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": "red", "runs": 766, "avgBest": 0.827, "avgDrop": 0.069 },
    { "candidate": "exp_voice_urban_soft_1107", "tier": null, "runs": 23, "avgBest": 0.788, "avgDrop": 0.053 },
    { "candidate": "v_current", "tier": "green", "runs": 576, "avgBest": 0.532, "avgDrop": 0.222 },
    { "candidate": "v_current", "tier": "yellow", "runs": 901, "avgBest": 0.661, "avgDrop": 0.139 },
    { "candidate": "v_current", "tier": "red", "runs": 806, "avgBest": 0.83, "avgDrop": 0.062 },
    { "candidate": "v_current", "tier": null, "runs": 1044, "avgBest": 0.752, "avgDrop": 0.087 }
  ]
}
```
