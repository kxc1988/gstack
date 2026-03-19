# gstack 翻译质量回归检查计划

## 检查目标

对 gstack 项目的所有中文翻译文件进行全面的质量回归检查，确保翻译的准确性、一致性、流畅性和完整性。

## 检查范围

### 1. 核心文档（P0 优先级）

- [ ] README.md → README.zh-CN.md
- [ ] ARCHITECTURE.md → ARCHITECTURE.zh-CN.md
- [ ] BROWSER.md → BROWSER.zh-CN.md
- [ ] docs/skills.md → docs/skills.zh-CN.md

### 2. 技能文件（P1 优先级）

- [ ] browse/SKILL.md → browse/SKILL.zh-CN.md
- [ ] review/SKILL.md → review/SKILL.zh-CN.md
- [ ] qa/SKILL.md → qa/SKILL.zh-CN.md
- [ ] ship/SKILL.md → ship/SKILL.zh-CN.md
- [ ] office-hours/SKILL.md → office-hours/SKILL.zh-CN.md
- [ ] debug/SKILL.md → debug/SKILL.zh-CN.md
- [ ] design-review/SKILL.md → design-review/SKILL.zh-CN.md
- [ ] design-consultation/SKILL.md → design-consultation/SKILL.zh-CN.md
- [ ] retro/SKILL.md → retro/SKILL.zh-CN.md
- [ ] document-release/SKILL.md → document-release/SKILL.zh-CN.md
- [ ] gstack-upgrade/SKILL.md → gstack-upgrade/SKILL.zh-CN.md
- [ ] setup-browser-cookies/SKILL.md → setup-browser-cookies/SKILL.zh-CN.md

### 3. 计划审查技能（P1 优先级）

- [ ] plan-ceo-review/SKILL.md → plan-ceo-review/SKILL.zh-CN.md
- [ ] plan-eng-review/SKILL.md → plan-eng-review/SKILL.zh-CN.md
- [ ] plan-design-review/SKILL.md → plan-design-review/SKILL.zh-CN.md

### 4. QA 相关文件（P2 优先级）

- [ ] qa/references/issue-taxonomy.md → qa/references/issue-taxonomy.zh-CN.md
- [ ] qa/templates/qa-report-template.md → qa/templates/qa-report-template.zh-CN.md

### 5. 审查相关文件（P2 优先级）

- [ ] review/checklist.md → review/checklist.zh-CN.md
- [ ] review/design-checklist.md → review/design-checklist.zh-CN.md
- [ ] review/greptile-triage.md → review/greptile-triage.zh-CN.md
- [ ] review/TODOS-format.md → review/TODOS-format.zh-CN.md

### 6. 其他文档（P2 优先级）

- [ ] CONTRIBUTING.md → CONTRIBUTING.zh-CN.md
- [ ] CHANGELOG.md → CHANGELOG.zh-CN.md
- [ ] CLAUDE.md → CLAUDE.zh-CN.md
- [ ] TODOS.md → TODOS.zh-CN.md

## 检查维度

### 1. 术语一致性检查

**检查项：**
- 技术术语是否与术语表一致
- 同一术语在不同文件中的翻译是否统一
- 专有名词（人名、公司名、产品名）是否保持正确

**重点术语：**
- gstack, Claude Code, skill, browser, daemon, Playwright, Chromium
- office-hours, plan-ceo-review, plan-eng-review, plan-design-review
- QA, QA-only, ship, review, debug, design-review, retro
- snapshot, ref, locator, accessibility tree, CSP, SPA
- cookie, session, workflow, bearer token, circular buffer

### 2. 准确性检查

**检查项：**
- 技术概念翻译是否准确
- 代码、命令、URL 是否保持原样
- 数字、日期、版本信息是否正确
- 链接是否保持正确且可访问

**常见问题：**
- 技术术语误译
- 代码块中的内容被错误翻译
- 命令参数被翻译
- 链接文本被翻译但 URL 被修改

### 3. 流畅性检查

**检查项：**
- 句子是否通顺自然
- 是否符合中文表达习惯
- 是否存在翻译腔（过度直译）
- 标点符号使用是否正确

**常见问题：**
- 过长的定语从句
- 被动语态滥用
- 代词过多导致指代不清
- 连接词使用不当

### 4. 完整性检查

**检查项：**
- 是否有遗漏的段落或章节
- 表格内容是否完整翻译
- 列表项是否完整
- 注释和说明是否翻译

**常见问题：**
- 跳过难翻译的段落
- 表格部分单元格未翻译
- 长列表部分项目遗漏
- 图片说明文字未翻译

### 5. 格式一致性检查

