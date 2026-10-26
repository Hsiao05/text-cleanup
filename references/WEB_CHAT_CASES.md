# Webpage and AI Chat Cleanup — Acceptance Cases

These are **behavioral tests** for the user's three representative scenarios. Source snippets are intentionally shortened; the omitted portions of a real primary article or a real AI answer must still be retained during an actual cleaning run. For retained primary content, never rewrite the prose or replace it with a summary.

## 23. Example 1 — Entire academic article webpage

**Input** (shortened from a journal-site full-page copy):

```text
Skip to Main Content
Taylor and Francis Online homepage
Journals&#x20;
Search&#x20;
Login | Register
Add to Cart
Home All Journals TFO Collections
Publication Cover
TFO Collections
Volume 0, 2026 - Issue : The Contributions of Paul Black
Journal homepage
144,958
Views
0
CrossRef citations to date
179
Altmetric
Original Articles
Assessment and Classroom Learning
Paul Black &Dylan Wiliam
Pages 7-74 | Published online: 28 Jul 2006
Cite this article
References Citations Metrics Reprints & Permissions Read this article
ABSTRACT
This article is a review of the literature on classroom formative assessment. Several studies show firm evidence that innovations designed to strengthen the frequent feedback that students receive about their learning yield substantial learning gains.
Previous article
View issue table of contents
Next article
Related research
People also read
Formative assessment: a critical review
Randy Elliot Bennett
Browse journals by subject
Back to top
Information for
Authors
Librarians
Copyright © 2026 Informa UK Limited Privacy policy Cookies
```

**Expected cleaned Markdown**:

```markdown
# Assessment and Classroom Learning

Paul Black & Dylan Wiliam

Pages 7-74 | Published online: 28 Jul 2006

## ABSTRACT

This article is a review of the literature on classroom formative assessment. Several studies show firm evidence that innovations designed to strengthen the frequent feedback that students receive about their learning yield substantial learning gains.
```

**Assertions:**

- Keep the title, authors, unaltered article-level metadata, and the full *pasted* abstract/body.
- Do not add a journal name/volume/issue to the article unless its association is clear. Collection navigation is not authoritative article metadata.
- Drop shopping/login/search elements, sidebar, popularity counters, promotional citations, unrelated `People also read` entries, subject directories and legal/footer text.
- `References` as an action/tab is removable. **The article's real bibliography section, if present, must be preserved.**
- Do not infer that the abstract is the article's complete body.

## 24. Example 2 — DeepSeek multi-turn copy with sidebar and visible AI process

**Input** (representative excerpt):

```text
DeepSeek
Retry clarification
英语系毕业论文格式要求
提取帖子文字
npm安装成功确认

我们需要回答中文。用户给了一段与Grok对话，要求“总结我的需求”。需要提炼核心目标。
你的需求可以总结为：你想用 Claude Code 整理 X 收藏夹，并用 Obsidian 管理。

我担心批量处理 X 的收藏（几百条）时会出现三个问题，想请你先给出方案：
1. 一次性全扒工作量大；
2. 逐条发送会干扰上下文；
3. 每条新开窗口会浪费 token。
最终目标是按主题归并成几份文档。

用户需要解决三个痛点：批量处理工作量大、单窗口上下文干扰、新开窗口重复读取 token。需要先说明假设。
搜索到 24 个网页
浏览 1 个页面

以下方案基于一个核心判断：抓取和处理应该解耦。
先说明我的假设：采用浏览器导出 JSON，每批 8–12 条。
```

**Expected cleaned Markdown** (substantive visible process retained):

```markdown
**AI 过程记录（原文可见，非最终回答）：**

> 我们需要回答中文。用户给了一段与Grok对话，要求“总结我的需求”。需要提炼核心目标。

**AI：**

你的需求可以总结为：你想用 Claude Code 整理 X 收藏夹，并用 Obsidian 管理。

**用户：**

我担心批量处理 X 的收藏（几百条）时会出现三个问题，想请你先给出方案：

1. 一次性全扒工作量大；
2. 逐条发送会干扰上下文；
3. 每条新开窗口会浪费 token。

最终目标是按主题归并成几份文档。

**AI 过程记录（原文可见，非最终回答）：**

> 用户需要解决三个痛点：批量处理工作量大、单窗口上下文干扰、新开窗口重复读取 token。需要先说明假设。

**AI：**

以下方案基于一个核心判断：抓取和处理应该解耦。

先说明我的假设：采用浏览器导出 JSON，每批 8–12 条。
```

**Assertions:**

- Preserve *two separate AI final answers* and the one clearly recovered user prompt, in their original order. No user prompt may be fabricated for the initial AI summary.
- Sidebars/history titles are neither prompts nor source content; remove them.
- Source-visible `我们需要…` / `用户需要…` planning text can be retained separately with a process label; it cannot be called a user turn or final answer.
- Search/browser counters are transient UI notices, so remove them by default.
- Preserve the `1, 2, 3` list structure and any real code, tables, links or qualifications from the complete answer.
- For `remove-all` process mode, omit both AI process blocks **without altering the other turns**.

## 25. Example 2 — AI final-answer wording versus thinking markers

**Input**:

```text
用户：请解释 Chain-of-Thought 是什么。
AI：Chain-of-Thought 通常指一种推理表示方式。本文还会说明为什么不应依赖它作事实来源。
```

**Expected:** Both are ordinary turns. **Do not** classify the AI answer as process text just because it discusses a reasoning-related term.

**Counterexample:** A standalone line `Planning search queries for this topic` between a user turn and the AI's final answer is process/status text, **not** the final answer.

