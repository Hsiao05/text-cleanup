---
name: text-cleanup
description: Faithfully clean and restore messy Chinese, English, and mixed-language text from PDF/OCR copies, pasted webpages, AI chat transcripts, and Markdown. Fix broken/fused words, hard wraps, stray/invisible characters, encoding artifacts, fullwidth Latin/digits, page headers/footers, web UI clutter, and chat turns without rewriting content. Configurable CJK/Latin/numeral/unit spacing; optional proofreading and other semantic normalization only upon explicit user request.
---

# Text Cleanup and Organization

## Purpose and boundary

Use this skill to turn already available text into a readable, faithful document. Repair **mechanical and layout defects**, not its ideas or writing quality. Suitable inputs include PDF-extracted text, OCR transcripts, **whole-page web copy**, **AI chat transcripts**, speech-to-text exports, rough notes, and malformed Markdown.

The **default (Levels 1–2)** is cleanup and structural restoration, not summarization, paraphrasing, proofreading, translation, fact-checking, or editorial revision. **Web-page and AI-chat inputs additionally authorize removing identifiable UI/navigation/operational noise**; do not treat deleting central article text or genuine messages as cleanup. **Level 3** comprises independent, opt-in content normalizations; never apply them merely because a text could be made more consistent. It complements `pdf-to-markdown`: that skill extracts/transcribes a source and may describe images; this skill cleans text that has already been extracted. If the user supplies a PDF and asks for extraction, perform extraction with the appropriate source-reading workflow before applying these cleanup rules. Never invent material missing from an extracted source.

**Priority order:** (1) explicit user requirements and chosen cleaning mode; (2) correct identification of main content/actual dialogue and preservation of its words, facts, and structure; (3) cleaning identifiable peripheral noise; (4) correctness of token boundaries, paragraphs, and reading order; (5) typography. A style rule never licenses a content change. **The no-omission rule applies to content retained after source-type filtering**, not to identifiable website chrome or chat application controls.

## Default operating profile

Unless the user specifies otherwise:

| Setting | Default | Allowed alternatives |
| --- | --- | --- |
| Cleaning depth (Level 1) | Faithful mechanical cleanup + source-aware extraction | Content-only page mode, conversation-only mode, or no deletion if explicitly requested |
| Chinese ↔ Latin-letter spacing (Level 2) | `add` | `preserve`, `remove` |
| Chinese ↔ Arabic-numeral spacing (Level 2) | `add` | `preserve`, `remove` |
| Number ↔ ordinary Latin unit spacing (Level 2) | `add` only in the standard spacing profile | `preserve` or `remove` with a no-space profile or explicit instruction |
| Paragraphs | Restore actual boundaries; discard visual-only wraps | Preserve source line breaks if requested |
| Output | Cleaned text, without a preface; retain the input's format where feasible | Markdown, plain text, or another explicitly requested format |
| AI process notes in chat | Keep **substantive, visibly copied** process text in a marked block; remove routine status/debug/UI notices | `keep-all`, `remove-all` on request |
| Speaker labels in chat | `用户：` and `AI：` for every confidently identified turn | Keep original role names, e.g. `Claude：` or `DeepSeek：`, on request |
| Optional Level 3 normalization | **Off**; do not suggest or ask about it in ordinary cleanup | Activate only the specific operation(s) explicitly requested by the user |
| Comments/change log | None | Provide separately if requested |

**The spacing profile has three boundary types:** Chinese↔Latin letters, Chinese↔Arabic numerals (including numbers next to Chinese units), and ordinary numbers↔Latin alphabetic units (`20 GB`, `5 km`, `10 kg`). Interpret instructions as follows:

