# Proposal

## Why

OpenTodo ships in six languages (English, German, French, Spanish, Italian, Portuguese). Chinese — around 1.1 billion speakers, and the largest self-hosting and home-server community after English — is not among them. A Chinese speaker reads the app in English today, and every piece of text around the app (the browser extension, the Android and iOS shells, the emails, the notification channel messages) is English too.

This is a personal fork (`AIDockerCloud/OpenTodo`) whose owner runs a self-hosted instance. The motivation is use, not upstream growth: the interface, the phone shells and the emails should read Chinese. The work is still shaped as a normal OpenSpec change so it can be carried upstream without a rewrite.

## What Changes

- **Simplified Chinese (`zh`) as a shipped language.** A complete catalog for the web app (`web/src/locales/zh.json`), the service worker's notification texts, the browser extension, the Android shell, the iOS shell's String Catalog, the server's emails (sign-in, invitations, the weekly digest, routines) and the notification channel messages. Settings → Language and region → Language lists `简体中文`.
- **A Chinese glossary column** in `docs/translating.md`, so the terms every surface shares (Task, Project, Inbox, Today, Settings …) are decided once.
- **Not BREAKING.** No new environment variable, no op schema change, no API change: `user_prefs.language` already accepts any code and older clients read an unknown one as `system`.

## Out of scope

- **Chinese quick add.** The quick-add tokenizer splits on whitespace (`web/src/lib/nlp/tokenize.ts`), so a Chinese line is one token and matches nothing. Chinese quick add needs a segmenter, not a vocabulary: it is its own change.
- **Traditional Chinese (`zh-Hant`).** `resolveLanguage` (`web/src/lib/i18n/locale.ts`) matches on the primary subtag, so `zh-TW` and `zh-Hans` both resolve to `zh`; offering both needs full-tag resolution first.
- **A bundled Chinese typeface.** The interface font is an 84 KB Latin-subset of Inter; a CJK subset is measured in megabytes and would break the first-load budget. Chinese text falls back to the reader's system font (design D4).
- **Translating the documentation and the in-app Help**, which stay English by the `localization` spec.
- **The `ot` CLI's output**, which stays English.

## Roadmap

Relates to the `more-languages` change (Dutch and the community-language tier) without depending on it: both add a language, neither needs the other's contributor kit. Once `more-languages` lands, `zh` becomes a community language with the same rules.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `localization`: Simplified Chinese joins the shipped languages; the catalog's own rules for a language whose plural system has one category.

## Impact

- **Data model, sync, op schema:** none. `language: "zh"` is an existing field with a new value.
- **E2EE:** none; no content fields.
- **API, CLI, MCP:** none. `internal/mail` gains `zh` in its language list and twelve `.zh.txt` templates; `internal/channels/messages.go` gains a `zh` table.
- **Bundle and binary size:** the Chinese catalog is its own lazily loaded chunk, never in the first load. Measured: 515 KB raw and 178 KB gzipped, about the same download as German (574 KB raw, 178 KB gzipped).
- **Fonts:** none added (design D4).
