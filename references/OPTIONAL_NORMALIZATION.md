# Optional Level 3 — User-Authorized Normalization

**Status: disabled by default.** Read and apply only when the user's instruction explicitly requests a Level 3 operation. Never suggest a menu of these options during ordinary text cleaning, ask the user to choose among them, or announce that these rules were not enabled. A request for “文本清洗”“排版整理”“格式规范” without a specific transformation runs Levels 1–2 only.

**Scope rule:** activate **only the operation(s) actually named**; do not silently enable adjacent operations. User instructions outweigh the examples below. First protect URLs, code, identifiers, formulas, tables, bibliographic metadata, quoted source material and proper names where changes would damage their meaning or traceability. When the target includes such protected spans explicitly, make narrowly scoped changes with verification. If uncertain, preserve the original rather than inventing missing information.

## A. Script and variant-character conversion

**Triggers:** “全文转简体 / 转繁体 / 按台湾正体转写 / 将异体字统一成指定字形”.

- Preserve proper names, quotations, historical writing and languages other than the requested Chinese variety unless the request clearly includes them.
- For ambiguous mappings, use sentence context and the specified target variant. Do not blindly map all characters one-to-one: e.g., `后/後`, `发/發/髮` can require context; Taiwan/Hong Kong/Mainland usages are not interchangeable in every case.
- A request to “统一中文格式” without a requested script is insufficient to authorize Simplified/Traditional conversion.

## B. Spelling, typos and grammar

**Triggers:** “纠正错别字 / 拼写检查并修正 / 校对语法”.

- Only correct confidently identifiable problems within the requested scope. Retain rare proper nouns, technical words, quotes and original source citations unless the user explicitly requests their correction.
- Do not silently adjust the factual claim, translation, style, intended emphasis, or speaker's position. If multiple repairs are plausible and no original is available, keep the token unchanged or flag the uncertainty only if the user requests annotations.
- Spelling, typo and grammar editing are **separate controls**; asking to fix spelling does not authorize rewriting grammar or style.

## C. Case, punctuation, and expressive repetition

**Triggers:** “把普通英文改为小写 / 统一英文大小写 / 按指定标点标准转换 / 删除刷屏式重复字”.

- Apply the requested transformation to the requested subset only. Protect `iPhone`, `GitHub`, `OpenAI`, abbreviations, initialisms, proper names and technical case-sensitive identifiers when the request is to normalize general prose; override only when the user clearly demands literal wholesale case conversion.
- If explicitly asked to standardize punctuation, choose Chinese or English punctuation based on the surrounding language and specified style; retain quotes, bibliographies and literal examples unless in scope.
- Even when repetition cleanup is enabled, treat `好好好好`, `？？？`, elongated speech and other expressive language as potentially meaningful. Only reduce **mechanical duplication** by default, and reduce rhetorical repetitions only when the user explicitly asks for that content-level edit.

## D. Date, numeral, units, and terminology

**Triggers:** “所有日期改为 YYYY-MM-DD / 中文数字统一阿拉伯数字 / 单位统一为 km / 用术语表统一简称”.

- **Do not normalize dates just because they have different but valid forms.** Convert only when the user specifically requests a target format. Preserve ambiguous `03/04/2026` unless day/month order can be established from source evidence.
- Numeral expression: change only where values and relationships are unambiguous. Protect chapter numbers, reference IDs, dates, years, poem titles, serial numbers, numeric ranges and idioms unless specifically included.
- Unit **notation** and unit **conversion** are different requests. For notation-only requests, preserve original magnitude. For true conversion, honor required precision, significant figures and target units; do not silently introduce approximations. A valid original `20GB` does not need harmonization merely because another paragraph uses `20 GB`.
- Terminology/abbreviation unification requires an explicit target form or a clearly provided glossary. `US`, `USA` and `美国` are not automatically interchangeable in all contexts; preserve named institutions and official titles.

## E. Output and verification

- Run the default Level 1–2 cleanup first (or as appropriate), then apply only the authorized Level 3 operation(s); never use optional transformations to justify omitting source sentences or reordering an AI dialogue.
- Compare all resulting numbers, dates, units, names, citations, annotations and quotes to the source. Every changed meaning-bearing token must be justified by the actual user instruction and reliable context.
- Return **only the requested cleaned text** by default, without a follow-up prompt, option menu, editorial disclaimer or separate change log. Provide an annotated diff/review only when requested.

See [`LEVEL1_LEVEL3_CASES.md`](LEVEL1_LEVEL3_CASES.md) for activation/isolation and edge-case acceptance tests.
