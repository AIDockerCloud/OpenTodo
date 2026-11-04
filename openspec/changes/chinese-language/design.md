# Design

## Context

Translated text lives in eight places, each with its own list of six languages:

| Surface | Where | Language list |
|---|---|---|
| Web app | `web/src/locales/<code>.json`, `SHIPPED` in `web/src/locales/index.ts` | `SHIPPED` |
| Service worker notifications | table in `web/public/push-sw.js` | the table's keys, pinned by a test |
| Browser extension | `extension/_locales/<code>/messages.json` | folders, listed in `extension/locales.test.js` |
| Android shell | `mobile/android/app/src/main/res/values-<code>/strings.xml` | folders |
| iOS shell | `mobile/ios/Shared/Localizable.xcstrings` | its localizations, checked against `SHIPPED` by `scripts/i18n/shell.mjs` |
| Emails and digest | `internal/mail/templates/*.<code>.txt`, `mail.Languages` | `mail.Languages` |
| Channel messages | `internal/channels/messages.go` | its table |
| Quick-add vocabulary | `web/src/lib/nlp/vocab/<code>.ts`, `internal/quickadd/vocab.json` | `quickadd.Bundled` |

`scripts/i18n-check.mjs` treats a catalog that `SHIPPED` does not list as work in progress: it prints "zh: 412 of 7082" and never fails. It becomes strict — every message present, every placeholder, tag and plural argument matching — the moment `zh` is added to `SHIPPED`. That is the lever this change uses: the catalog is translated and checked in the loose mode, then registered in one commit that flips every surface at once.

## Decisions

**D1. `zh` means Simplified Chinese, written `简体中文`.** `resolveLanguage` matches on the primary subtag (`zh-CN` → `zh`, `zh-Hans` → `zh`, `zh-TW` → `zh`), so one code serves every Chinese browser. Traditional Chinese would need full-tag resolution and a second catalog; it is out of scope (proposal).

**D2. An AI drafts, the maintainer reviews.** The `localization` spec forbids machine translation for a shipped catalog, and every entry here is machine-drafted. Two honest outcomes: as a personal fork the constraint does not bind and `zh` ships as drafted; as an upstream pull request the native-speaker maintainer reviews every entry before `zh` joins `SHIPPED`, and is named in `docs/translating.md`. The tasks keep the review as the last gate before registration, whichever route is taken.

**D3. Chinese plurals use one category.** `Intl.PluralRules('zh')` resolves to `["other"]`, so a plural source collapses to its `other` branch alone: `{n, plural, one {# task} other {# tasks}}` → `{n, plural, other {# 个任务}}`. `scripts/i18n-check.mjs` fails a translation that uses `one` for `zh` and one that omits `other`, so this is enforced, not a style choice.

**D4. No bundled Chinese typeface.** The interface font is an 84 KB Latin subset of Inter with an explicit `unicode-range`; Chinese code points fall through to the reader's system font (PingFang on macOS and iOS, Microsoft YaHei on Windows, Noto Sans CJK on Android and most Linux desktops). A CJK subset that covers the catalog is several megabytes in the smallest useful form and would break `web/bundle-budget.json` for a language the first load never needs. The cost is that Chinese glyph width and weight differ slightly from the Latin text around it; the alternative is a font download per Chinese user on first load, which the change rejects.

**D5. Chinese quick add is a separate change.** `tokenize()` splits the line on whitespace, so `明天下午三点开会` is a single token that matches no vocabulary entry, and the parser has no way to strip `明天` from the title. A vocabulary file alone would produce wrong titles. Chinese needs character-level segmentation and a different set of grammar rules (no spaces before `下周三`, a classifier between numbers and units, `点`/`分` for times). Out of scope here; in this change a Chinese user types dates in English, as every non-six-language user does today.

**D6. Register and punctuation.** Plain, direct Chinese: imperative for buttons (`删除`, not `请删除`), no honorific `您` (the equivalent of the other catalogs' *du*/*tu*/*tú*), full-width `，。：；？！` and `“”`, half-width digits and Latin, no space between Chinese and Latin. `Settings → Security` becomes `设置 → 安全`, keeping the ` → ` separator the check reads. Terms follow the glossary in D7 and in `docs/translating.md`.

**D7. One glossary, in `docs/translating.md`.**
The terms live in the glossary column of `docs/translating.md`, beside the other
five languages, so there is one place to read them and one place to change them —
the same place the Dutch change (D1 of `more-languages`) puts its own. Simplified
Chinese adds a few of its own (例行任务 for Routine, 专注 for Focus, 优先级 for
Priority) and fixes a few it must not drift on:

- Section is 分区, never 章节, which is a chapter of a document.
- Due date is 截止日期 and Deadline is 最后期限; the two never share a word, and
  a shortcut that sets a due date says 设为…到期 rather than 截止.
- The calendar keeps its three apart: a slot is 时段, a time sector is 时间区间
  and a frame is 时间框.
- An occurrence of a recurring task is 这一次 where it is the button and 实例
  where it is counted.

**D8. Batches of one source file.** Messages are translated in fragments kept outside the repository (`~/tmp/zh/batches/`) and merged into `web/src/locales/zh.json` by a small script, ordered largest-first so that every batch is one feature's text and reuses its neighbours' wording. After each merge `node scripts/i18n-check.mjs` prints the new count and the fragment is kept for review. The catalog file is written in the order of `messages.json` so a diff between two batches is a pure insertion.

**D9. Verification without `npm ci`.** The repository's JavaScript chain (vite, vitest, storybook) needs `web/node_modules`; the translation checks do not — they need `typescript` only, which `scripts/i18n/source.mjs` requires from `web/package.json`. A `npm i typescript` installs enough for `node scripts/i18n-check.mjs` and `node --test scripts/i18n/*.test.mjs`, and `node --test extension/locales.test.js` needs nothing at all. The Android and iOS catalogs have no runner of their own outside those scripts, so their key and `%`-specifier parity is checked with the same rules by hand before registration (tasks 4.3, 4.4).

## Risks

- **A half-registered language.** `zh` in `SHIPPED` while any surface lacks it fails `make check`, `extension/locales.test.js` and `shell.mjs` at once. Mitigated by one registration commit, after every surface is complete (tasks 5.1).
- **Length, in the other direction.** Chinese is shorter than English, so truncation is unlikely; the risk is vertical — 13 px Chinese lines need slightly more line height than Latin at the same size. Reviewed on the phone-width screenshots in task 6.2.
- **A term that drifts.** `分区` for Section touches hundreds of strings; the glossary is applied from the first batch and `docs/translating.md` records it, so a review can grep for the wrong variant.