- **“中西文之间不加空格 / 中英文和数字都不加空格 / 混排不添加空格”** → `preserve` for **all three** boundary types. In particular, do **not** turn `20GB` into `20 GB` or `20公斤` into `20 公斤`. Existing ordinary gaps stay unless removal is requested.
- **“中西文之间不要有空格 / 紧凑排版 / 把中西文及数字单位间的空格都删掉”** → `remove` for **all three** boundary types, where the gap is only optional typography (e.g. `20 GB` → `20GB`). Never remove syntactically or semantically required spaces.
- **“中英文之间不添加空格”** → Chinese/Latin `preserve`, and **also** number/Latin-unit `preserve`; Chinese/numeral retains the established mode **unless** the request refers to all mixed-language spacing. **“中文和数字之间不添加空格”** sets Chinese/numeral `preserve` independently.
- **“中英文之间不加空格，但中文数字按标准排版”** → Chinese/Latin `preserve`, Chinese/numeral `add`, number/Latin-unit `preserve` unless the user specifies a different unit convention.
- **“按中文排版规范加空格”** → `add` for all three unless a more specific override exists.
- **“保持原文空格 / 不要调整排版空格”** → preserve the source's optional typographic gaps; repair only clearly corrupt spacing.
- **No-add (`preserve`) and no-space (`remove`) are different:** `20 GB` remains `20 GB` under `preserve` but may become `20GB` under `remove` when it is ordinary prose and not a protected expression. Required spaces between **different English words**, mathematical tokens, code and other protected structures always remain.

Respect explicit per-boundary exceptions. Do not ask for a spacing preference or promote optional normalization during routine processing; follow the default if nothing relevant was specified.

## Processing workflow

### 1. Inspect and protect before editing

Read **all available text** and first classify the material as **document/OCR**, **copied webpage**, **AI chat log**, or **mixed**. Determine language(s), intended reading order, paragraphs, headings, lists, quotations, tables, references, notes, speaker changes, and explicit layout conventions. Apply source-type rules below **before** broad whitespace normalization. For mixed inputs, identify subregions and apply the corresponding policy locally. Distinguish raw plain text from actual Markdown. For long inputs, process by logical sections while preserving continuity and cross-section references.

Identify spans that must be treated as **opaque** unless the user asks to edit them: fenced/inline code, commands, file paths, URLs, email addresses, identifiers, Markdown link destinations and syntax, literal HTML/XML tags, YAML front matter, equations/LaTeX, scientific notation, regular expressions, JSON, CSV/TSV fields, time stamps, numeric data, and metadata. Preserve literal formatting inside diagrams, poetry, verse, code blocks, lyrics, fixed-width layouts, and preformatted quotations. Protect Markdown tables and existing footnote definitions before any global whitespace operation. **Recover human-readable pasted text without treating literal source examples as formatting debris.**

Do not erase or reinterpret leading/trailing spaces that have syntactic meaning (Markdown hard breaks, nested lists, code indentation, tabular columns, etc.). Do not apply a broad regex to the entire document when it would touch protected spans.

### 1A. Extract the primary content of a pasted web page

This section applies when the user pastes an **entire rendered page** rather than only the desired article, post, help text, or discussion. The user normally wants the **main page content**, not a textual dump of the website interface. If they explicitly request a complete accessibility/UI transcript, disable page filtering.

1. Locate the central content using title, author/date/byline, article body, abstract, main section headings, URL/page context (when present), and continuity. For a discussion/thread, preserve **relevant posts, quoted replies, comment authors, and reply order**. For an article, retain title, authors, publication/revision date, publisher/journal when identifiable, issue/volume/pages/DOI when clearly attached, abstract, body, figures/tables/captions as available, inline citations, references and substantive notes. Do **not** fabricate missing metadata or treat a web publication date as the original article date without evidence.
2. Remove clearly peripheral website chrome: `Skip to Main Content`, logo/site home links, login/register, account/cart, search fields and filters, navigation/breadcrumbs, share/print/download buttons, cookie dialogs, language selectors, sticky footers, `Back to top`, promotional banners, recommendation carousels, `People also read`, `Related articles`, `Cited by` lists, broad subject-directory links, social widgets, newsletter signup, copyright/address/legal/site policies, page counters (`Views`, `Altmetric`, generic citation counts), and `Previous/Next article` controls **when they are only navigation**. These are categories, not a mandatory keyword blacklist.
3. Do **not** delete a main-body section solely because it says `References`, `Related Work`, `Methods`, `Comments`, or `Citations`. Distinguish real document sections from similarly named site tabs, advertising, and recommendation modules using surrounding structure and content. Main discussion comments can be central evidence, whereas an unrelated recommended article teaser normally is not.
4. Extract one coherent primary item when the page clearly focuses on one item. If the requested scope includes several items, preserve all of them in original order. If a passage's status (primary vs. peripheral) is uncertain, **keep it rather than silently delete it**. Keep valuable publication metadata even if displayed near navigation.
5. Normalize reliable HTML entity debris in plain text, e.g. `&#x20;`, `&nbsp;`, `&amp;`, `&quot;`, only when it is rendered-text encoding noise. Do not decode text in source code, literal examples, URL query strings, markup, or Markdown syntax where this would change meaning.
6. Avoid copying browser/site UI into the result merely to mirror the input. Do not summarize the article: remaining prose is preserved verbatim except for mechanical fixes. If the page copy contains only a title and abstract rather than the full article, output only those; **never claim the full text was recovered**.

