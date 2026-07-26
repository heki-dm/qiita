---
name: qiita-cli
description: Use when the user wants to write, preview, publish, pull, or otherwise manage Qiita articles in this repository via the Qiita CLI (npx qiita ...). Covers login/auth setup, article creation, preview server, publishing, pulling remote changes, and troubleshooting common errors like missing credentials.
---

# Qiita CLI operations

This repo manages Qiita articles locally via `@qiita/qiita-cli` (installed as a devDependency). Articles live as markdown files in `public/`. All commands below are run from the repo root: `/Users/heki/develop/heki-dm/qiita`.

## First-time setup / re-auth

If any command fails with `ENOENT ... credentials.json`, the user has not logged in yet (or credentials were wiped). Fix:

1. Have the user issue a token at:
   https://qiita.com/settings/tokens/new?read_qiita=1&write_qiita=1&description=qiita-cli
   (must have both `read_qiita` and `write_qiita` scopes)
2. Run `npx qiita login` and paste the token when prompted (interactive — cannot be scripted/non-interactively piped by Claude).
3. This creates `~/.config/qiita-cli/credentials.json` (or under `$XDG_CONFIG_HOME/qiita-cli`).

Never ask the user for their token directly or try to write credentials.json yourself — always have them run `npx qiita login` interactively.

## Common commands

- `npx qiita preview` — starts local preview server (default http://localhost:8888, configurable via `qiita.config.json` host/port). Downloads existing Qiita articles on start.
- `npx qiita new <basename>` — creates `public/<basename>.md` with frontmatter template (title, tags, private, updated_at, id, etc).
- `npx qiita publish <basename>` — publishes/updates that one article on Qiita.
- `npx qiita publish --all` — publishes/updates all articles.
- `npx qiita publish <basename> --force` (or `-f`) — force overwrite remote with local content.
- `npx qiita pull` — syncs unmodified local files with remote Qiita state.
- `npx qiita pull --force` (or `-f`) — force overwrite local files with remote content.
- `npx qiita posting-campaigns` — lists active posting campaigns (UUIDs for frontmatter `posting_campaign_uuid`).
- `npx qiita version` — print CLI version.
- `npx qiita help` — help text.

Useful global options: `--credential <dir>`, `--config <dir>`, `--root <dir>`, `--verbose`.

## Article file structure

One markdown file per article under `public/`. Frontmatter fields: `title`, `tags` (list), `private` (bool), `updated_at` (auto-set on publish), `id` (auto-set UUID on first publish), `organization_url_name`, `slide` (bool), `ignorePublish` (bool — true skips this file on `publish --all`), `posting_campaign_uuid`, `agreed_posting_campaign_term`.

There is no CLI delete — removing the local `.md` file does NOT delete the article on Qiita. Deletion must be done on qiita.com directly.

## This repo's config

`qiita.config.json`: `includePrivate: false`, `host: localhost`, `port: 8888`. Private articles are only pulled if `includePrivate: true` here AND `private: true` in the article's frontmatter.

## Notes

- `login`, and anything needing a TTY prompt, cannot be run non-interactively by Claude — tell the user to run it themselves in their terminal.
- If errors persist after confirming login, check https://github.com/increments/qiita-discussions/discussions for known issues.
