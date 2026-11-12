---
name: iot-scenario-delivery
description: "Use this skill when a contest/assignment asks for a complete 'scenario innovation and design' IoT monitoring-system deliverable in one sitting — a runnable multi-site monitoring web system plus project demo PPT, draw.io application topology, full source zip, device-selection DOCX/XLSX, AI-prompt iteration log, demo script and screenshots. Triggers include 模块C、场景创新与设计、VibeCoding、赛题、参赛交付、物联网监测系统、设备选型说明、AI提示词全过程记录、应用拓扑工程文件、项目演示PPT、完整源码、智慧矿山/园区/工厂安全监测. It scaffolds a proven zero-dependency Node.js + vanilla-JS monitoring platform from a short brief.json, then generates every document and artifact from one config so the whole package is finished in about 30 minutes."
---

# 竞赛式物联网监测系统全流程交付

## 目的

把「读赛题 → 做系统 → 写文档 → 做 PPT → 出拓扑 → 截屏 → 打包」这一整套重复劳动压缩成
**填一份 `brief.json` + 跑两条命令**。已验证的完整骨架（约 1 万行、6 个页面、告警引擎、
演练沙盘、SSE 实时推送）与全部文档生成器都随本 skill 提供，不需要重新设计或从零写代码。

## 何时使用

- 赛题/作业要求「自主完成需求分析、设备选型、功能开发、可视化展示与功能演示」并列出
  PPT、应用拓扑工程文件、完整源码、设备选型说明、AI 提示词全过程记录等提交材料。
- 需要在半小时内产出一套**可离线运行、可现场演示**的多站点监测系统与配套交付包。
- 类似主题复用：智慧矿山、化工园区、智慧工厂、隧道/管廊、粮仓、数据中心、楼宇等
  「多点位 + 分级告警 + 联动处置」场景。

## 不可动摇的硬约束

1. **稳定数据契约**：后端告警与仿真逻辑依赖 8 个固定测点槽位
   `ch4 / ch4b / co / o2 / temp / humidity / dust / wind`，方向分别为
   `up / up / up / low / up / up / up / range`。**key 与 direction 不可改**；
   `name / short / unit / 阈值 / 量程 / 型号 / 描述 / 位置` 可自由改。
   场景 key `gas / co / vent / dust / normal` 亦不可改，只改显示名、描述与时长。
2. **结构固定**：3 个站点 × 每站 3 个区域 × 每区 8 个测点 = 72 测点。改站点/区域名称与数量
   通过 `domain.sites`，但保持 `faces[].id` 唯一且为 ASCII。
3. **零第三方运行时依赖**：`npm start` / `npm test` 必须直接可跑，不联网装包。
   文档生成用 Python 库（python-docx / python-pptx / openpyxl / Pillow），由脚本自动定位解释器。
4. **唯一数据源**：`config/domain.json`。禁止在文档或 PPT 里手写数量、阈值、型号；
   改数据后必须全量重跑 `build_all.py`，否则材料与系统口径不一致。
   改阈值/量程时必须同步 `simulation.scenarioTargets` 与 `clamps`（见 `brief-schema.md`），
   否则演练注入值越不过阈值，现场演示不产生告警。
5. **中文文件一律用 Python 显式 UTF-8 读写**。禁止用 Windows PowerShell 5.1 的
   `Get-Content -Raw` / `Set-Content` 改写 UTF-8 中文文件（会静默乱码）。

## 30 分钟标准流程

| 时段 | 动作 | 产出 |
| --- | --- | --- |
| 0:00–0:04 | 读赛题，列出交付物与场景名词 | 主题要点 |
| 0:04–0:10 | 复制 `assets/brief.mine.json` 或 `brief.park.json`，只改 `project` / `glossary` / `content` 的痛点与创新点 | `brief.json` |
| 0:10–0:12 | 运行 `new_project.py` | 可运行项目骨架 + 全部 Markdown 文档 |
| 0:12–0:15 | 运行 `build_all.py`（约 2 分钟） | 全部 7 类交付物 + 9 张截图 |
| 0:15–0:23 | 抽查 3 张截图、PPT 首页/末页、check 报告；按赛题补写 `content.qa` / `content.promptVersions` | 针对性微调 |
| 0:23–0:27 | 修改 `config/domain.json` 的阈值/设备参数（若赛题指定），重跑 `build_all.py` | 口径一致版 |
| 0:27–0:30 | `npm start` 现场走一遍演示路径，确认 `交付清单.md` | 交付 |

**不要**把时间花在：重写前端样式、手写 PPT 版式、逐页调 Word 排版、手动截屏、
手工打 zip。这些由脚本完成；只把时间花在赛题解读与内容判断上。