### 1B. Reconstruct AI conversations as multi-turn records

This section applies to ChatGPT, Claude, DeepSeek, Gemini, and other AI chat pages pasted in full, including those containing a sidebar history, tool notices, reasoning previews, and multiple prompts. **Don't assume everything between two user questions is an AI final answer.**

1. **Strip chat-app chrome first:** conversation-list titles, sidebars, sign-in/navigation, model picker, `Retry`, `Regenerate`, `Copy`, `Edit`, voting/share buttons, empty input placeholders, generic disclaimers, progress counters, temporary `Searching...`/`浏览 1 个页面`/`搜索到 24 个网页` notices, and similar operational metadata **unless it materially describes a step/result the user asked to document**. Generic notices like `Claude is AI and can make mistakes` are app footer text, not part of the exchange.
2. **Segment chronologically into turns.** Identify real roles from exported message markers first (`User`, `Human`, `You`, `Assistant`, model name, timestamps/role blocks). Where markers were lost, use adjacent dialogue and obvious request/response form. Preserve every identifiable distinct user message, including very short follow-ups and corrections. Do not merge several questions, reorder messages, or convert AI paraphrases of a user request into actual user prompts.
3. **Label each turn consistently:** `**用户：**` and `**AI：**` (or `**Claude：**` / `**DeepSeek：**` when a distinction is useful). For Markdown chat exports, put a blank line between turns; preserve existing answer paragraphs, headings, lists, code fences, and citations. Do not force a full AI answer into a single blockquote just to indicate it is an answer.
4. **Separate visible process text from the final answer.** Source-visible planning / reasoning previews / tool narration that are **substantive** should be preceded by `**AI 过程记录（原文可见，非最终回答）：**` and formatted as a Markdown blockquote (`> ...`) where feasible. Never represent such process as a user prompt, a verified fact, or the AI's final answer. Routine implementation chatter, UI tool-progress counters, duplicate status messages, and pure operational traces are discarded by default. The optional `keep-all` mode retains all visible process/tool events with explicit labels; `remove-all` drops process content, **but always retains actual user turns and final answers**. Do not invent, reconstruct, or expose unprovided hidden reasoning.
5. **Handle missing/orphan turns conservatively:** if the paste starts with an AI answer but no preceding user prompt, label it as an AI answer and **do not synthesize the question**. If no reliable speaker can be established, mark the segment `**说话者无法确认：**` or preserve the original visible label. If the input contains a quotation of a prior chat inside one user's prompt, keep it **inside that user's message**; do not misclassify quoted `用户：` / `AI：` as active turns.
6. **Separate 'thought', tool results, and answers by function, not keywords alone.** For example, an AI response teaching how to analyze reasoning can legitimately mention “思考过程”; it remains final-answer content. A `Searching for ...` log between two turns is not a user question. An AI analysis draft beginning `用户需要...` is usually process text, not an actual request from the user.
7. Do not summarize, combine or rewrite multiple rounds. Retain useful links, code examples, tables, decisions, cautions and answers within the actual dialogue. When a conversation contains an answer that was later amended, keep **both**, in order, so the evolution remains clear.

**Example 1 (webpage):** For an academic journal page containing `Skip to Main Content`, `Add to Cart`, a paper title, authors, page range, online date, `ABSTRACT`, recommendation cards, subject links and site footer, retain the **paper's title + attributable metadata + complete abstract/body that was actually pasted**. Discard journal navigation, shopping/search actions, metrics unrelated to article content, recommendations and legal site footer. See acceptance case 23.

