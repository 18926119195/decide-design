# decide-design 入库指南

[一分钟定位](#一分钟定位) · [沿聊天时间线看灵感](#沿聊天时间线看灵感) · [从代码反查来由](#从代码反查来由) · [走读与维护记录](#走读与维护记录)

## 一分钟定位

这是独立的名词表选材评测仓。输入是书块语料和人工标注题单，输出是预测材料、命中页与精确率/召回率。原作者想观察同一个词出现在书上不同地方时，附近的其他词能否帮 decide 找到真正有关的材料。阅读基线为 [main · 6a1ac3a](https://github.com/18926119195/decide-design/commit/6a1ac3a853b8a2107a3af248014b07bb029ff0bc)。

## 沿聊天时间线看灵感

**如何读状态。** 已实现表示在所列版本找到与灵感相符的代码路径。部分实现表示已有其中一段。未实现表示已查的入口尚无对应实现。证据不足时标待确认。归档与设计文稿按资料本身的状态描述。

日期按会话中的北京时间；9 月 14 日原件按行号读取。9 月 13 日聊天的 [Markdown](docs/chat-export/decide-design-chat-29c55109.md) 和 [JSONL](docs/chat-export/decide-design-chat-29c55109.jsonl) 是同一会话的两种呈现。

| 编号与聊天原件 | 当时提出的思路 | 当前状态与代码证据 |
| --- | --- | --- |
| <a name="idea-i01"></a>I01 · 9 月 13 日 10:23，[会话 6，L64–71](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/docs/chat-export/decide-design-chat-29c55109.md?plain=1#L64-L71) | 规则先从全书提取名词形成 S₀，遮住已有词后把残文交给模型补 S₁。 | **已实现可选路径。** 构表器先建规则层，评测开启 `residualLlm` 时再调用模型补缺。 [构表和补缺](src/harness/runNounTableMaterial.ts#L314-L348) |
| <a name="idea-i02"></a>I02 · 9 月 13 日 10:36，[会话 9，L147–177](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/docs/chat-export/decide-design-chat-29c55109.md?plain=1#L147-L177) | 把同形词去重后给模型勾选，同时保留同一词每次实际出现的位置。 | **已实现。** 词表按归一后的词面组织，定位函数返回来源与偏移。 [归一与抽取](src/core/bookNounIndex.ts#L230) · [定位命中](src/core/bookNounIndex.ts#L386) |
| <a name="idea-i03"></a>I03 · 9 月 13 日 10:36，[拓扑定义 L168–177](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/docs/chat-export/decide-design-chat-29c55109.md?plain=1#L168-L177) | 每个命中点取前后若干异名词做一个袋，由 decide 选出证据段，最终供推理使用。 | **部分实现。** 拓扑袋、decide 和选材打分已接入。这个独立仓的 CLI 输出评测结果，后续推理留在应用侧。 [邻词袋](src/core/bookNounIndex.ts#L483) · [评测入口](scripts/eval-decide.ts#L64-L101) |
| <a name="idea-i04"></a>I04 · 9 月 13 日 16:02–16:07，[会话 36–38](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/docs/chat-export/decide-design-chat-29c55109.md?plain=1#L596-L644) | 先考虑按目录章节去重以保留异义，随后追问展开每个命中点后，是否还需要章节边界。 | **章内去重未落地。** 当前按全书词面维护词表，再把各处命中交给拓扑。 [全书名词表](src/core/bookNounIndex.ts#L274-L353) |
| <a name="idea-i05"></a>I05 · 9 月 13 日 16:13，[会话 40](https://github.com/18926119195/decide-design/blob/6a1ac3a853b8a2107a3af248014b07bb029ff0bc/docs/chat-export/decide-design-chat-29c55109.md?plain=1#L669-L675) | 把准度验证抽成独立项目，直接拿选材结果和人工金标比较。 | **已实现。** 独立 CLI 和 Web 以语料包为输入，评分留在评测仓。 [评测入口](scripts/eval-decide.ts) · [初始评测提交](https://github.com/18926119195/decide-design/commit/a1ccc04) |
| <a name="idea-i06"></a>I06 · 9 月 14 日，[分批讨论 L422–429](9.14的聊天记录#L422-L429) | 每两个 S₀ 遮罩定一组，组间并行，长组内按顺序接续。 | **未落地于本仓入口。** 原讨论的工具路径指向另一工作目录，本仓监督器按残文批次循环。 [监督批次](src/harness/superviseS0S1.ts#L228-L304) |
| <a name="idea-i07"></a>I07 · 9 月 14 日，[监督与导出 L2170–2176](9.14的聊天记录#L2170-L2176) | 把构表质量和每次迭代依据集中导出，供作者与 AI 比较。 | **已实现导出。** Web 有构表监督和评测监视报告。在固定离线夹具中监督成功，夹具 S₁ 为 0。 [监督路由](src/server/main.ts#L219-L305) · [评测报告组装](src/harness/buildMonitorExport.ts) |

## 从代码反查来由

- `src/core/bookNounIndex.ts` 回看 [I01](#idea-i01)–[I04](#idea-i04)，负责词面、书中位置和异名词袋。
- `src/harness/runNounTableMaterial.ts` 回看 [I01](#idea-i01)、[I03](#idea-i03)，负责从问题、词表和材料一路运行到预测 key。
- `src/harness/superviseS0S1.ts` 与 `src/harness/buildMonitorExport.ts` 回看 [I06](#idea-i06)–[I07](#idea-i07)，分别回答词表补齐与选材诊断。
- `scripts/eval-decide.ts` → `src/harness/score.ts` 回看 [I05](#idea-i05)。旧 [设计契约](docs/book-noun-pipeline-contract.md) 里的 CLI 描述属于早期阶段，当前实际入口见脚本。

## 走读与维护记录

第一次跟踪一个词，直接打开 [冒烟夹具](scripts/smoke-noun.ts)，找第 1 页的 Lacan 和邻近的异名词，再根据 [I03](#idea-i03) 的定位函数看它们怎样入袋。也可运行 `npm run smoke` 查看 `Lacan@p1` 的输出。本次 Node 24 上该夹具和构表监督冒烟通过；真实模型和外部书包未在本次运行。后续修订这一页时，保留原聊天位置与编号，给修正后的状态补确切提交、入口和行为证据。

**指南如何更新。** 以仓库和版本划定范围，从聊天提取原始目标，再用代码与产物校正对当时决策的理解。新证据出现时，回到原编号修订记录。这种读法借鉴了钱学森的系统工程和工程控制论。 [《组织管理的技术——系统工程》](https://doi.org/10.13255/j.cnki.jusst.2011.06.020) · [《工程控制论》](https://openlibrary.org/books/OL49218199M/Engineering_cybernetics)
