## MODIFIED Requirements

### Requirement: Shipped languages and their catalogs
English, German, French, Spanish, Italian, Portuguese and Simplified Chinese SHALL be maintained in the repository and SHALL be complete in every release. A language SHALL be offered in Settings only when its catalog is complete. Shipped catalogs SHALL be written and reviewed by people; machine translation SHALL NOT be used for a shipped catalog. A community language follows the same rule. A shipped language SHALL be translated on every surface that carries its text: the web catalog, the service worker's notification texts, the browser extension, the Android and iOS shells, the server's emails and the notification channel messages.

#### Scenario: Complete languages offered
- **WHEN** the user opens the Language panel
- **THEN** the seven shipped languages are listed, each in its own name

#### Scenario: Work-in-progress catalog
- **WHEN** a catalog in the repository is incomplete
- **THEN** it is not offered in Settings and the release check reports its missing messages

#### Scenario: A Chinese browser
- **WHEN** a user whose browser prefers `zh-CN` signs in for the first time
- **THEN** the interface is shown in Simplified Chinese and the panel's System entry names it

#### Scenario: A missing entry fails the build
- **WHEN** a change adds a message without its Chinese entry
- **THEN** `make i18n-check` fails and names the message

#### Scenario: A surface left behind
- **WHEN** the web catalog lists Chinese and the iOS shell's catalog has no Chinese localization
- **THEN** `make i18n-check` fails and names the shell key

### Requirement: Plural and number forms follow the language
Messages with a count SHALL use the active language's plural categories, and counts SHALL be written with the language's digit grouping. A language whose plural system has a single category SHALL use that category alone, not the source's `one` branch. Dates follow `date-time-preferences`.

#### Scenario: Singular and plural
- **WHEN** the language is German and a proposal has 1 change, then 2 changes
- **THEN** the texts read `1 Änderung` and `2 Änderungen`

#### Scenario: Grouped count
- **WHEN** the language is French and a search placeholder names 10000 tasks
- **THEN** the number is written as `10 000`

#### Scenario: One category
- **WHEN** the language is Simplified Chinese and a proposal has 1 change, then 2 changes
- **THEN** both texts use the single `other` form (`1 处更改`, `2 处更改`) and no `one` form exists in the catalog

#### Scenario: A `one` form in a one-category language
- **WHEN** a Chinese translation of a plural message uses `one`
- **THEN** `make i18n-check` fails and names the message

#### Scenario: Grouped count in Chinese
- **WHEN** the language is Simplified Chinese and a search placeholder names 10000 tasks
- **THEN** the number is written as `10,000`
