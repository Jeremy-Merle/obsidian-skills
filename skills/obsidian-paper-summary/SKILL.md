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
3. **Search Zotero**: Use the `Zotero-MCP` `search_library` tool to search the library by the paper title. If an item's `title` matches the paper title, take its `key` and replace `{KEY}` in the frontmatter with that key (for example, `zotero://select/library/items/ABC123`). If there is no match, leave the URL as `zotero://select/library/items/`.
4. **Read the original PDF from Zotero**:
   - If step 3 finds a matching item, use `get_item_details` to enumerate its child PDF attachments and inspect their attachment titles and filenames. Normalize both values by removing a trailing `.pdf` extension and comparing names case-insensitively.
   - Exclude every attachment whose normalized title or filename ends with `-compare`; these are translated Chinese-English comparison versions.
   - Select an attachment whose normalized title or filename is exactly `PDF` or `Full Text PDF`. Call `get_content` with that attachment's `attachmentKey` and use its content as the primary source for the paper's main text.
   - If no eligible original attachment exists, do not read a `*-compare` PDF. Fall back to the paper content provided by the user in step 2; if the user did not provide usable content, ask them to supply the original paper.
5. **Determine the short title**:
   - If step 3 finds a matching Zotero item, use `get_item_details` to inspect its `shortTitle` field.
     - **If present**: Use that `shortTitle` directly as the output filename.
     - **If absent**: Generate a short title using the Short Title Rules below. ~~(Zotero-MCP currently has no tool for writing a short title, so do not write it back for now; restore this behavior when support is added.)~~
   - If step 3 finds no matching item, generate a short title using the Short Title Rules below and use it as the output filename.
6. **Generate** the Obsidian note using the output template, including the `文献阅读: true` property.
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
Zotero: zotero://select/library/items/{KEY}
文献阅读: true
---

## Paper Full Title

> [!abstract] 核心贡献
> - 简洁列出核心贡献与创新点
> - 简洁列出其他关键贡献

> [!question] 不足
> - 简洁列出主要局限性
> - 简洁列出其他关键不足

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

- **Frontmatter**: 每篇生成笔记都必须包含 `文献阅读: true` 属性。`Zotero` 字段通过 `Zotero-MCP` 的 `search_library` 工具自动查找：以论文标题搜索，若匹配到结果则取返回条目的 `key` 字段拼接为 `zotero://select/library/items/{key}`；若未匹配则留空。
- **二级标题 `##`**: 使用论文完整标题作为第一个二级标题。
- **Callout（核心贡献与不足）**: 使用 `> [!abstract] 核心贡献` 格式，以简洁的项目列表概括论文贡献与创新点；随后另起一个 `> [!question] 不足` Callout，以简洁的项目列表概括论文的主要局限性。
- **内容梳理**: 使用三级标题 `###` 划分子章节，层次清晰。
- **数学公式**: 使用 LaTeX 语法（行内 `$...$`，块级 `$$...$$`）。
- **内部链接**: 如引用 vault 中已有笔记，使用 `[[wikilink]]` 格式。
- **语言**: 默认使用中文撰写笔记内容，论文专有名词保留英文。
- **禁止分割线**: 不要使用 `---`（水平分割线）在段落或章节之间插入分隔，保持内容紧凑连贯。

## Notes

- 如果用户提供的论文信息不完整（如只有标题没有正文），应提示用户补充内容。
- 根据论文实际内容灵活调整「论文内容梳理」下的子章节，不必死板套用模板（例如理论论文可能没有实验部分）。
- 保持笔记简洁，避免冗余，目标是让用户能在 2-3 分钟内快速回顾论文要点。
