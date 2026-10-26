# Text Cleanup — Examples and Acceptance Cases

These examples specify the desired output **only when the context is sufficiently clear**. They are behavioral tests, not instructions to perform naive global replacements. Unless stated otherwise, the default standard profile adds Chinese↔Latin, Chinese↔number and ordinary number↔Latin-unit spacing where safe; general no-add/no-space requests cover all three.

## Core cleanup

### 1. Width-limited English lines

**Input**

```text
The representation of cultural identity
has changed in recent years.
```

**Expected**

```text
The representation of cultural identity has changed in recent years.
```

### 2. Width-limited Chinese lines

**Input**

```text
人工智能辅助写作正在改
变学生的学习方式。
```

**Expected**

```text
人工智能辅助写作正在改变学生的学习方式。
```

### 3. Broken English word

**Input:** `Several environ mental factors were considered.`

**Expected:** `Several environmental factors were considered.`

### 4. Line-end hyphenation

**Input**

```text
This is an inter-
national study.
```

**Expected:** `This is an international study.`

### 5. Real lexical hyphen must survive

**Input**

```text
The well-
known approach is used.
```

**Expected:** `The well-known approach is used.`

### 6. Missing spaces between English words

**Input:** `Thisstudyexamineslanguagelearning.`

**Expected:** `This study examines language learning.`

**Counterexample:** `The therapist arrived.` must **not** become `The the rapist arrived.`

### 7. Excess spaces in English

**Input:** `This    paper   compares   two methods.`

**Expected:** `This paper compares two methods.`

### 8. Unwanted spaces in continuous Chinese prose

**Input:** `本 文 讨 论 人 工 智 能 辅 助 写 作。`

**Expected:** `本文讨论人工智能辅助写作。`

### 9. Genuine paragraph boundaries

**Input**

```text
第一段讨论研究背景。

第二段介绍研究方法。
```

**Expected**: two distinct paragraphs, separated by one blank line. Do not merge them.

## Spacing profiles

### 10. Default typography

**Input:** `这个AI工具有3个功能，速度为20Gbps，提升15%。`

**Expected:** `这个 AI 工具有 3 个功能，速度为 20 Gbps，提升 15%。`

### 11. Explicitly do not add Chinese-adjacent spaces

**User instruction:** `中英文和中文与数字之间都不要添加空格。`

**Input:** `这个AI工具有3个功能，容量20GB，提升15%。`

**Expected:** `这个AI工具有3个功能，容量20GB，提升15%。`

### 12. Preserve existing spaces, without adding new ones

**User instruction:** `中英文之间不添加空格，已有的保留。`

**Input:** `我用 AI 写作，也用GPT，另有20GB和 10 GB 空间。`

**Expected:** `我用 AI 写作，也用GPT，另有20GB和 10 GB 空间。`

### 13. Compact mode actually removes optional boundary spaces

**User instruction:** `中英文、中文数字之间都不要留空格。`

**Input:** `这个 AI 工具有 3 个功能，使用 20 GB 空间。`

**Expected:** `这个AI工具有3个功能，使用20GB空间。`

### 14. Independent control

**User instruction:** `中英文之间不加空格，但中文与数字之间按照标准排版。`

**Input:** `使用AI处理20GB文件，共3项任务。`

**Expected:** `使用AI处理 20GB文件，共 3 项任务。`

## Protected content and conservative edits

### 15. Markdown and URL safety

**Input**

```markdown
使用[官网](https://example.com/a-b?q=x%20y)查看 `value = 10`。
```

**Expected**: preserve the link target and inline-code contents exactly; only normalize the surrounding Chinese prose where applicable. No alteration to `%20`, `a-b`, `q=x`, or the spaces in `value = 10`.

### 16. Japanese prose

**Input:** `日本語の文章をAIで翻訳する。`

**Expected**: preserve the Japanese source as written unless the user explicitly asks to reformat Japanese. Chinese copywriting spacing defaults are not Japanese typesetting rules.

### 17. Academic citations, punctuation, and references

**Input:** `Results were significant (Smith & Lee, 2024, p. 12).`

**Expected**: identical. Do not normalize the citation style or change the page number.

### 18. Real line breaks for list items

**Input**

```markdown
1. Research question
2. Methods
3. Findings
```

**Expected**: three distinct items, unchanged. Do not merge into one paragraph.

### 19. Ambiguous segmentation

**Input:** `therapist` or an unfamiliar coined expression, identifier, or product name.

**Expected**: preserve the token. Do not guess a segmentation to make it look more like natural language.

### 20. Typed unit vs. product code

**Input:** `20GB 存储；5G 网络；IPv6 地址。`

**Expected:** `20 GB 存储；5G 网络；IPv6 地址。` (only `20GB` is clearly quantity + unit; retain `5G` and `IPv6` as conventional terms).

### 21. Footnotes and fenced code

**Input**

````markdown
正文[^3]。

[^3]: 原文注释。

```python
print("hello  world")
```
````

**Expected**: retain reference `[^3]`, its definition, and the literal double space inside the Python string. Do not renumber or change code.

### 22. Do not silently proofread

**Input:** `This resarch considers 3 samples.`

**Expected:** `This resarch considers 3 samples.` (the apparent spelling error `resarch` is outside mechanical spacing cleanup; preserve it).

## Final acceptance criteria

- Every statement in **retained primary content and actual conversation turns** is present once, in the original sequence. For full-page website copies and AI chat exports, identifiable UI/navigation/operational noise is removed under the source-aware rules.
- The user's per-boundary spacing overrides are respected, including the coordinated numeric-unit setting when mixed-language spacing is disabled.
- Paragraph boundaries, notes, tables, Markdown syntax, and protected spans are intact.
- Running the same cleanup profile again makes no substantive changes.
- Ambiguity is preserved rather than resolved by an unsupported guess.

## Web and AI-conversation cases

See [`WEB_CHAT_CASES.md`](WEB_CHAT_CASES.md) for cases 23–32, covering whole academic webpage copying, DeepSeek process text, multi-turn Claude histories, page headers/footers, article bibliographies versus UI tabs, role quotations, HTML entities, and user-requested unfiltered copying.

## Level 1 and optional-normalization cases

See [`LEVEL1_LEVEL3_CASES.md`](LEVEL1_LEVEL3_CASES.md) for cases 33–53, including Unicode/encoding cleanup, fullwidth Latin/digits, boilerplate and duplicate handling, no-add/unit spacing, and explicitly activated Level 3 operations.
