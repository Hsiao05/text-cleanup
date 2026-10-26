---
name: text-cleanup
description: Clean and structurally organize messy Chinese, English, and mixed-language text while preserving source meaning and wording. Use for copied PDF/OCR text, pasted full web pages, AI chat exports (including multi-turn chats, visible process notes, and navigation clutter), transcripts, plain text, or Markdown with hard wraps, broken/joined words, stray spaces, page headers/footers, and formatting debris. Extract primary page content, recover user/AI turns, and distinguish AI process text from final answers. Supports configurable Chinese–Latin/Chinese–number spacing.
---

# Text Cleanup and Organization

## Purpose and boundary

Use this skill to turn already available text into a readable, faithful document. Repair **mechanical and layout defects**, not its ideas or writing quality. Suitable inputs include PDF-extracted text, OCR transcripts, **whole-page web copy**, **AI chat transcripts**, speech-to-text exports, rough notes, and malformed Markdown.

This is a **cleanup and structural-restoration** task, not summarization, paraphrasing, proofreading, translation, fact-checking, or editorial revision. **Web-page and AI-chat inputs additionally authorize removing identifiable UI/navigation/operational noise**; do not treat deleting central article text or genuine messages as cleanup. It complements `pdf-to-markdown`: that skill extracts/transcribes a source and may describe images; this skill cleans text that has already been extracted. If the user supplies a PDF and asks for extraction, perform extraction with the appropriate source-reading workflow before applying these cleanup rules. Never invent material missing from an extracted source.

**Priority order:** (1) explicit user requirements and chosen cleaning mode; (2) correct identification of main content/actual dialogue and preservation of its words, facts, and structure; (3) cleaning identifiable peripheral noise; (4) correctness of token boundaries, paragraphs, and reading order; (5) typography. A style rule never licenses a content change. **The no-omission rule applies to content retained after source-type filtering**, not to identifiable website chrome or chat application controls.

## Default operating profile

Unless the user specifies otherwise:

| Setting | Default | Allowed alternatives |
| --- | --- | --- |
| Cleaning depth | Faithful mechanical cleanup + source-aware extraction | Content-only page mode, conversation-only mode, or no deletion if explicitly requested |
| Chinese ↔ Latin-letter spacing | `add` | `preserve`, `remove` |
| Chinese ↔ Arabic-numeral spacing | `add` | `preserve`, `remove` |
| Number ↔ ordinary Latin unit spacing | `add` where applicable | Follow explicit formatting instructions |
| Paragraphs | Restore actual boundaries; discard visual-only wraps | Preserve source line breaks if requested |
| Output | Cleaned text, without a preface; retain the input's format where feasible | Markdown, plain text, or another explicitly requested format |
| AI process notes in chat | Keep **substantive, visibly copied** process text in a marked block; remove routine status/debug/UI notices | `keep-all`, `remove-all` on request |
| Speaker labels in chat | `用户：` and `AI：` for every confidently identified turn | Keep original role names, e.g. `Claude：` or `DeepSeek：`, on request |
| Comments/change log | None | Provide separately if requested |

Interpret spacing overrides precisely:

- **“中英文之间不添加空格”** → Chinese/Latin mode `preserve`: do not introduce spaces at those boundaries, but do not strip existing intentional ones. **“中文和数字之间不添加空格”** independently sets Chinese/numeral mode `preserve`.
- **“中英文、中文数字之间都不要加空格”** → set both to `preserve`.
- **“中英文之间不要留空格 / 删除所有中英文之间的排版空格 / 紧凑排版”** → use `remove` for the specified boundary type(s), but retain word separators and protected syntax.
- **“按中文排版规范加空格”** → use `add` for both.
- **“保持原文空格 / 不要调整排版空格”** → respect that request; only fix the explicitly identified defects.
- Requests about Chinese↔Latin spacing do **not** abolish required spaces between English words, or silently change units (`10 GB`), punctuation, or code.

Prefer the user's document-wide established convention over the defaults. Don't ask which profile to use unless the ambiguity will materially affect the requested result.

## Processing workflow

### 1. Inspect and protect before editing

Read **all available text** and first classify the material as **document/OCR**, **copied webpage**, **AI chat log**, or **mixed**. Determine language(s), intended reading order, paragraphs, headings, lists, quotations, tables, references, notes, speaker changes, and explicit layout conventions. Apply source-type rules below **before** broad whitespace normalization. For mixed inputs, identify subregions and apply the corresponding policy locally. Distinguish raw plain text from actual Markdown. For long inputs, process by logical sections while preserving continuity and cross-section references.

Identify spans that must be treated as **opaque** unless the user asks to edit them: fenced/inline code, commands, file paths, URLs, email addresses, identifiers, Markdown link destinations and syntax, HTML/XML tags, YAML front matter, equations/LaTeX, scientific notation, regular expressions, JSON, CSV/TSV fields, time stamps, numeric data, and metadata. Preserve literal formatting inside diagrams, poetry, verse, code blocks, lyrics, fixed-width layouts, and preformatted quotations. Protect Markdown tables and existing footnote definitions before any global whitespace operation.

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

### 2. Remove meaningless whitespace and encoding artifacts

