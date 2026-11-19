# docuverse-decide-eval · 名词表准度

首次接手可从[入库指南](ONBOARDING.md)了解仓库用途，沿聊天追踪灵感与实现状态，也可从代码反查设计来由。

独立评测仓（**不改**主仓 `docuverse-shanchule-peek-main`）。

## 默认策略：名词表

```
建 S（规则去重）→（可选）残文 LLM 补 S₁
→ 勾选 S_q → 字符串定位 → ±n 异名词拓扑袋 → decide 过闸 → 对照 gold
```

## 启动 Web（导入语料）

```powershell
cd C:\Users\Administrator\Desktop\docuverse-decide-eval
# 复制 .env.example → .env 并填 LLM_API_KEY
& "C:\Program Files\cursor\resources\app\resources\helpers\node.exe" --experimental-strip-types src/server/main.ts
```

打开：**http://127.0.0.1:5188/**

1. 上传主仓导出的 `*.docuverse.json`  
2. 填 gold 题（`goldPages` / `goldBookKeys`）  
3. 「跑名词表评测」

## CLI

```powershell
node --experimental-strip-types scripts/eval-decide.ts --corpus path\to\book.docuverse.json --eval fixtures\eval.example.json
```

可选：`--residual-llm`

## 冒烟（无 LLM）

```powershell
node --experimental-strip-types scripts/smoke-noun.ts
```
