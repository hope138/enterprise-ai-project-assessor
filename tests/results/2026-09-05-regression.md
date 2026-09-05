# 2026-09-05 三结论回归测试

## 测试目的

验证修复后的 Skill 能否针对三个决策对象稳定输出 `GO`、`VALIDATE-FIRST` 和 `NO-GO`，并正确处理审批范围、证据分类、五道门、一票否决以及“本阶段投资合理性/规模化 ROI”双层判断。

## 修复版本

本轮测试包含以下逻辑修复：

1. 决策对象不成立时输出 `EVIDENCE-NEEDED / NOT-ISSUED`，不再使用 `VALIDATE-FIRST` 冒充项目结论。
2. 证据分类改为 `verified_fact`、`documented_claim`、`stakeholder_claim`、`assumption`、`unknown` 和 `conflict`；PRD、逐字稿和汇报数字不再自动升级为已验证事实。
3. 第四门区分 `stage_investment_status` 与 `scale_roi_status`；阶段投入通过不等于规模化 ROI 通过。

## 测试结果

| 测试 | 输入 | 预期结论 | 实际结论 | 关键范围 | 结果 |
|---|---|---|---|---|---|
| T01 | 服饰商品属性提取与上架 Copilot | `GO` | `GO` | 一个女装品牌、8 周、10% 新增 SKU、人工确认 | 通过 |
| T02 | 服饰趋势预测与款式设计 Copilot | `VALIDATE-FIRST` | `VALIDATE-FIRST` | 不批准 30 万元大范围 POC；收敛到一个品类、周期和节点 | 通过 |
| T03 | 电商跨渠道全自动定价 Agent | `NO-GO` | `NO-GO` | 否决三个渠道、约 8,000 SKU、无人工确认的生产写权限 | 通过 |

自动核对了 20 项决定性断言，失败数为 `0`。

## P0 修复专项结果

| 检查项 | T01 | T02 | T03 |
|---|---|---|---|
| 文档数字是否避免标为 `verified_fact` | 通过 | 通过 | 通过 |
| 本阶段投入与规模化 ROI 是否分开 | 阶段 `PASS`；规模化 `NOT-ASSESSABLE` | 两者均 `NOT-ASSESSABLE` | 阶段 `FAIL`；规模化 `NOT-ASSESSABLE` |
| 是否只在决策对象清楚时签发项目结论 | 通过 | 通过 | 通过 |

## 测试完整性说明

本轮由同一任务完成规则修复、生成输出和断言核对，属于功能回归测试。执行上下文已经知道预期断言，因此不能替代独立盲测。发布前应在三个全新任务中只提供 Skill 和原始输入，不提供 `tests/expected/`，再核对输出是否仍然一致。