**Example 2 (AI chat):** For a DeepSeek page copied with conversation-title sidebar, a genuine user question about processing X bookmarks, tool-status lines, visible AI planning text and a final solution, remove the sidebar/tool-status noise; label the question `**用户：**`, the final response `**AI：**`, and any retained substantive visible planning text separately as `**AI 过程记录…：**`. Preserve earlier turns if present; a previous AI summary does not automatically become a user prompt. See acceptance cases 24–25.

**Example 3 (multi-turn Claude chat):** For a paste beginning with an AI answer about NAS and cloud storage, followed by the user saying the budget is too high, another AI answer, a user asking about Windows-only local tagging, and another AI answer, retain the full **AI → 用户 → AI → 用户 → AI** sequence. Omit only navigation/progress/disclaimer artifacts, not the content of either answer. Do not invent the missing first question. See case 26.

### 2. Level 1 — Clean mechanical defects, encoding artifacts, and noise

Apply Level 1 by default **only to ordinary text spans**, after isolating protected syntax. These are repair operations, not a license to standardize style or wording. For each uncertain case, prefer retaining the original over a plausible-looking change.

**2A. Encoding and invisible-character safety**

1. If operating on an actual byte file, detect/read its real encoding before decoding when possible; don't blindly relabel or re-encode text. In already-decoded pasted text, normalize ordinary line endings (`CRLF`/`CR` → logical `LF`), remove a stray file-start BOM (`U+FEFF`), and decode unambiguously escaped HTML entities **only where they are rendered-text debris**. Preserve literal encoding examples, code, markup, URLs and query strings.
2. Identify replacement characters (`�`, `U+FFFD`), visible mojibake, or garbled sequences. Repair **only** when a unique recovery is demonstrable from available source or reversible evidence. A replacement character usually means information is lost: do not invent the absent character, reword around it, or claim to have restored the original. Keep the uncertain span in place unless the user specifically requests annotated uncertainty.
3. For ordinary prose, remove **clearly extraneous** zero-width space (`U+200B`), word joiner (`U+2060`), stray internal BOM, nonprinting control characters, and copy/paste artifacts. Remove discretionary soft hyphen (`U+00AD`) only when it is an accidental hyphenation/display artifact; determine whether to join adjacent word fragments using the word-boundary rules below.
4. **Never strip all invisible Unicode characters globally.** Protect emoji ZWJ (`U+200D`) sequences, variation selectors (`U+FE0F`), intentional ZWNJ/ZWJ in languages that need them, bidirectional-control contexts, combining marks, non-breaking formatting meaningful to formulas, and other intentional writing-system behavior. Retain line/tab controls when structural, and do not corrupt protected code or structured files.

**2B. Whitespace and fullwidth ASCII**

1. Normalize accidental multi-spaces and tabs in prose to the contextually correct separator; trim stray prose-edge whitespace and extra empty lines. Collapse layout-only spacing within a Chinese phrase (`文 本 清 洗` → `文本清洗`). Preserve real word boundaries, list/table columns, Markdown hard breaks/indentation, citations, fixed-width content, and intentional visual spacing.
2. Treat `U+00A0` (NBSP) and `U+3000` (ideographic space) as ordinary layout whitespace **only when clearly pasted as accidental prose padding**. Do not destroy their meaning in Japanese text, names, preformatted content or nonbreaking technical notation.
3. Convert **fullwidth ASCII Latin letters and digits** in ordinary, non-decorative prose (`ＡＢＣ１２３` → `ABC123`) when their width is an obvious input/paste artifact. Do **not** apply global Unicode `NFKC`, which can also alter circled numbers, mathematical notation, compatibility forms and symbols; do not automatically convert fullwidth punctuation, Japanese typography, citations, branded typography, or other deliberate character choices.
4. Preserve original Simplified/Traditional Chinese characters, variant glyphs, accents, diacritics, IPA, symbols and original wording. Default Level 1 **does not** translate, convert scripts, alter letter case, standardize dates/numerals/units, or correct spelling.

