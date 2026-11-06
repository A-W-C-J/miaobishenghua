# zhuque-evolution

朱雀（AI 文本检测）对抗迭代的公开数据集与逐期审计档案。官网可视化：
[wordflower.cn/evolution](https://wordflower.cn/evolution)（与本目录同源，每 3 天同步）。

## 数据集

`dataset/evolution.json`（schema 1）：

- `generations[]` — 修订指令代际（iter8 / GEPA2 / GEPA3 / other_revise / no_revise）终检 softmaxConfidence 分布：`n / median / p10 / p90 / lt05 / ge08`
- `dailyFirstPass[]` — 逐日首次检测通过章的中位 conf（no_revise 层）
- `genreReference{}` — 题材参照表：书级 p25 / p50 / p75 分位
- `candidatePool` — 候选池配置（id / weight / enabled）与 A/B 遥测（candidate × book_tier：avg_best / avg_drop / pass02 / still80）
- `history[]` — 逐期核心指标快照

## 口径

- 干净集：`updated_at − detectedAt ∈ [−10min, 1h]` 且正文 > 200 字；检测器对同文本确定性，存量分数即精确标签
- 书级聚合：题材 / 声口结论按 novel_id 先取书内成功检测中位再分桶，章节级聚合会被单本大书主导
- 择优策略：#1242（2026-10-26）后终稿取多轮 softmax 最低轮，#1977（2026-10-31）起取 humanRatio 最高轮，跨时点代际对比含选择偏差
- 判定线（PR-D）：同 book_tier 层 n ≥ 50 且 Δavg_best ≥ 0.05 或 Δavg_drop ≥ 0.03；题材分层另需 ≥ 8 本
- 遥测仅匿名聚合数字：无正文、无用户标识、无书名、无指令全文

## 更新

每 3 天由数据管线自动刷新（export → analyze → 候选池判定 → 本同步）；`dataset/history/` 与 `reports/` 只增不改。

## 目录

- `dataset/evolution.json` — 最新一期数据集
- `dataset/history/` — 逐期快照存档
- `reports/` — 候选池判定审计报告原文

## 本期（2026-11-06）

- 章节干净集 6259 · 剧集干净集 1040 · conf &lt; 0.2 过检范文 506 篇
- 候选池报告：2026-11-04 —— red 层样本过线但效应未达熔断/轮换线 → 不动作，不写库。人工已将权重调整为 0.5/0.5（启动后 7 分钟），指令文本校验完好。

## License

[MIT](../LICENSE)

## Acknowledgments

- [DSPy](https://github.com/stanfordnlp/dspy) — 提示词优化的声明式编程框架，代际指令迭代的基础设施
- [GEPA](https://github.com/gepa-ai/gepa) — Reflective prompt evolution，GEPA2 / GEPA3 代指令的优化方法来源
