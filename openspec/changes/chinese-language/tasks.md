# Tasks

## 0. Before starting

- [x] 0.1 Decide the route: personal fork (catalog ships as drafted) or upstream pull request (a named native-speaker maintainer reviews every entry, design D2); record the decision in this file
- [x] 0.2 `git checkout -b develop` off `main`; verify `git status` is clean apart from `.workbuddy/`

## 1. Glossary and tooling

- [x] 1.1 A `简体中文` column in the glossary of `docs/translating.md` (design D7), and the language added to the first paragraph's list only at 6.1; verify the table renders and every row has an entry
- [x] 1.2 Batch tooling outside the repository: `~/tmp/zh/batches/NN.json` fragments merged into `web/src/locales/zh.json` in `messages.json` order (design D8); verify a merge of an empty fragment leaves the file byte-identical
- [x] 1.3 `web/node_modules/typescript` only (`npm i typescript` without the rest of the tree, design D9); verify `node scripts/i18n-check.mjs` prints `ok (7082 messages, every shipped catalog complete)` before any translation is added

## 2. The web catalog

One fragment per source area, largest first, each merged and checked before the next (design D8). `node scripts/i18n-check.mjs` must print `zh: <n> of 7082 (work in progress, not offered)` with the expected `n` after every merge.

- [x] 2.1 App shell and core surfaces (`web/src/app/` root files: navigation, task row and its menus, views Today/Upcoming/Anytime/Someday, bulk actions, undo) — about 2885 messages, the bulk of the work
- [x] 2.2 Task detail panel (`app/task-detail/`, `app/notes/`, `app/bulk/`, `app/menus/`) — about 400
- [x] 2.3 Settings (`lib/settingsCopy.ts`, `app/settings*`, `app/display/`, `app/profile/`) — about 600
- [x] 2.4 Rules, review and method (`lib/rules/`, `app/review/`, `app/method/`, `lib/method/`, `app/rules/`) — about 700
- [x] 2.5 Routines, assistant, palette, filter, calendar, locations, themes, focus (`app/routines/`, `app/assistant/`, `lib/assistant/`, `app/palette/`, `app/filter/`, `lib/filter/`, `app/calendar/`, `app/locations/`, `app/themes/`, `lib/theme/`, `app/focus/`) — about 1300
- [x] 2.6 Errors and server messages (`lib/i18n/errors.ts`) — 136; every code keeps its meaning and the server's fallback still reads
- [x] 2.7 Long tail: shortcuts, activity, push texts, webhooks, colour names, fields, import/export, help links, page titles — the remainder
- [x] 2.8 Read the catalog once end to end for terms, register, punctuation and tone (design D6, D7); no entry left in English by accident

## 3. Service worker and browser extension

- [x] 3.1 `zh` in the notification table in `web/public/push-sw.js` (six strings); verify the table's test passes and `{date}` survives
- [x] 3.2 `extension/_locales/zh/messages.json` (57 keys, same placeholders, `extName` ≤ 75 and `extDescription` ≤ 132 characters); verify `node --test extension/locales.test.js`
- [x] 3.3 `extension/locales.test.js`: the shipped list gains `zh`; verify the same test

## 4. Phone shells and server text

- [x] 4.1 `mobile/android/app/src/main/res/values-zh/strings.xml` (49 strings, same `%s`/`%d` specifiers, `<xliff:g>` untouched); verify against `values/strings.xml` key by key
- [x] 4.2 `mobile/ios/Shared/Localizable.xcstrings`: a `zh` localization for all 30 keys, `state: "translated"`, same `%@`/`%lld` specifiers; verify with `scripts/i18n/shell.mjs`'s rules
- [x] 4.3 `internal/mail/templates/*.zh.txt` (twelve templates: digest, email change, invitation, member invitation, password reset, project invite, routine, test, verify signup ×2, workspace invite); verify placeholder parity with the `.en.txt` files
- [x] 4.4 `zh` in `mail.Languages` and a `zh` table in `internal/channels/messages.go` (25 messages, `{name}` kept); verify the Go tests that walk those tables

## 5. Registration

- [x] 5.1 One commit that flips every surface: `{ code: 'zh', name: '简体中文' }` in `SHIPPED`, and nothing else that changes behavior; verify `node scripts/i18n-check.mjs` (now strict) is silent and `node --test extension/locales.test.js` passes
- [x] 5.4 The language list that the app reads the preference through: `LanguagePref` in `web/src/lib/types.ts` and `LANGUAGES` in `web/src/lib/userPrefs.ts` both list the shipped codes. A code missing there is read as `system`, so the choice in Settings does not stick. Both now derive from `SHIPPED`, so a language that ships is always one the reader can keep. Verify: `web/src/lib/userPrefs.test.ts` reads every shipped code back.
- [x] 5.5 The plugin (`obsidian/src/strings.ts`) and the digest emails (`web/src/app/DigestCard.tsx`) have their own language lists; `zh` is in both, and the plugin's `zh-TW` now reads the Simplified catalogue (D1).
- [x] 5.2 `make i18n-check`, `make ui-lint`, `make check` where the toolchain is installed; `go test ./internal/mail/... ./internal/channels/... ./internal/quickadd/...`
- [x] 5.3 Bundle: the `zh` chunk stays out of the first load and `make bundle-budget` passes

## 6. Review

`review.md` beside this file is the maintainer's checklist: the gates below are open until a person has walked it.

- [x] 6.1 `docs/translating.md`, `docs/language.md` and the `localization` spec name the language and its maintainer
- [ ] 6.2 Look at the app in Chinese at 375 px and 1280 px, light and dark: line height of wrapped Chinese text, truncation in the sidebar and the task row, button widths, the Settings panel's language list
- [ ] 6.3 Say plainly what is not Chinese yet: quick add (design D5), the documentation, the in-app Help, the `ot` CLI