**检查项：**
- 标题层级是否与原文一致
- 代码块格式是否正确
- 表格格式是否保持
- 列表格式是否正确
- 粗体、斜体等强调格式是否保留

**常见问题：**
- 标题级别错误
- 代码块语言标识丢失
- 表格列对齐错误
- 列表缩进不一致

### 6. 上下文一致性检查

**检查项：**
- 跨文件引用是否正确
- 技能名称、命令名称是否统一
- 前后文术语是否一致
- 与术语表的匹配度

## 检查方法

### 1. 自动化检查

**工具使用：**
- 使用 diff 工具对比原文和译文的文件结构
- 检查代码块数量是否一致
- 检查标题数量是否一致
- 检查链接数量是否一致

**检查脚本：**
```bash
# 统计原文和译文的行数、代码块数、标题数
# 对比关键数据是否一致
```

### 2. 人工抽查

**抽样策略：**
- 每个文件至少抽查 3 个关键章节
- 重点检查开头、中间、结尾部分
- 检查表格、列表等特殊格式
- 检查代码示例和命令

**检查比例：**
- P0 文件：100% 检查
- P1 文件：至少 50% 抽查
- P2 文件：至少 30% 抽查

### 3. 对比检查

**并排对比：**
- 打开原文和译文进行对比
- 逐段检查翻译质量
- 标记可疑的翻译
- 记录发现的问题

## 问题分类

### 严重问题（Critical）

- 技术术语错误导致误解
- 关键信息遗漏
- 代码、命令被错误翻译
- 链接失效或错误

### 中等问题（Major）

- 翻译不准确但不影响理解
- 术语不一致
- 格式错误
- 语句不通顺

### 轻微问题（Minor）

- 标点符号使用不当
- 个别词汇可以更优化
- 风格不统一但不影响理解

## 检查流程

### 阶段 1：准备（1 小时）

1. 复习术语表，熟悉关键术语
2. 准备检查工具和脚本
3. 创建问题记录模板
4. 制定检查优先级

### 阶段 2：自动化检查（1 小时）

1. 运行自动化检查脚本
2. 统计文件结构差异
3. 识别明显的格式问题
4. 生成初步检查报告

### 阶段 3：人工检查（6-8 小时）

1. 按优先级顺序检查文件
2. 记录发现的问题
3. 标注问题类型和严重程度
4. 提出修改建议

### 阶段 4：问题汇总（1 小时）

1. 整理所有发现的问题
2. 按文件和问题类型分类
3. 生成详细的检查报告
4. 提出修复优先级建议

### 阶段 5：修复验证（2-3 小时）

1. 根据检查报告修复问题
2. 验证修复后的质量
3. 进行最终抽查
4. 生成最终质量报告

## 问题记录模板

```markdown
## [文件名]

### 问题 1
- **位置**: 第 X 行
- **类型**: [术语错误 | 遗漏 | 格式 | 流畅性 | 其他]
- **严重程度**: [Critical | Major | Minor]
- **原文**: [英文原文]
- **译文**: [当前翻译]
- **建议**: [修改建议]
- **说明**: [问题说明]

### 问题 2
...
```

## 质量标准

### 通过率要求

- **P0 文件**: 95% 以上无问题
- **P1 文件**: 90% 以上无问题
- **P2 文件**: 85% 以上无问题

### 零容忍问题

- 技术术语严重错误
- 代码、命令被错误翻译
- 关键信息遗漏
- 链接失效

## 交付物

1. **检查报告**: 详细的问题清单和分类统计
2. **修复建议**: 针对每个问题的修改建议
3. **质量评分**: 每个文件的质量评分和总体评价
4. **术语表更新**: 发现的新术语或需要调整的术语
5. **最佳实践**: 翻译过程中的最佳实践总结

## 时间安排

| 阶段 | 时间 | 产出 |
|------|------|------|
| 准备 | 1 小时 | 检查模板、工具准备 |
| 自动化检查 | 1 小时 | 初步检查报告 |
| 人工检查 | 6-8 小时 | 详细问题清单 |
| 问题汇总 | 1 小时 | 完整检查报告 |
| 修复验证 | 2-3 小时 | 最终质量报告 |
| **总计** | **11-14 小时** | **完整的质量回归检查** |

## 风险评估

### 高风险

- 翻译质量问题影响用户体验
- 技术错误导致用户误操作
- 术语不一致造成混淆

### 缓解措施

- 严格执行检查标准
- 优先修复严重问题
- 持续更新术语表
- 建立翻译质量检查清单

## 后续改进

1. 建立翻译审查流程
2. 定期更新术语表
3. 创建翻译风格指南
4. 建立翻译质量监控机制
5. 培训翻译人员
