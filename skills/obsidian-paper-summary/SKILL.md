---
name: obsidian-paper-summary
description: Summarize and organize academic papers into structured Obsidian notes for archiving and quick review. Use when the user provides a paper (PDF, text, or link) and wants a structured summary note. Generates Obsidian Flavored Markdown with properties, callouts, and organized sections. Requires the obsidian-markdown skill to be available.
---

# Paper Summary Skill

Summarize academic papers into structured Obsidian notes for archiving and quick browsing. This skill produces a standardized note format that captures the paper's core contributions, content breakdown, and critical evaluation.

## Prerequisites

**This skill requires the `obsidian-markdown` skill to be loaded in the current agent session.** Before generating any output, verify that the `obsidian-markdown` skill is available. If it is NOT available, **stop immediately** and inform the user:

> ⚠️ 错误：`obsidian-markdown` 技能未导入，无法生成符合 Obsidian 格式的笔记。请先导入 `obsidian-markdown` 技能后重试。

Do NOT proceed with note generation if the prerequisite skill is missing.

## Workflow

1. **Verify** the `obsidian-markdown` skill is available. If not, stop and report the error above.
2. **Read** the paper content provided by the user (PDF, pasted text, URL, etc.).
3. **Search Zotero**: 使用 `Zotero-MCP` 的 `search_library` 工具，以论文标题作为查询词在库中搜索。若返回结果中存在 `title` 与论文标题匹配的条目，取其 `key` 字段，将 frontmatter 中的 `{KEY}` 替换为该 key 值（如 `zotero://select/library/items/ABC123`）。若未找到匹配结果，则留空为 `zotero://select/library/items/`。
4. **Read PDF from Zotero**: 若步骤 3 匹配到条目，使用 `Zotero-MCP` 的 `get_item_details` 或 `get_content` 查找该条目下的附件 PDF。若找到 PDF 附件，读取其内容作为论文正文来源；若未找到 PDF，则回退使用用户提供的论文内容（步骤 2）。
5. **Determine the short title**:
   - 若步骤 3 匹配到 Zotero 条目，使用 `get_item_details` 获取该条目详情，检查其 `shortTitle` 字段是否存在且非空。
     - **存在**: 直接使用该 `shortTitle` 作为输出文件名。
     - **不存在**: 按下方 Short Title Rules 生成短标题。~~(由于Zotero-MCP暂时没有写入短标题的工具，暂时不写入，后续工具更新可恢复)然后使用 `Zotero-MCP` 将该短标题写入该条目的 `shortTitle` 字段。~~
   - 若步骤 3 未匹配到条目，则按 Short Title Rules 生成短标题作为输出文件名。
6. **Generate** the Obsidian note following the output template.
7. **Invoke** the `obsidian-markdown` skill conventions to ensure all syntax (frontmatter, callouts, wikilinks, etc.) is valid Obsidian Flavored Markdown.

## Short Title Rules

The note filename uses the paper's short title:

- If the paper explicitly provides a short title or acronym, use it directly.
- If not, generate one by taking the **first letter of each content word** in the title (skip articles, prepositions, conjunctions like "a", "an", "the", "of", "for", "and", "on", "in", "with", "to", "by").
- The result must be **ALL UPPERCASE**.
- Generally **no more than 6 characters**. If the acronym exceeds 6 characters, select the most representative words to stay within the limit.
- Special cases: well-known acronyms (e.g., BERT, GPT, YOLO, ResNet) should use their established form regardless of the letter rule.

Examples:
- "Attention Is All You Need" → `AIAYN`
- "Deep Residual Learning for Image Recognition" → `ResNet` (well-known)
- "Generative Adversarial Networks" → `GAN` (well-known)
- "A Survey of Large Language Models" → `SLLM`

## Output Template

The generated note MUST follow this exact structure:

````markdown
---
tags:
  - paper
Zotero: zotero://select/library/items/{KEY}
---

## Paper Full Title

> [!abstract] 核心贡献
> 用 2-4 句话概括论文的核心贡献和创新点。

## 论文内容梳理

### 研究背景与动机

简述研究问题、现有方法的不足、本文的出发点。

### 方法细节

详细描述论文提出的方法/模型/框架，包括：
- 整体架构
- 关键模块/组件
- 核心算法或公式（使用 LaTeX）

### 实验验证

- 数据集与评估指标
- 主要实验结果
- 消融实验（如有）

### 结论

论文的主要结论和未来工作方向。

## 论文评价

### 优势

- 列出论文的主要优点和创新之处

### 不足

- 列出论文的局限性和不足之处

### 改进空间

- 提出可能的改进方向和后续研究思路
````

## Formatting Rules

- **Frontmatter**: `tags` 至少包含 `paper`，可根据论文领域添加更多标签（如 `NLP`、`CV`、`RL` 等）。`Zotero` 字段通过 `Zotero-MCP` 的 `search_library` 工具自动查找：以论文标题搜索，若匹配到结果则取返回条目的 `key` 字段拼接为 `zotero://select/library/items/{key}`；若未匹配则留空。
- **二级标题 `##`**: 使用论文完整标题作为第一个二级标题。
- **Callout**: 使用 `> [!abstract] 核心贡献` 格式，内容精炼概括论文贡献。
- **内容梳理**: 使用三级标题 `###` 划分子章节，层次清晰。
- **数学公式**: 使用 LaTeX 语法（行内 `$...$`，块级 `$$...$$`）。
- **内部链接**: 如引用 vault 中已有笔记，使用 `[[wikilink]]` 格式。
- **语言**: 默认使用中文撰写笔记内容，论文专有名词保留英文。
- **禁止分割线**: 不要使用 `---`（水平分割线）在段落或章节之间插入分隔，保持内容紧凑连贯。

## Notes

- 如果用户提供的论文信息不完整（如只有标题没有正文），应提示用户补充内容。
- 根据论文实际内容灵活调整「论文内容梳理」下的子章节，不必死板套用模板（例如理论论文可能没有实验部分）。
- 保持笔记简洁，避免冗余，目标是让用户能在 2-3 分钟内快速回顾论文要点。
