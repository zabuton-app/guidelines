# 10 Writing Language

## Principle

**Every artifact that remains in a repository is written in English.** The zabuton series is published as OSS and is expected to be distributed through three channels (GitHub Release / AUR / Microsoft Store), so everything is unified in a language that developers visiting the repositories can read.

This covers all of the following.

- Code comments, identifiers, and log output
- READMEs, documentation under `docs/`, and landing pages
- Commit messages
- GitHub issues and pull requests (titles, bodies, and review comments)
- CI configuration and comments in shell scripts

Conversation with the user (the developer themself) can stay in Japanese as before. Only what remains in the repository is subject to being written in English.

## Exceptions (Written in Japanese)

| Target | Reason |
| ------ | ------ |
| The single kanji of a display name (巡, 刻, etc.) | Brand convention. See [00 Brand and Naming](00-brand.md) |
| The Japanese i18n catalog (`locales/ja.ts` / `i18n/ja.ts`) | The Japanese catalog is the source of truth. See [07 i18n](07-i18n.md) |
| The "日本語" label in the language select | The `label` in `LANGUAGES` uses each language's own name for itself |
| The Japanese translation of these guidelines (`ja/`) | A translation kept in separate files. The English version is the source of truth |
| Each app's `CLAUDE.md` | A local operational file that is not committed |

## When in Doubt

- **UI strings always go through the i18n catalog**. Do not write Japanese literals directly in the source (the kanji of the display name is the only exception)
- **English is the source of truth for public documentation**. If a Japanese version becomes necessary, split it into a separate file and treat the English version as authoritative
- **Temporary files such as review notes, work logs, and research notes do not belong in the repository in the first place**. The moment you want to write something in Japanese, that is a sign the file should live outside the repository

## Pre-Release Check

The following command finds Japanese text that has slipped into tracked files. Confirm that every hit falls within the exception table above. `-I` is required to exclude binaries (fonts, PNGs, etc. that happen to contain these byte sequences).

```bash
git ls-files -z | xargs -0 grep -IlP '[\x{3040}-\x{30ff}\x{4e00}-\x{9fff}]'
```
