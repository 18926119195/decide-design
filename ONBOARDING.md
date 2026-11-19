# decide-design 入库指南

这个仓库评估从一本书里选出的材料是否命中人工标注。先运行离线夹具，再跟踪一条问题从书块进入名词表、候选袋和打分的完整路径。审阅基线为 [main · 6a1ac3a](https://github.com/18926119195/decide-design/commit/6a1ac3a853b8a2107a3af248014b07bb029ff0bc)。

## 从这里开始

1. 读 [9 月 13 日导出的协作记录](docs/chat-export/decide-design-chat-29c55109.md) 和它的 [JSONL 原件](docs/chat-export/decide-design-chat-29c55109.jsonl)。先看“锁定热路径”及用户确认的拓扑袋。两份文件是同一次会话的两种呈现。
2. 从 [语料解析](src/harness/loadCorpus.ts) 读到 [评测 CLI](scripts/eval-decide.ts)，沿 [runNounTableMaterial](src/harness/runNounTableMaterial.ts) 到 [scoreCase](src/harness/score.ts)。Web 的入口是 [src/server/main.ts](src/server/main.ts) 的 `/api/run`，参考 [运行说明](README.md)。
3. 查 [构表监督](src/harness/superviseS0S1.ts) 和 [监视报告](src/harness/buildMonitorExport.ts)。`fixtures/` 放题单示例，`results/` 放评测产物和早期监督样本。Gold 一律以本次题单核对。

## 从聊天到代码

| 时间与证据 | 原意与当前实现 |
| --- | --- |
| 9 月 13 日，[初始评测 harness](https://github.com/18926119195/decide-design/commit/a1ccc04) 与 [聊天归档提交](https://github.com/18926119195/decide-design/commit/0cd5b66) | 建立书块名词层、匹配命中、拓扑邻词、decide 与 gold 评分。代码位于 [bookNounIndex](src/core/bookNounIndex.ts) 和 [评测编排](src/harness/runNounTableMaterial.ts)，CLI 当前实际走名词表。 |
| 9 月 14 日，[分批讨论 L422](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/9.14%E7%9A%84%E8%81%8A%E5%A4%A9%E8%AE%B0%E5%BD%95#L422) | 用户明确提出两个 S₀ 遮罩分组、组间并行、组内长包串联。记录指向另一个本机工作目录。本仓监督器按残文批次推进，尚未找到该组间并行、组内串行协议的可执行接线。 |
| 9 月 14 日，[停闸 L2012 与导出要求 L2170](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/9.14%E7%9A%84%E8%81%8A%E5%A4%A9%E8%AE%B0%E5%BD%95#L2012-L2023) | 用户把停闸的 Δn 定义为本轮硬闸通过的抄写条数，且要求一键导出诊断。当前服务已有监督 JSON/Markdown 和 monitor。监督器有轮数上限及若干停止条件，Δn 的同义契约仍需逐轮夹具对齐。 [导出入口](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/9.14%E7%9A%84%E8%81%8A%E5%A4%A9%E8%AE%B0%E5%BD%95#L2170-L2176) |

[早期契约](docs/book-noun-pipeline-contract.md) 对 CLI 走既有 decide 的描述用于理解当时的设计，当前评测读取实际调用。此库适合固定为早期对照。[9 月 16 日的 main 快照](https://github.com/18926119195/decide-design-main)增加了偏移量试验，基线成绩需要单独记录。

## 观测、判据与纠偏

- **参考目标**：每个预测 key 都能回到输入书块。固定语料版本、题单、gold 模式、模型和 `neighborN`。记录命中数、P/R、预测页及 bookKey，并同时保留候选和决定。
- **扰动**：中英文别名、取词规则、OCR 噪声、空残文和模型超时。先重跑规则阶段，以 `S₀`、候选袋和输出 key 拆开定位偏差。模型参与时增加批次和耗时记录。
- **门限**：确定性夹具的预测 key 皆属于输入，外来 key 应为 0。性能比较采用同一题单的 P、R 和失败数，允许的折损、预算及接受取舍由评测维护者在开跑前确认。
- **执行顺序**：先补可分享的最小书包和正负 gold，再为残文新增、零新增、坏响应编写断言。确认停闸规则后才替换调度。若精度下降，按 S₀、S_q、候选袋、decide、score 的次序找到第一处偏差，将同一题的预期、实测和纠偏补入 PR。

```sh
npm run smoke
node --experimental-strip-types scripts/smoke-supervise.ts
```

本次在 Node 24.22.1 实跑，两条命令均通过。规则样例得到 4 个命中和 4 个袋，监督夹具输出 `|S₁|=0`，只覆盖该固定样例。真实 LLM、外部语料和 Web 导出未在本次复核跑通。准备好脱敏语料后可运行 `npm run eval -- --corpus <book.docuverse.json> --eval fixtures/eval.example.json`，将尖括号占位替换为实际路径。

## 方法依据

本指南把总体设计落实为边界、目标和任务接口，把工程控制中的反馈落实为观测、判据和纠偏。维护者用原始协作记录、代码和固定样本共同核验定性判断与量化结果，据此决定下一轮行动。[《工程控制论》](https://openlibrary.org/books/OL49218199M/Engineering_cybernetics) · [《组织管理的技术——系统工程》](https://doi.org/10.13255/j.cnki.jusst.2011.06.020) · [《一个科学新领域——开放的复杂巨系统及其方法论》](https://cnki.istiz.org.cn/kcms/detail/detail.aspx?dbcode=CPFD&dbname=cpfd1999&filename=ZGXC199008001001)