- Normalize line endings and accidental whitespace runs in ordinary prose. Replace multiple ordinary interword spaces, stray tabs, nonbreaking spaces, and copy/paste artifacts with the one correct separator **only when their function is clear**.
- Remove spaces between adjacent Chinese characters when they are obvious layout debris: `文 本 清 洗` → `文本清洗`. Keep genuine separation between field values, table cells, item labels, poetry, and deliberate letterspacing.
- Strip incidental spaces just inside ordinary Chinese punctuation or at prose line edges. Preserve spaces with syntactic purpose.
- Preserve original Chinese/traditional characters, accents, diacritics, IPA, mathematical symbols, emoji sequences, and linguistic notation. Do not indiscriminately use Unicode NFKC normalization, remove all zero-width characters, or collapse whitespace across protected structures. Normalize an invisible artifact only when it is demonstrably extraneous.
- Remove repeated PDF-style page numbers, running headers/footers, or duplicated extraction fragments **only if identifiable as layout artifacts** (including `Page 5`, `— 5 —`, repeated journal/chapter title, recurring legal footer, scan watermark, or website sticky header). Check page position, repetition, and surrounding context. Do not remove actual dates, bibliography numbers, source quotes, meaningful repetition, or note text.

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

**Punctuation.** Fix obvious spacing around punctuation and unambiguous Chinese full-width versus English half-width punctuation in ordinary prose. Follow Chinese-copywriting conventions where safe: Chinese-prose commas, colons, brackets, and sentence-ending punctuation usually use full-width marks; standalone English sentences and special English names use half-width marks. Preserve intended punctuation choices, quote text, ellipses, stylistic emphasis, source errors, and bibliographic punctuation. Do **not** suppress repeated exclamation/question marks or change quotation-mark styles merely because a style guide prefers another form. Do not replace `“”` with `「」` by default.

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
- Chinese ↔ Arabic digits: `共有3项结果` → `共有 3 项结果`.
- Ordinary numbers ↔ alphabetic units: `20GB` → `20 GB` only where `GB` is clearly a unit, not part of a product/identifier.
- Percentage/degree sign: `15%`, `90°`; do not introduce `15 %` or `90 °`.
- Full-width Chinese punctuation normally has no padding: `使用 AI，效果明显。`, not `使用 AI ， 效果明显。`.
- Preserve official spellings and internal formatting (`豆瓣FM` if that is the source name, `GitHub`, `Wi-Fi`, `IPv6`, `H₂O`, `GPT-6`, `2FA`, version numbers).
- Do not alter a link URL or markup to manufacture display-text spacing. Spacing **outside** protected inline markup may be normalized if syntax and user preference permit.

Example of the three Chinese↔Latin and Chinese↔number profiles (with both set to the same value):

```text
Input:     这个AI工具有3个功能。
add:       这个 AI 工具有 3 个功能。
preserve:  这个AI工具有3个功能。
remove:    这个AI工具有3个功能。
```

`preserve` and `remove` differ when source text *already* has boundary spaces: `这个 AI 工具有 3 个功能。` stays unchanged under `preserve`, but becomes `这个AI工具有3个功能。` under `remove`. These modes change only optional Chinese-adjacent typographic spaces, not all spaces.

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
4. Chinese↔Latin and Chinese↔number profiles were applied **independently** and consistently; exceptions still work.
5. Markdown links, tables, footnotes, quotes, code, formulas, and indentation still parse correctly.
6. No spelling/grammar/content rewriting, unsupported inference, or unexpected capitalization changes occurred.
7. Page/article metadata and full primary body were not confused with related links; headers/footers/page numbers were deleted only where clearly non-content. In chat mode, every real user/AI turn remains in sequence, substantive retained process text is clearly marked, and no synthetic question appears.
8. An already-cleaned document would remain effectively unchanged under the same settings (idempotence).

For an ambiguous token boundary, name, reference, OCR reading, speaker assignment, main-content boundary, or paragraph relationship, **preserve the original**. Do not invent a replacement just to produce a tidy document. If the user requests a report, list such unresolved cases separately; otherwise return the faithful cleaned text without inserting editorial notes into the body.

## Output contract

Return **only the cleaned text** by default: no greeting, explanation, summary, apology, change log, or enclosing Markdown code fence. Retain original structure and format where possible. If the user asks for Markdown, output properly formed Markdown; if they ask for a file, create the requested text/Markdown file. If they explicitly request an annotated diff or explanation, provide it separately from the cleaned document.

Do not silently truncate long inputs. When output/file limits prevent full completion, clearly identify what portion was processed rather than presenting partial text as complete.

## Additional examples and edge cases

See [`references/TEST_CASES.md`](references/TEST_CASES.md) for basic cleanup and spacing tests, and [`references/WEB_CHAT_CASES.md`](references/WEB_CHAT_CASES.md) for full-page extraction, multi-turn AI conversation reconstruction, visible-process formatting, and deletion boundaries. Apply the policy above to each case; examples are not a license for blanket token replacements.

## Source guidelines

- User's `pdf-to-markdown` skill: fidelity, paragraph and heading logic, footnotes, table integrity, reading order, and no invented text.
- [sparanoid/chinese-copywriting-guidelines](https://github.com/sparanoid/chinese-copywriting-guidelines): Chinese typography defaults for spacing, punctuation, full-/half-width usage, and name capitalization. Its optional/debatable preferences must **not** override source fidelity or the user's explicit style choice.
