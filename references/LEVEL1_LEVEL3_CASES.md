# Level 1 and Optional Level 3 — Acceptance Cases

Cases **33–53** extend the core 1–22 and web/chat 23–32 tests. Unless the instruction specifically says otherwise, only default Levels 1–2 are active. All examples are for *ordinary prose* except where protection is explicitly tested.

## Level 1 defaults

### 33. Safe fullwidth ASCII letters and digits

**Input:** `本次使用ＡＩ处理２０２６年的数据。`

**Expected (default standard spacing):** `本次使用 AI 处理 2026 年的数据。`

**Counterexample:** Do not use blanket NFKC on `①`, mathematical styled symbols, emoji sequences or source code.

### 34. Remove stray zero-width space but not meaningful joiners

**Input:** `文本清\u200b洗` (the `\u200b` notation represents a stray U+200B between 清 and 洗).

**Expected:** `文本清洗`.

**Counterexamples:** Preserve the emoji sequence `👩‍💻` (ZWJ) and meaningful ZWNJ/ZWJ in languages which use them. Do not strip all `U+200C/U+200D/U+FE0F` unconditionally.

### 35. BOM and mixed line endings

**Input:** A UTF-8 decoded plain-text file beginning with `U+FEFF` and containing CRLF/CR line endings in ordinary prose.

**Expected:** Remove the leading BOM and represent logical line endings consistently without merging genuine paragraphs.

### 36. Replacement character is not recoverable

**Input:** `研�结果表明...`

**Expected:** Keep the uncertain `�` within the text instead of guessing which character was lost. Do not report a successful repair.

### 37. Preserve intentional typographic and mathematical Unicode

**Input:** `H₂O 的分子式；① 第一项；👩‍💻；á`.

**Expected:** Do not globally transform the subscript, circled number, emoji variation/joiner sequences or combining accent. Outside protected elements, apply only the active ordinary Chinese/Latin spacing rule.

### 38. HTML entities only in rendered prose

**Input**

```text
测评&nbsp;&amp;&nbsp;反馈；代码示例：`&amp;`
```

**Expected:** Decode `&nbsp;` and `&amp;` in the rendered prose as ordinary whitespace/`&`, leaving the inline-code literal `&amp;` unchanged.

### 39. Page noise and substantial credits

**Input:** A scraped article with navigation `首页 | 下一页`, a central article and byline, `责任编辑：张某`, and repeated `返回顶部` controls.

**Expected:** Remove identifiable navigation and controls. Retain original article/byline; keep the editor credit when it is genuine publication metadata, even if it appears near the footer.

### 40. Repeated extraction vs. rhetorical repetition

**Input:** One paragraph appears twice due to an identifiable scraper duplicate; elsewhere a speaker writes `好好好好！！！`.

**Expected:** Remove only the scraper duplicate. Keep the expressive text exactly, apart from clearly accidental outside whitespace.

### 41. CJK-adjacent numerals and units in the default profile

**Input:** `下载20GB数据，携带10公斤器材，成功率15%。`

**Expected:** `下载 20 GB 数据，携带 10 公斤器材，成功率 15%。`

### 42. General no-add request also disables numerical unit padding

**User instruction:** `中西文之间不加空格。`

**Input:** `下载20GB数据，携带10公斤器材，成功率15%。`

**Expected:** `下载20GB数据，携带10公斤器材，成功率15%。` (no added spaces around Chinese/Latin, Chinese/digits, or number/Latin units).

### 43. Existing spaces under no-add are preserved

**User instruction:** `中西文之间不添加空格，已有的空格保留。`

**Input:** `下载 20 GB 数据，使用AI工具和 10 kg 配件。`

**Expected:** `下载 20 GB 数据，使用AI工具和 10 kg 配件。`

### 44. Compact no-space removes only optional padding

**User instruction:** `中西文之间以及数字和单位之间都不要留空格。`

**Input:** `下载 20 GB 数据，携带 10 公斤器材。The file is 20 GB.`