**2C. Source-aware peripheral noise and duplicates**

1. Remove identifiable page numbers, running heads/feet, recurring publication labels, scan overlays, website sticky headers, and duplicated extraction fragments only when context establishes that they are layout or extraction artifacts. Use repetition, placement and boundaries; don't delete standalone numbers if they may be references, list numbers, dates or values.
2. Remove webpage/application boilerplate such as navigation, banners, ads, footer copyright/legal text, `点击展开全文`/`点击查看更多`/`返回顶部` controls, unrelated recommended links, and operation buttons **only when they serve interface/navigation functions rather than content**. Preserve an article's actual author/editor credits, substantive copyright/licensing notices, references and attributed metadata when material to the primary item; `责任编辑：XXX` is **not** an automatic deletion trigger.
3. De-duplicate mechanically repeated headers/paragraph copies or obvious scrape-loop artifacts **only when genuinely redundant** and outside a deliberate quotation or argumentative repetition. Do **not** collapse expressive repetition (`好好好好`, `！！！！！`), parallelism, dialogue, or duplicate-looking but distinct evidence by default.
4. Follow the detailed webpage and conversation filtering rules in 1A–1B. Their special rules take precedence over generic keyword matches. If uncertain whether a segment is primary content, retain it.

### 3. Restore words and correct boundaries

These repairs require lexical and sentence-level evidence, not blind search/replace.

**Broken English words.** Join fragments only when they are clearly one word broken by OCR/spacing/line width:

```text
environ mental factors  →  environmental factors
inter-\nnational          →  international
```

Distinguish real compounds and hyphenation. Preserve `well-known`, `state-of-the-art`, minus signs, ranges, and intentional editorial hyphens. A terminal hyphen used **solely** for line wrapping may be removed; a lexical hyphen must remain.

**Words fused together.** Insert missing English word boundaries when grammar and vocabulary make the segmentation clear:

```text
Thisstudyexamineslanguagelearning. → This study examines language learning.
```

Some contiguous strings are valid names or tokens. `therapist`, `together`, `notebook`, `GitHub`, `iPhone`, `OpenAI`, `GPT-6`, `HTML5`, `5G`, and URLs must not be split by a generic algorithm. If several tokenizations are plausible, keep the source string rather than fabricate an interpretation.

**CJK and mixed-language boundaries.** Repair spaces accidentally inserted *inside* a meaningful Chinese expression, but never alter fixed brand names, original Japanese/Korean typography, technical symbols, or intentional separators. Do not assume that every Han character occurs in Chinese prose; apply the Chinese typography profile to Chinese-language contexts, not automatically to Japanese text.

**Punctuation (Level 2).** Fix unambiguous **spacing defects** around Chinese and English punctuation in ordinary prose. Follow Chinese-copywriting conventions where safe, but do **not** automatically convert legitimate source punctuation solely to enforce fullwidth/halfwidth uniformity. Preserve intended punctuation choices, quotation punctuation, ellipses, expressive repetition (`！！！！！`), bibliographic formats and stylistic emphasis. Do not rewrite `“”` as `「」` or merge repeated marks by default; optional content/style normalization belongs to Level 3.

### 4. Restore paragraphs and structural line breaks

Classify each original newline before changing it:

1. **Visual-only line wrap:** imposed by page/screen width and the sentence continues → merge into the same paragraph.
2. **Actual paragraph break:** supported by blank lines, indentation conventions, topic/speaker changes, source layout, or clear section structure → retain one blank line between paragraphs.
3. **Structural line break:** heading, list item, transcript speaker turn, address, poem/lyric line, code, table row, caption, or footnote → preserve appropriately.
4. **Uncertain boundary:** preserve the original separation instead of guessing a join.

When merging visual wraps, choose the separator according to the adjoining text: English word + English word normally needs one space; continuous Chinese prose normally does not; Chinese–Latin or Chinese–numeral boundaries follow the active spacing mode; punctuation, explicit hyphens, and special syntax override this rule.

```text
The author argues that
language learning changes over time.
→ The author argues that language learning changes over time.

人工智能技术的快速发
展正在改变写作方式。
→ 人工智能技术的快速发展正在改变写作方式。
```

