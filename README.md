# 科研工作区

这是一个轻量科研协作目录。通用方法材料放在 `docs/`，具体课题的状态、方案和实验资产放在 `projects/`。

## 当前项目

- [FocalAfford](projects/20260922-FocalAfford/README.md)
- 当前状态：课题已确定，尚无本方法实验结果。
- 当前任务：先运行局部教师 A0-A4 快速对照，再由结果决定是否进入学生蒸馏验证。
- 当前协议：[projects/20260922-FocalAfford/experiments/README.md](projects/20260922-FocalAfford/experiments/README.md)

每个项目的 `README.md` 是该项目唯一状态入口。根目录不平行维护具体课题状态。

## 新建项目

确定题目名称后运行：

```powershell
PowerShell -ExecutionPolicy Bypass -File .\scripts\new_topic.ps1 `
  -Name "题目名称" -Venue "候选会议" -Deadline "YYYY-MM-DD"
```

脚本只创建项目 `README.md`、`EVIDENCE.md` 和 `DECISIONS.md`。实验、论文与代码目录等真正需要时再建立。

## 默认工作方式

1. 先把研究问题、可观察 failure 和停止条件写清楚；
2. 查最接近、最可能覆盖主张的论文与官方代码；
3. 先做最便宜、最可能推翻假设的检查；
4. 证据支持后才扩大实验或设计方法；
5. 只记录改变研究判断的事实，避免为流程制造文档。

## 目录

- `AGENTS.md`：不可妥协的研究、协作与文件归属规则；
- `projects/`：实际研究项目。每个项目的 `README.md` 是唯一状态入口；idea、实验、论文和归档均留在项目内部；
- `docs/`：仅保存可跨项目复用的科研方法、Agent 协作和论文实操经验；
- `templates/`：项目、证据、决策和实验记录模板；
- `scripts/`：新建项目与项目完整性校验脚本。

判断文件位置时先问：它是否只服务于一个课题？如果是，就放入对应的 `projects/<日期>-<课题名>/`，不要放在根目录或 `docs/`。

旧题目和旧实验留在各自仓库或归档目录，本仓库不保存平行副本。