**Expected (ordinary prose context):** `下载20GB数据，携带10公斤器材。The file is 20GB.`

**Counterexample:** `This is a test`, `2 m + 3 m` inside an equation, `value = 10 GB` in code, and Markdown syntax must not lose required spaces.

### 45. Chinese/Latin-only override also disables Latin unit padding

**User instruction:** `中英文之间不添加空格，但中文和数字仍按标准排版。`

**Input:** `本项目使用AI处理20GB文件，共3批。`

**Expected:** `本项目使用AI处理 20GB文件，共 3 批。`

### 46. No Level 3 conversion in default mode

**User instruction:** `清洗这段文本。`

**Input:** `學術寫作在2026年10月26日開始。This resarch is about USA.`

**Expected:** `學術寫作在 2026年10月26日開始。This resarch is about USA.` (except for allowed mechanical cleanup, do not convert script, date, spellings, casing or terminology).

## Optional Level 3 (explicit opt-in only)

### 47. Simplified conversion alone

**User instruction:** `把这段中文转成简体，其他内容不要改。`

**Input:** `學術寫作；This resarch uses 20GB.`

**Expected:** `学术写作；This resarch uses 20GB.` The instruction “其他内容不要改” overrides ordinary optional formatting, so `20GB` and `resarch` remain untouched.

### 48. Requested date format only

**User instruction:** `把明确无歧义的日期统一成 YYYY-MM-DD，其余仅做基本清洗。`

**Input:** `开始于2026年10月26日；另一条记录为03/04/2026。`

**Expected:** `开始于 2026-10-26；另一条记录为 03/04/2026。` (ambiguous numeric date remains unchanged).

### 49. Explicit spelling correction, without collateral edits

**User instruction:** `纠正这句话的英文拼写错误。`

**Input:** `This resarch uses 20GB. The iPhone app is useful.`

**Expected:** `This research uses 20 GB. The iPhone app is useful.` Correct `resarch`, keep product case and unchanged factual content. Formatting remains subject to the default profile.

### 50. No automatic conversion of legitimate alternatives

**User instruction:** `清理空格和换行。`

**Input:** `Published: 28 Jul 2006. 版本2.0。USA / US / 美国。`

**Expected:** Preserve date, version notation and all three distinct source expressions; correct only unequivocal whitespace/layout defects and default Chinese-mixed spacing. Do not force one canonical date, abbreviate countries, or rewrite terms.

### 51. Case normalization is opt-in, protect official casing

**User instruction:** `只把普通英文句子的句首字母调整为大写，保留专有名词大小写。`

**Input:** `the iPhone app works with GitHub.`

**Expected:** `The iPhone app works with GitHub.` Do not mechanically change the brands to `Iphone` or `Github`.

### 52. Terminology stays heterogeneous unless explicitly standardized

**User instruction:** `整理文本中的无意义换行。`

**Input:** `本报告提到USA、US和美国，单位分别为km与千米。`

**Expected:** Keep the distinct source terms (`USA`, `US`, `美国`, `km`, `千米`) while applying ordinary spacing rules. Do not silently select one nationality/measurement expression.

### 53. Requested punctuation cleanup does not automatically remove emphasis

**User instruction:** `规范标点周围的空格。`

**Input:** `太好了 ！！！！！ 今天终于完成了。`

**Expected:** `太好了！！！！！今天终于完成了。` Retain the repeated exclamation marks because the user requested *spacing* cleanup, not content-level emphasis reduction.

## Required safeguards

- Level 1 is default; Level 3 is off unless a particular operation is explicitly requested.
- Do not show a normalization-mode selection prompt in the standard output, and do not suggest one after the cleaned text.
- “中西文不加空格” disables new padding **including between numeric values and Latin units**; no-add and no-space remain distinct.
- All transformations are context-sensitive. Protected syntactic spans, real paragraphs, names, note markers, mathematical expressions, references, and actual chat turns remain intact.
- Reapplying a fixed profile does not change its own output again.
