# 自动校验规则（Validation Gates，机器可读）

> 用途：把《设计期检查清单》落成**可机读的自动校验规则**，作为 PRD 第 11 节 Validation 门禁的执行标准。

> 由 **Validator Agent（Pipeline 的质检环节）** 在每阶段后 / 导出前消费：逐条对 AVG Project JSON 求值，
> `BLOCKER` 未过 → 阻断发布并打回重生成；`WARN` → 提示设计师；`INFO` → 人工复核。

> 机器可读版本见 `validation_rules.json`（与本文同源于 `gen_validation.py`）。


---

## 一、规则总表（按阶段）

### 挑战

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-CH01` | `stages.challenge / coursePackage.objectives / learningGameplayMap.mappings` | len(stages.challenge)>=1 且 每个 objective 在 mappings 中被 mechanic∈{challenge,exploration,decision} 覆盖 | **BLOCKER** | 存在未被真实挑战覆盖的学习目标 | 自动 |
| `VR-CH02` | `stages.challenge[].clueSource` | 每个 challenge 含 clueSource 字段且指向 exploration 线索 | **WARN** | 挑战缺少可推理的线索来源（沦为直接讲授） | 自动 |
| `VR-CH03` | `stages.challenge[].difficulty` | 每个 challenge 标注 difficulty∈{low,mid,high} 且非全部 high | **WARN** | 挑战难度未标注或整体过高（无渐进） | 自动 |

### 探索

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-EX01` | `kit.cards.statusVars` | len(kit.cards.statusVars) <= 7 | **BLOCKER** | 全局状态变量(flag)超过 7 个，存在组合爆炸风险 | 自动 |
| `VR-EX02` | `multiplayerDesign.groupExploration / infoDistributionRules` | len(groupExploration)>=2 且各组 infoDistributionRules 互不相同 | **WARN** | 缺少多视角/信息差（各组探索内容雷同） | 自动 |
| `VR-EX03` | `multiplayerDesign.finalConvergenceNode` | finalConvergenceNode 非空 | **WARN** | 缺少自然汇合点（瓶颈），分支可能组合爆炸 | 自动 |
| `VR-EX04` | `stages.npcs[]` | 每个 npc 含 memory/motivation/agency 三字段且非空 | **WARN** | NPC 缺记忆/动机/能动性三支柱（非活体） | 自动 |

### 抉择

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-DE01` | `stages.decisions[].options` | 每个 decision 的 options 长度 >= 2 | **BLOCKER** | 存在仅 1 个选项的『假抉择』 | 自动 |
| `VR-DE02` | `stages.decisions[].options[].tradeoff` | 每个 option.tradeoff 非空 且 非『明显正确』 | **BLOCKER** | 选项缺少真实 trade-off（疑似对错题） | 自动 |
| `VR-DE03` | `stages.decisions[].options[].leadsTo / consequence` | 同一 decision 下不同 option 的 consequence/leadsTo 互不相同 | **BLOCKER** | 不同选择导致相同后果（illusion of choice） | 自动 |
| `VR-DE04` | `stages.decisions（跨节点间距）` | 决策平均间隔 ∈ [3,8] 节点 且 有变化（非恒定密度） | **WARN** | 抉择密度恒定/过密，缺少呼吸 | 自动 |
| `VR-DE05` | `（人工）` | 抉择从戏剧时刻自然涌现，未打断叙事 | **INFO** | 需人工确认抉择并非生硬插入 | 人工 |

### 后果

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-CO01` | `stages.decisions[].options[].consequence` | 每个 option.consequence 至少改变 1 个 statusVar 或 ability | **BLOCKER** | 选择未产生可见能力/关系变化 | 自动 |
| `VR-CO02` | `stages.consequences[].threshold` | 存在阈值门槛：某 statusVar/ability 达阈值解锁场景/结局/能力 | **WARN** | 缺少阈值开门机制（计量无意义） | 自动 |
| `VR-CO03` | `stages.endings / reflection（非最优路径）` | 非最高 ability 的路径也对应 unique content（结局/复盘） | **WARN** | 存在唯一完美解，非最优路径无内容 | 自动 |

