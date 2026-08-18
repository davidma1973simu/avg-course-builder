# AVG Course Builder —— 课程→剧本杀式 AVG 场景化学习平台

> 把企业培训课程自动化转换成「剧本杀式」多人场景化学习产品（**线下产品包**）的 DIY 设计平台。
> 核心用户：乙方讲师 / 课程公司 / 甲方 L&D·人才发展专家。价值锚点：**速度 / 质量 / 成本**碾压传统手工剧本杀与定制严肃游戏。

---

## 这是什么

一份基于两份源文档（《设计全景图》《产品和技术规范》）做的研究与产品设计产出：

- **MRD** —— 市场需求文档（市场、痛点、竞品、商业模式）
- **PRD** —— 产品需求文档（12 阶段 Pipeline、AVG Project JSON Schema、Agent 架构、Validation 门禁）
- **Task Backlog** —— 开发任务清单（Epic / 用户故事 / 验收标准 / 估算）
- **Competitor Comparison** —— 竞品功能对比详表（含"课程→剧本杀"产出物赛道）
- **Capability Assets** —— AVG×剧本杀 设计能力资产库（可调用 Skill）
- **Core_Design_Assumption** —— 核心设计假设（锁定）：远场景 + 强制应用迁移桥
- **Sample_Input_Checklist** —— 样本课程输入清单（可编辑 Word / PPT：空白模板 + 情境领导示范）
- **validation_rules.json** —— 38 条可机读自动校验规则（供 Validator Agent 消费，含 VR-RE05 应用迁移桥）

**核心设计假设（锁定 · 不可动摇）**：AVG 探索世界默认采用「远」（虚构类比）情景；无论近 / 远，复盘后**必须强制「应用迁移桥」**把所学迁移到学员自身真实工作场景。**没有这条，学习与产品不成立**（校验 VR-RE05，BLOCKER）。

定位澄清（v2 起）：平台承载**设计功能**，最终产出物是和竞品一样的**线下产品包**（实体教学材料 + 形式载体），在线下由引导师运营。**线上运营（实时多人 SaaS / LMS / 在线数据采集）不在产品范围**。

---

## 仓库结构

```
docs/                        # 最终交付文档（HTML 预览版 + Word 可分享版）
  MRD.{html,docx}
  PRD.{html,docx}
  Task_Backlog.{html,docx}
  Competitor_Comparison.{html,docx}
  Capability_Assets.{html,docx}
  Core_Design_Assumption.{html,docx}   # 核心设计假设（锁定）
  Sample_Input_Checklist.{docx,pptx}   # 样本输入清单（可编辑）
  validation_rules.json      # 机器可读校验规则集
skill/avg-scenario-design-kit/   # 可调用能力资产 Skill（开发时一键加载）
  SKILL.md
  references/                # AVG 机制库 / 剧本杀 playbook / 案例库 / 检查清单 / 物料规格 / 校验规则
scripts/                     # 单源生成器（已纳入仓库，可重跑刷新 docs/）
  gen_template.py           # 产出 Sample_Input_Checklist.docx/.pptx
  gen_core_assumption.py    # 产出 Core_Design_Assumption.html/.docx
  render_assets.py          # 由 Skill 源重渲染 Capability_Assets.html/.docx
  inject_prd.py             # 向 PRD 注入核心假设加注段
```

> 注：`docs/` 内文档由 `scripts/` 下单源生成器产出，生成器已纳入本仓库，可重跑刷新；修改 Skill（`skill/avg-scenario-design-kit/`）后运行 `render_assets.py` 即可同步 Capability Assets。

---

## 如何查看

- **HTML 文档**：直接用浏览器打开 `docs/*.html`；或在本仓库开启 **GitHub Pages**（见下）获得在线链接。
- **Word 文档**：`docs/*.docx` 可直接下载用 Office / WPS 打开。
- **能力资产 Skill**：复制到 `~/.workbuddy/skills/avg-scenario-design-kit/` 即可在 WorkBuddy 中随时调用。

### 开启 GitHub Pages（获得可分享在线链接）

1. 仓库 → Settings → Pages
2. Source 选择 `Deploy from a branch`，Branch 选 `main`，目录选 `/ (root)` 或 `/docs`
3. 保存后访问 `https://<用户名>.github.io/<仓库名>/docs/MRD.html` 等

---

## 文档版本

| 文档 | 版本 | 关键变更 |
| --- | --- | --- |
| MRD / PRD | v2 | 定位改为「设计平台→线下包 / DIY」，线上运营划出范围 |
| Task Backlog / Competitor | v2 | E5/E6 重构为设计期预览 / 离线包导出；竞品重框为线下包生产者 |
| Capability Assets | v1 + ② ③ + 核心假设 | 接入自动校验规则 + 线下包实体物料规格 + 锁定「远场景/应用迁移桥」核心假设（VR-RE05） |
| Core_Design_Assumption | v1 | 核心设计假设（锁定）：远场景默认 + 强制应用迁移桥 |
| Sample_Input_Checklist | v1 | 可编辑 Word/PPT 模板（空白 + 情境领导示范） |