Do not merge two genuine paragraphs because there was only one physical newline; do not split one paragraph arbitrarily for readability. Reconnect cross-page sentences only when the continuation is evident. Retain source reading order; never reorder sentences to improve flow.

### 5. Apply Chinese typography (configurable)

Default `add` follows the pertinent conventions of [Chinese Copywriting Guidelines](https://github.com/sparanoid/chinese-copywriting-guidelines), **subject to fidelity and the user's override**:

- Chinese ↔ English/Latin: `使用AI写作` → `使用 AI 写作`.
- Chinese ↔ Arabic digits: `共有3项结果` → `共有 3 项结果`, including numeric quantities adjacent to Chinese units (`20公斤` → `20 公斤`).
- Ordinary numbers ↔ alphabetic units in the **default standard profile**: `20GB` → `20 GB` only where `GB` is clearly a unit, not part of a product/identifier.
- **When mixed-language spacing is disabled, unit spacing is disabled with it**: `使用AI处理20GB文件` stays `使用AI处理20GB文件` under the general `preserve` profile; under the general `remove` profile, `使用 AI 处理 20 GB 文件` may become `使用AI处理20GB文件`. An explicit request concerning only Chinese↔number spacing does not automatically determine the independent Latin-unit rule.
- Treat **dates and times as structured notation**: do not re-space within an already valid date (`2026年10月26日`), rewrite it as `2026-10-26`, or change day/month order solely to fit a style preference. A clear outer number/Chinese boundary may still follow the chosen profile, but preserving the coherent date expression takes precedence. Date-format conversion is opt-in Level 3.
- Percentage/degree sign: `15%`, `90°`; do not introduce `15 %` or `90 °`.
- Full-width Chinese punctuation normally has no padding: `使用 AI，效果明显。`, not `使用 AI ， 效果明显。`.
- Preserve official spellings and internal formatting (`豆瓣FM` if that is the source name, `GitHub`, `Wi-Fi`, `IPv6`, `H₂O`, `GPT-6`, `2FA`, version numbers).
- Do not alter link URLs or markup to manufacture display-text spacing. Spacing **outside** protected inline markup may be normalized if syntax and user preference permit.

Example of the three coordinated spacing profiles (when no user override separates boundary types):

```text
Input:     这个AI工具有3个功能，处理20GB文件。
add:       这个 AI 工具有 3 个功能，处理 20 GB 文件。
preserve:  这个AI工具有3个功能，处理20GB文件。
remove:    这个AI工具有3个功能，处理20GB文件。
```

`preserve` and `remove` differ when source text *already* has optional gaps: `这个 AI 工具有 3 个功能，处理 20 GB 文件。` stays unchanged under `preserve`, but becomes `这个AI工具有3个功能，处理20GB文件。` under `remove`. These modes **never** remove actual English interword spaces, structure-bearing whitespace or technical notation that requires separate tokens.

When a mixed-language boundary coincides with actual source corruption, first restore the correct word/number tokens, then apply the selected typographic spacing profile. Never let `remove` join separate English words or destroy Markdown syntax.

### 6. Restore document structure with restraint

- Preserve original heading wording, numbering, and relative hierarchy. Convert to `#`, `##`, etc. only when Markdown is requested or already in use and a heading is structurally evident. Do not invent section titles or rename existing ones.
- Preserve meaningful list bullets/numbers, nesting, checkboxes, and item boundaries. Do not turn ordinary sentences into new lists merely for aesthetics.
- Preserve quotation boundaries, blockquote markers, captions, citations, and source-language punctuation.
- For simple tables, preserve header/cell/row alignment; reformat into a Markdown table only if requested or already represented that way and column relations are reliable. For merged/complex cells, retain an unambiguous existing structure; never infer missing cell values.
- Keep endnotes, footnotes, footnote markers, reference numbering, and their links intact. Never silently renumber or relabel them. Bibliographies retain the original ordering and citation style.
- Preserve placeholders such as `【图片内容：…】`, `[无法辨认]`, and formulas. Do not describe images you have not seen, invent chart data, resolve OCR ambiguities from guesswork, or delete uncertainty markers.
- Remove merely decorative separators only if demonstrably accidental; keep section separators if meaningful or required by Markdown.

### 7. Validate the cleaned document

Check silently, comparing against the source, that:

1. No sentences, names, numbers, units, dates, URLs, citations, notes, or table cells **within retained primary content/actual chat turns** were lost or added. Deletion of peripheral page/chat chrome is deliberate and documented by the source-type rules.
2. Clearly accidental internal English spaces and fused words were repaired; valid names and identifiers were not broken.
3. Only width-induced wraps were merged; all real paragraphs and structural newlines survive.
4. Chinese↔Latin, Chinese↔number, and number↔Latin-unit spacing follow the **requested coordinated/no-space profile** or any explicit per-boundary exceptions, consistently.
5. Suspect encoding debris, BOM, invalid controls, invisible characters and fullwidth Latin/digits were fixed only when safe; meaningful Unicode sequences were preserved.
6. Markdown links, tables, footnotes, quotes, code, formulas, and indentation still parse correctly.
7. No spelling/grammar/content rewriting, unauthorized Level 3 transformations, unsupported inference, or unexpected capitalization changes occurred.
8. Page/article metadata and full primary body were not confused with related links; headers/footers/page numbers were deleted only where clearly non-content. In chat mode, every real user/AI turn remains in sequence, substantive retained process text is clearly marked, and no synthetic question appears.
9. An already-cleaned document would remain effectively unchanged under the same settings (idempotence).

For an ambiguous token boundary, name, reference, OCR reading, speaker assignment, main-content boundary, or paragraph relationship, **preserve the original**. Do not invent a replacement just to produce a tidy document. If the user requests a report, list such unresolved cases separately; otherwise return the faithful cleaned text without inserting editorial notes into the body.

## Level 3 — Independent opt-in normalization (off by default)

**Do not load or invoke Level 3 for ordinary cleaning, do not prompt the user to enable it, and do not append a menu/suggestion after a cleaned result.** Only explicit user instructions such as “转为简体”“统一日期为 YYYY-MM-DD”“纠正错别字” authorize the **named** operation(s). “清洗一下 / 整理排版 / 格式统一” by themselves authorize Levels 1–2, not discretionary rewriting.

When an operation is specifically requested, use [`references/OPTIONAL_NORMALIZATION.md`](references/OPTIONAL_NORMALIZATION.md) for its scope, safeguards, and examples. The user may enable one operation without enabling any other. Do not force normalization of acceptable alternative formats (dates, units, proper names, punctuation, capitalization) simply for consistency. Preserve source data and original evidence when conversion is uncertain.

## Output contract

Return **only the cleaned text** by default: no greeting, explanation, summary, apology, change log, or enclosing Markdown code fence. Retain original structure and format where possible. If the user asks for Markdown, output properly formed Markdown; if they ask for a file, create the requested text/Markdown file. If they explicitly request an annotated diff or explanation, provide it separately from the cleaned document.

Do not silently truncate long inputs. When output/file limits prevent full completion, clearly identify what portion was processed rather than presenting partial text as complete.

## Additional examples and edge cases

See [`references/TEST_CASES.md`](references/TEST_CASES.md) for basic cleanup and spacing tests, [`references/WEB_CHAT_CASES.md`](references/WEB_CHAT_CASES.md) for full-page extraction and AI chat reconstruction, and [`references/LEVEL1_LEVEL3_CASES.md`](references/LEVEL1_LEVEL3_CASES.md) for Unicode/encoding safety, boilerplate deletion and opt-in checks. The separate [`references/OPTIONAL_NORMALIZATION.md`](references/OPTIONAL_NORMALIZATION.md) is **instructions for opt-in transformations only**, not part of the default cleanup. Examples never license blanket replacements.

## Source guidelines

- User's `pdf-to-markdown` skill: fidelity, paragraph and heading logic, footnotes, table integrity, reading order, and no invented text.
- [sparanoid/chinese-copywriting-guidelines](https://github.com/sparanoid/chinese-copywriting-guidelines): Chinese typography defaults for spacing, punctuation, full-/half-width usage, and name capitalization. Its optional/debatable preferences must **not** override source fidelity or the user's explicit style choice.