### 成长

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-GR01` | `stages.growth.abilityRadar` | stages.growth 含 abilityRadar 定义（用于复盘量化） | **WARN** | 成长缺少可量化能力雷达 | 自动 |

### 结局

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-EN01` | `stages.endings` | len(stages.endings) >= 2 | **BLOCKER** | 仅单一『通关』结局，缺少平行价值结局 | 自动 |
| `VR-EN02` | `（人工）` | 结局间无『真/假』排行，每个结局配得上其路径 | **WARN** | 疑似存在唯一真结局，弱化其他路径 | 人工 |
| `VR-EN03` | `stages.endings[].abilityRadar` | 每个 ending 含 abilityRadar 且彼此不同 | **WARN** | 不同结局未对应不同能力雷达（复盘不可区分） | 自动 |
| `VR-EN04` | `（人工）` | 每个 ending 提供闭环（呼应开头目标/秘密） | **INFO** | 需人工确认结局闭环 | 人工 |

### 复盘

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-RE01` | `stages.reflection[].aarSteps` | aarSteps 四步齐全：goal/event/gap/action | **BLOCKER** | 复盘缺少 AAR 结构化四步 | 自动 |
| `VR-RE04` | `kit.takeawayTemplates` | kit 含 takeawayTemplates（abilityRadar / actionPlan / consensus 至少其一） | **BLOCKER** | 离线包未内嵌可带走的成果模板 | 自动 |
| `VR-RE02` | `stages.reflection[].principles` | 含心理安全原则标记：askBeforeTell / evidenceAnchored / nameBehaviorNotPerson | **WARN** | 复盘未贯彻心理安全原则 | 自动 |
| `VR-RE03` | `stages.reflection[].commitment` | 以具体、有时限的承诺收尾（commitment 字段非空） | **WARN** | 复盘未落到具体行动承诺 | 自动 |
| `VR-RE05` | `stages.reflection.realScenarioApplication / kit.takeawayTemplates` | reflection 含 realScenarioApplication（真实挑战+选用方法+有时限第一步），且 kit.takeawayTemplates 含 'realScenarioApplication' | **BLOCKER** | 缺少应用迁移桥，所学无法落地到真实工作 | 自动 |

### 离线包

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-OFF01` | `kit.script` | kit.script 非空 | **BLOCKER** | 脚本/剧本为空 | 自动 |
| `VR-OFF02` | `kit.cards` | kit.cards 含 infoCards / npcCards / decisionCards / taskCards / statusVars 全部非空 | **BLOCKER** | 离线包卡片组件不全 | 自动 |
| `VR-OFF03` | `kit.roleTracks` | 5 <= len(kit.roleTracks) <= 6 | **BLOCKER** | 角色分轨非 5–6 组 | 自动 |
| `VR-OFF04` | `kit.facilitatorGuide` | kit.facilitatorGuide 非空（DM 手册） | **BLOCKER** | 缺引导师(DM)手册 | 自动 |
| `VR-OFF05` | `kit.reflection / materialList` | kit 含复盘物料（reflection 字段或 materialList 含复盘手册/知识卡片） | **BLOCKER** | 缺复盘手册/知识卡片 | 自动 |
| `VR-OFF06` | `multiplayerDesign.facilitatorPanel / syncNodes / finalConvergenceNode` | 三者均存在（保证 1 讲师带 6 组可行） | **BLOCKER** | 多人运行设计缺失，1讲师无法带6组 | 自动 |
| `VR-OFF08` | `kit.export.format` | format 含 'pdf'（至少印刷就绪） | **BLOCKER** | 未产出可印刷格式 | 自动 |
| `VR-OFF07` | `kit.export.printSpec / materialList` | 两者非空（套用 offline_package_spec.md 规格） | **WARN** | 缺少印刷规格/物料清单 | 自动 |