## 两条核心命令

设 `SKILL` 为本 skill 目录（Windows 通常为 `C:\Users\<用户>\.zcode\skills\iot-scenario-delivery`）。

```bash
# 1) 由 brief.json 生成项目（<OUT> 为目标目录，例如 ./park-safety-iot）
python "$SKILL/scripts/new_project.py" --brief ./brief.json --out <OUT> --force

# 2) 一键生成全部交付物（自动起服务、CDP 截图、出文档/表格/PPT/拓扑、冒烟、打包、核验）
python "$SKILL/scripts/build_all.py" --project <OUT>

# 迭代期加速：跳过截图（用占位图）/ 跳过 DOCX
python "$SKILL/scripts/build_all.py" --project <OUT> --skip-screenshots --quick
```

生成后运行项目：

```bash
cd <OUT> && npm start        # http://127.0.0.1:<project.port>
cd <OUT> && npm test         # 告警引擎 / 演练 / 历史查询
```

## brief.json 填写要点

最小可用文件只需 `project`（9 个字段）+ `glossary` + `content.background/pain/innovations`。
其余字段（`goals / values / limits / qa / demo / promptVersions / defects / architecture / roadmap`）
有基于实时配置的完整默认值，赛题有明确要求时再覆盖。

- `glossary`：**换主题的关键**。按「长词优先」对复制后的所有文本文件、主数据、脚本文案做替换。
  例：`矿井→园区`、`工作面→装置区`、`瓦斯→可燃气体`、`甲烷→可燃气`、`矿用→工业级`、
  `深地智安→安芯智控`。参考 `assets/brief.park.json`。
- `domain`：可选，对 `config/domain.json` 做深度覆盖（dict 递归合并，list 整体替换）。
  赛题指定了站点名、阈值或设备清单时在这里写；未指定的部分沿用默认。
- `content.promptVersions` / `content.defects`：赛题要求真实记录，**务必把本次对话中的
  真实提示词版本与实际缺陷补进去**再重建文档；模板默认值只是结构与示例。
- 完整字段说明见 `references/brief-schema.md`。

## 交付物与验收

`build_all.py` 产出（均在 `<OUT>/deliverables/`）：

| 交付物 | 文件 |
| --- | --- |
| 项目演示 PPT（14 页） | `项目演示.pptx` |
| 应用拓扑工程文件 | `topology/application-topology.drawio` + `.json` |
| 项目完整源码 | `项目完整源码.zip` |
| 设备选型说明 | `设备选型说明.docx` + `设备选型说明.xlsx`（6 工作表） |
| AI 提示词全过程记录 | `AI提示词全过程记录.docx` |
| 演示脚本与答辩要点 | `演示脚本与答辩要点.docx` |
| 系统截图 | `screenshots/01…09*.png` |
| 一键提交包 | `<项目名>-参赛交付包.zip` |

自动核验：`config/check-report.json`（文件存在且非空、PPT/DOCX/XLSX 可解析、源码包内容、
README 项目名一致）；接口冒烟见控制台 `SMOKE OK`；截图与 JS 异常见 `config/build-report.json`。
`check.py` 中 **required 项全部 OK** 才可交付。

## 失败回退

| 现象 | 处理 |
| --- | --- |
| Edge CDP 起不来（沙箱阻止命名管道） | 加 `--skip-screenshots`，PPT 用占位图；或在允许的环境重跑一次 `build_all.py` |
| 提示缺 Python 库 | `python -m pip install python-docx python-pptx openpyxl Pillow`，或指向 DSH 内置 Python |
| 端口被占用 | 改 `brief.json` 的 `project.port`，重新 `new_project.py` 或在 `config/domain.json` 修改后重跑 |
| 截图是旧界面 | `capture.mjs` 已禁用缓存并加时间戳；仍异常时删 `deliverables/screenshots/` 重跑 |
| 文档与系统数值不一致 | 只改 `config/domain.json`，然后**全量**重跑 `build_all.py` |

## 附带的知识

- `references/workflow-30min.md`：时间盒 SOP、命令速查、现场演示路径。
- `references/delivery-spec.md`：赛题评分点、每份材料必须包含的章节。
- `references/pitfalls.md`：从一次真实 260 步会话中提炼的踩坑清单（编码、缓存、CDP、
  字段覆盖、PPT 对比度、Excel 单位混算等），调试前先读。
- `references/visual-system.md`：配色、字号、版式规范，微调 UI 时遵守。
- `assets/examples/`：上一版获奖级文档范文（README / 三份文档 / 交付清单 / domain 样例）。
- `assets/starter/`：完整可运行骨架，配置驱动，已通过 `npm test`。