## 26. Example 3 — Claude copy beginning with an orphan AI answer

**Input** (condensed, preserving the turn order of the user's example):

```text
Claude
设计手机自动上传与自动打标的处理流程
明白了。既然没有常开的电脑，又每天都会新增很多图，方案的重点就变成两件事：找一个 24 小时在线的存储，以及把“存图”这个动作变成自动上传。
先解决“常开设备”：两条路
买一台入门 NAS（更推荐）
NAS 本来就是为 7×24 小时开机设计的。

可是现在硬盘什么的也贵，有点超预算了

Comparing free cloud server options against account risks
预算有限的话，其实不一定要买 NAS 和大硬盘。
方案一：几乎零成本，用免费云服务器
甲骨文云（Oracle Cloud）有一个“永久免费”套餐。

那如果我先不在云端部署，只在Windows电脑上把图片打标签归类呢

Comparing local tools like szurubooru and digiKam for portability
完全可以，而且这是个很稳妥的起步方式。
路线一：digiKam，标签直接写进图片文件里（最推荐）

Claude is AI and can make mistakes. Please double-check responses.
```

**Expected cleaned Markdown**:

```markdown
**AI：**

明白了。既然没有常开的电脑，又每天都会新增很多图，方案的重点就变成两件事：找一个 24 小时在线的存储，以及把“存图”这个动作变成自动上传。

先解决“常开设备”：两条路

买一台入门 NAS（更推荐）

NAS 本来就是为 7×24 小时开机设计的。

**用户：**

可是现在硬盘什么的也贵，有点超预算了

**AI：**

预算有限的话，其实不一定要买 NAS 和大硬盘。

方案一：几乎零成本，用免费云服务器

甲骨文云（Oracle Cloud）有一个“永久免费”套餐。

**用户：**

那如果我先不在云端部署，只在 Windows 电脑上把图片打标签归类呢

**AI：**

完全可以，而且这是个很稳妥的起步方式。

路线一：digiKam，标签直接写进图片文件里（最推荐）
```

**Assertions:**

- Preserve the **AI → 用户 → AI → 用户 → AI** sequence. Do not invent the first user's missing question from the sidebar title.
- Remove status-headlines beginning `Comparing …` when they are mere operational summaries, and remove the generic Claude footer.
- The full actual answers (the long NAS, cloud, digiKam explanations) **must remain complete in a real run**. This condensed example does not license omission.
- The spacing profile still applies within recovered turns (`Windows电脑` → `Windows 电脑` in default mode).

## 27. Page header/footer versus meaningful notes

**Input**:

```text
Journal of Language Research
— 12 —
The sample included 43 learners.
[^7]: The questionnaire was adapted from the original study.
Journal of Language Research
— 13 —
The results are shown in Table 2.
```

**Expected:** Remove both repeated journal running headers and the standalone page numbers; preserve the two body sentences and footnote `[^7]` *without changing its identifier or content*. Restore a single paragraph only if the surrounding source proves that it is one paragraph; otherwise preserve two paragraphs.

**Counterexample:** `The results are shown on p. 12.` contains a meaningful page reference and must be preserved.

## 28. Website heading that resembles navigation

**Input**:

```text
Paper title
ABSTRACT
One sentence of abstract.
REFERENCES
[1] Smith, J. (2024). A study of feedback.
Related research
People also read
Other paper title
```

**Expected:** Keep the paper title, abstract, `REFERENCES` heading and `[1]` entry. Remove `Related research`, `People also read`, and the unrelated teaser. Do not use a naive blacklist that deletes every section named `References`.

## 29. Quoted conversation embedded in one user turn

**Input**:

```text
用户：帮我分析下面这段聊天，但不要重排它：
> 用户：能帮我检查吗？
> AI：可以。
AI：你粘贴的是一个两轮对话的引用。
```

**Expected:** **Two active turns only**: the actual user's request (including the two quoted lines) and the AI's response. The quoted `用户：` and `AI：` must not become new, active turns.

## 30. Decoding entities without damaging literal syntax

**Input**:

```text
AI&#x20;and&#x20;assessment &amp; feedback
`&amp;` is an example HTML entity.
```

**Expected:** The prose becomes `AI and assessment & feedback`; the inline-code literal `` `&amp;` `` remains unchanged.

## 31. User explicitly requests an unfiltered copy

**User instruction:** `请完整保留网页全部导航、推荐、页脚和操作控件，只修复空格。`

**Expected:** **Do not** perform article-only extraction; preserve those extra page sections, even though the default webpage profile would remove them. Only fix allowed mechanical defects.

## 32. Uncertain speaker or main-content boundary

**Input:** A fragment without role markers that could be either a user quoting another AI or a direct AI answer.

**Expected:** Preserve the fragment with `**说话者无法确认：**` (or its original role label) instead of inventing a user turn. In webpage mode, if it is unclear whether a small article-like passage is related content or a section of the target article, preserve it until the boundary is clear; do not delete based solely on its position.

## Combined acceptance criteria

- Main content is intact and in source order; clear web navigation and app chrome are absent except when explicitly requested.
- Academic bylines, abstracts, real references, footnotes and dates are retained without inventing citation metadata.
- Genuine user prompts and AI final answers are distinguished, complete, chronological, and consistently labeled across *all* turns.
- Substantive source-visible process text is formatted separately; routine progress/status messages and generic platform disclaimers are absent by default.
- Original strings, code blocks, URLs, tables and facts are not rewritten; only allowed mechanical corrections and structural labels are introduced.
- Re-cleaning the already-cleaned result with the same profile yields no further substantive edits.