### 通用

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-GEN01` | `决策图可达性` | 每个 option.leadsTo 指向存在的节点；从起点可达所有 ending；无死锁分支 | **BLOCKER** | 存在死锁/不可达分支 | 自动 |
| `VR-GEN03` | `exploration/decision/consequence/reflection` | 四者均非空（探索→决策→后果→复盘 完整闭环） | **BLOCKER** | 缺少完整学习闭环（纯观看） | 自动 |
| `VR-GEN04` | `learningGameplayMap.mappings + coursePackage.objectives` | 每个 objective 在 mappings 中 coverage=='full' | **BLOCKER** | 学习目标未被游戏行为充分覆盖 | 自动 |
| `VR-GEN05` | `multiplayerDesign` | facilitatorPanel + syncNodes + groupExploration/infoDistributionRules 完整（1讲师可运行6组） | **BLOCKER** | 一个讲师无法同时运行六组 | 自动 |
| `VR-GEN02` | `（启发式）` | 不存在唯一『最优攻略』直接破解（需权衡的多条路径存在） | **WARN** | 疑似唯一最优解直接通关 | 自动 |

### 分层（3×2 矩阵）

| 规则ID | 检查对象 / JSON 路径 | 自动条件（伪代码） | 严重度 | 失败信息 | 执行 |
| --- | --- | --- | --- | --- | --- |
| `VR-TIER01` | `kit.totalDuration / stages.avgDuration` | avgDuration == totalDuration * 0.5（AVG 探索体验严格占 50%） | **BLOCKER** | AVG 时长未占总时长 50%，违反核心时间约束 | 自动 |
| `VR-TIER02` | `stages.import / stages.reflection / stages.application / stages.idp` | 非 AVG 四活动齐全：导入/复盘/应用/IDP 均存在且非空 | **BLOCKER** | 非 AVG 50% 的四项活动缺失，培训无法落地 | 自动 |
| `VR-TIER03` | `kit.extraction / kit.competencyModel` | Track=C2 时必含 extraction 与 competencyModel；C1 可豁免 | **WARN** | C2 缺少萃取阶段或能力模型 | 自动 |
| `VR-TIER04` | `stages.decisions` | 决策节点最小数：L1≥1 / L2≥3 / L3≥5（按 kit.tier 校验） | **WARN** | 该层决策节点数低于复杂度下限 | 自动 |
| `VR-TIER05` | `kit.priceAnchor / kit.effortAnchor` | 与 tier×track 建议区间匹配（L1·C1 0.5-1d/¥3-8K … L3·C2 10-20d/¥80-200K+） | **INFO** | 定价/工时锚偏离分层建议区间（人工复核） | 人工 |


---

## 二、扩展节点子 Schema（规则引用的字段，供工程实现一致）

> AVG Project JSON 主结构见 PRD 第 5 节；下方为规则直接引用的节点子结构，Validator 据此取值。

```json
{
  "DecisionNode": {
    "id": "",
    "prompt": "",
    "context": "",
    "options": [
      {
        "id": "",
        "label": "",
        "tradeoff": "",
        "leadsTo": "",
        "consequence": ""
      }
    ]
  },
  "EndingNode": {
    "id": "",
    "type": "best|compromise|collapse",
    "abilityRadar": {},
    "closure": ""
  },
  "ReflectionNode": {
    "aarSteps": {
      "goal": "",
      "event": "",
      "gap": "",
      "action": ""
    },
    "principles": [
      "askBeforeTell",
      "evidenceAnchored",
      "nameBehaviorNotPerson"
    ],
    "commitment": "",
    "takeawayTemplateRef": ""
  },
  "NPCEntity": {
    "name": "",
    "memory": "",
    "motivation": "",
    "agency": "",
    "attitude": ""
  },
  "TakeawayTemplate": {
    "kind": "abilityRadar|actionPlan|consensus",
    "fields": []
  }
}
```


---

## 三、校验执行说明（给 Validator Agent）

1. **加载**：读取当前 AVG Project JSON（统一数据模型）。
2. **求值**：对每条 `auto==true` 的规则，按 `path` 取字段、按 `check` 判定；`auto==false` 的规则输出人工复核项。
3. **分级**：收集 `BLOCKER` 失败 → 阻断 `meta.status` 进入 `published`，返回定位（ruleId + JSON path）给生成 Agent 重生成；`WARN` → 进 Quality Report；`INFO` → 人工确认清单。
4. **产物**：输出 `kit.qualityReport`（AVG Quality Check + Prototype Test Report 合并），随离线包交付。
5. **门禁闭环**：导出离线包（E6）前必须 `VR-OFF*` 全过；发布前必须全部 `BLOCKER` 全过。


> 调用方式：开发某阶段时读本文件对应行，或让 Validator Agent 直接加载 `validation_rules.json`。
