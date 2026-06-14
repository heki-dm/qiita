---
title: 「npm for agent context」- Microsoft製の Agent Package Manager (APM) を試してみた
tags:
  - AI
  - apm
  - MCP
  - GitHubCopilot
  - ClaudeCode
private: false
updated_at: '2026-06-14T15:10:05+09:00'
id: 0fa51870b9b3da303ddf
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

Claude Code や GitHub Copilot、Cursor などの AI コーディングエージェントを使っていると、ツールごとに `CLAUDE.md`、`.github/copilot-instructions.md`、`.cursorrules` のように設定ファイルの形式が異なり、同じ「指示」「スキル」「MCP サーバー設定」をツールごとに作り直す必要があるという悩みがあります。

この課題に対して Microsoft が公開したのが **APM (Agent Package Manager)** です。「npm for agent context」と説明されており、実際にインストールして使ってみました。

公式サイト: <https://microsoft.github.io/apm/>
リポジトリ: <https://github.com/microsoft/apm>

## APM とは

APM は、AI エージェント向けの**コンテキスト（指示・スキル・プロンプトなど）を管理するための依存関係マネージャー**です。

npm / pip / cargo のような「マニフェスト + ロックファイル」方式を採用しており、1 つの `apm.yml` を書くだけで、Claude Code・GitHub Copilot・Cursor・Codex・Gemini・OpenCode・Windsurf など複数のハーネス（AI エージェント実行環境）向けに、それぞれが理解できるネイティブなファイル形式へ展開してくれます。

### 管理対象の 7 つのプリミティブ

APM が管理できるコンテキストは、以下の 7 種類に整理されています。

1. **Instructions** - プロジェクト全体のガイドライン
2. **Skills** - 再利用可能な手順・知識
3. **Prompts** - スラッシュコマンド
4. **Agents** - 専門的なペルソナ（サブエージェント）
5. **Hooks** - ライフサイクルハンドラ
6. **Commands** - カスタム CLI
7. **MCP servers** - 外部ツール統合

### 3 つの約束

- **可搬性**: 1 つのマニフェストで複数のハーネスに対応
- **セキュリティ**: 隠れた Unicode 文字のスキャンや、コンテンツハッシュによる固定
- **ガバナンス**: 組織のポリシーをインストール時に適用

APM 自体は AI エージェントのランタイムではなく、「各ツールが理解できるファイルを配置するだけ」という設計になっている点もポイントです。

## インストールしてみる

公式の手順に従い、インストールスクリプトを使ってバイナリをインストールします。

```bash
curl -sSL https://raw.githubusercontent.com/microsoft/apm/main/install.sh | sh
```

公式サイトでは短縮URL (`https://aka.ms/apm-unix`) が案内されていますが、実体は上記の GitHub 上のインストールスクリプトです。

実行すると、OS・アーキテクチャを自動判別して GitHub Releases から最新バイナリを取得し、`/usr/local/bin` に配置してくれます。

```text
+--------------------------------------------------------------+
|                         APM Installer                        |
|              The NPM for AI-Native Development               |
+--------------------------------------------------------------+

Detected platform: darwin-arm64
Target binary: apm-darwin-arm64.tar.gz
Fetching latest release information...
Latest version: v0.20.0
Download URL: https://github.com/microsoft/apm/releases/download/v0.20.0/apm-darwin-arm64.tar.gz
Downloading APM...
[+] Download successful
Extracting binary...
[+] Extraction successful
Testing binary...
[+] Binary test successful
Installing APM CLI to /usr/local/bin...
[+] APM installed successfully!
Version: Agent Package Manager (APM) CLI version 0.20.0 (43a00c2)
Location: /usr/local/bin/apm -> /usr/local/lib/apm/apm

Installation complete!

Quick start:
  apm init my-app          # Create a new APM project
  cd my-app && apm install # Install dependencies
  apm run                  # Run your first prompt
```

インストール後は以下のように動作確認できます。

```bash
$ apm --version
Agent Package Manager (APM) CLI version 0.20.0 (43a00c2)
```

なお、Homebrew (`brew install microsoft/apm/apm`) や pip (`pip install apm-cli`) でのインストールにも対応しています。

`apm --help` で主なコマンドを確認すると、npm に近い構成になっていることが分かります。

```text
Commands:
  audit         Scan installed packages for hidden Unicode characters
  compile       Compile APM context into distributed AGENTS.md files
  init          Initialize a new APM project
  install       Install APM and MCP dependencies
  lock          Resolve dependencies and write apm.lock.yaml
  mcp           Discover, inspect, and install MCP servers
  prune         Remove APM packages not listed in apm.yml
  publish       Publish a package to a registry.
  update        Refresh APM dependencies to the latest matching refs
  ...
```

## プロジェクトを初期化する

```bash
apm init myagent -y
```

実行すると、最小構成の `apm.yml` が生成されます。

```yaml
name: myagent
version: 1.0.0
description: APM project for myagent
author: heki
# Which agent platforms to deploy to (uncomment to pin):
# targets:
#   - copilot
#   - claude

dependencies:
  apm: []
  mcp: []
includes: auto
scripts: {}
```

`dependencies.apm` に他プロジェクトのパッケージ、`dependencies.mcp` に MCP サーバーを宣言していく形です。

## パッケージをインストールしてみる

公式サンプルパッケージをインストールしてみます。

```bash
cd myagent
apm install microsoft/apm-sample-package#v1.0.0
```

ここで最初に詰まったのが、ターゲットとなるハーネスを検出できないというエラーでした。

```text
[x] No harness detected

APM scanned for harness markers (.claude/, CLAUDE.md, .cursor/, .cursorrules,
.github/copilot-instructions.md, ...) but found none in this project.

Fix with one of:
  apm targets
  apm install <pkg> --target claude
  apm install <pkg> --target copilot
```

新規プロジェクトでは `.claude/` や `.github/copilot-instructions.md` のようなマーカーが存在しないため、どのハーネス向けにファイルを展開するかを明示する必要があるようです。今回は Claude Code 向けに `--target claude` を指定しました。

```bash
apm install --target claude
```

```text
[>] Installing dependencies from apm.yml...
[i] Targets: claude  (source: --target flag)
  [+] microsoft/apm-sample-package #v1.0.0 @fb285168
  |-- 1 agents integrated -> .claude/agents/
  |-- 2 commands integrated -> .claude/commands/
  |-- 1 rule(s) integrated -> .claude/rules/
  |-- 1 skill(s) integrated -> .claude/skills/
  [+] github.com/github/awesome-copilot/skills/review-and-refactor #default @a1fc7f1a
  |-- Skill integrated -> .claude/skills/
[i] Added apm_modules/ to .gitignore
[*] Installed 2 APM dependencies in 10.0s.
```

指定したパッケージが内部で参照していた `github/awesome-copilot` のスキルまで、依存関係としてまとめて解決・インストールされているのが分かります。

生成されたファイルは以下の通りです。

```text
.claude/agents/design-reviewer.md
.claude/commands/accessibility-audit.md
.claude/commands/design-review.md
.claude/rules/design-standards.md
.claude/skills/review-and-refactor/SKILL.md
.claude/skills/style-checker/SKILL.md
apm.lock.yaml
apm.yml
apm_modules/  (パッケージのソースをキャッシュ)
```

`.claude/commands/design-review.md` の中身を見ると、Claude Code のスラッシュコマンドとしてそのまま使える Markdown が展開されていました。

```markdown
---
description: Review code for design system compliance and visual consistency
---

# Design Review

Review the current codebase for design system compliance. Check for:

1. **Color Usage** — Are colors from the design token palette?
2. **Typography** — Do headings and body text follow the typography scale?
...
```

インストール後、`apm.yml` には依存パッケージが追記され、`apm.lock.yaml` には解決された commit hash やファイルごとのハッシュが記録されます。

```yaml
dependencies:
  apm:
  - microsoft/apm-sample-package#v1.0.0
  mcp: []
```

```yaml
lockfile_version: '1'
generated_at: '2026-06-13T15:48:29.410048+00:00'
apm_version: 0.20.0
dependencies:
- repo_url: microsoft/apm-sample-package
  resolved_commit: fb2851683be0e0e7711421d518bd8dba23b0b1f6
  resolved_ref: v1.0.0
  deployed_files:
  - .claude/agents/design-reviewer.md
  - .claude/commands/accessibility-audit.md
  ...
  content_hash: sha256:744cca54cc8ff7ca90aa1dd621c2f98c6291cd793815afe8518001cc94b8aba9
```

この `apm.lock.yaml` をチームで共有すれば、`apm install` を実行した全員が**バイト単位で同一のコンテキスト**を再現できる、というのが APM の売りのようです。

## audit でドリフトを検出する

`apm audit` を実行すると、インストール済みパッケージとロックファイルの内容に差分（ドリフト）がないかチェックできます。

```bash
$ apm audit
[>] Scanning all installed packages...
[>] Replaying install (cache-only)...
[+] Replayed 2 package(s)
[>] Diffing scratch vs working tree...
[+] No drift detected

[*] 8 file(s) scanned -- no issues found
```

CI で `apm audit --ci` を実行すれば、誰かが展開後のファイルを手で書き換えてしまった、といった事故を検知できそうです。

## 対応ハーネス一覧

`apm targets` で、現在のプロジェクトがどのハーネス向けに展開されているか/対応可能かを確認できます。

```text
  TARGET       STATUS     SOURCE                                   DEPLOY DIR
  ------------ ---------- ---------------------------------------- ----------
  claude       active     .claude/                                 .claude/
  copilot      inactive   needs .github/copilot-instructions.md    .github/
  cursor       inactive   needs .cursor/                           .cursor/
  codex        inactive   needs .codex/                            .codex/
  gemini       inactive   needs GEMINI.md                          .gemini/
  opencode     inactive   needs .opencode/                         .opencode/
  windsurf     inactive   needs .windsurf/                         .windsurf/
  kiro         inactive   needs .kiro/                             .kiro/
```

主要なエージェント実行環境がほぼ網羅されており、`apm install --target <name>` を切り替えるだけで同じ依存関係を別のハーネス向けに展開できます。

## apm.yml をどう設計するか

ここまでは公式のサンプルパッケージを使った動作確認でしたが、実際に自分のパッケージを作る場合、`apm.yml` をどう設計すればよいのか整理してみます。

### パッケージのレイアウトは3種類

APM が認識するパッケージレイアウトは大きく3つあります。

1. **単一スキル**: リポジトリのルートに `SKILL.md` を置くだけのシンプルな構成。`agents/` や `scripts/` を同居させることも可能で、依存関係管理が必要なら `apm.yml` を追加した HYBRID パッケージにできます。
2. **複数プリミティブ**: `.apm/` 配下に `skills/` `agents/` `instructions/` などのサブディレクトリを切り、複数のプリミティブをまとめて管理する構成。`apm install` 時にそれぞれが個別に展開されます。
3. **Claude プラグイン**: 既に `plugin.json` を持つリポジトリなら、構成変更なしでそのまま APM パッケージとして扱えます。

「1つのまとまった機能を配るだけ」なら単一スキル、「複数の関連するスキル・エージェントをセットで配りたい」なら `.apm/` 構成、というのが基本的な判断基準になりそうです。

### `includes: auto` の役割

`apm init` で生成される `apm.yml` には `includes: auto` という項目があります。これは「`.apm/` 配下にあるものを全部デプロイ対象にする」という設定で、これがないとローカルのプリミティブは一切配布されません。

```yaml
includes: auto
```

公開するプリミティブを限定したい場合は、`auto` の代わりに明示的なパスのリストを指定することもできます。「社内向けの実験的なスキルは含めず、完成したものだけ配る」といった調整がここでできそうです。

### 依存関係の書き方

`dependencies.apm` には、他リポジトリ全体を依存として指定する方法と、リポジトリ内の特定のプリミティブだけを指定する方法の2通りがあります。

```yaml
dependencies:
  apm:
    - microsoft/apm-sample-package#v1.0.0          # リポジトリ全体をタグ指定で依存
    - github/awesome-copilot/skills/review-and-refactor  # 1つのスキルだけを依存
  mcp:
    - microsoft/azure-devops-mcp                    # MCP サーバーも依存関係として管理
```

また `devDependencies` を使うと、配布物には含めたくない開発・テスト用の依存（社内テストスキルなど）を分離できます。`apm pack` でパッケージ化する際に自動的に除外される点が、npm の `devDependencies` と同じ感覚で扱えそうです。

### ターゲットの決め方

`apm install` がどのハーネス向けに展開するかは、以下の優先順位で決まります。

1. `--target` フラグ
2. `apm.yml` の `targets:` フィールド
3. ファイルシステム上の自動検出(`.claude/`、`.cursor/`、`.github/copilot-instructions.md` など)

複数人で使うパッケージの場合、環境によって検出結果がブレるのを避けるために `apm.yml` に `targets:` を明示しておくのが安全そうです。

```yaml
targets:
  - claude
  - copilot
```

なお、スキルの展開先はハーネスによって差があります。Claude Code は `.claude/skills/` に展開されますが、Copilot・Cursor・OpenCode・Codex・Gemini は共通の `.agents/skills/` を読みに行くため、1回の `apm install` で複数ハーネスに同じスキルを行き渡らせることができます。一方でエージェント定義(`agents/`)はハーネスごとに専用ディレクトリ(`.github/agents/` など)に展開される、という違いがある点は覚えておくとよさそうです。

### 実際に作ってみた構成例

別記事で紹介した [zensical-skills](https://github.com/techolve/zensical-skills) は、上記の「複数プリミティブ」レイアウトの実例です。

```yaml
name: zensical-skills
version: 1.0.0
description: APM project for zensical-skills
author: Techolve
dependencies:
  apm: []
  mcp: []
includes: auto
scripts: {}
```

- Zensical の公式ドキュメントをページ単位で `SKILL.md` 化し、`.apm/skills/` 配下に48個並べる
- ドキュメントサイト編集に特化したサブエージェントを `.apm/agents/` に1つ追加
- 外部依存は持たないため `dependencies` は空、`includes: auto` で `.apm/` 配下を丸ごと配布

利用側のリポジトリでは、これを依存として1行追加するだけで済みます。

```yaml
dependencies:
  apm:
    - techolve/zensical-skills
```

「ツール固有の公式ドキュメントを Skill 化して配布する」というパターンは、`includes: auto` + 外部依存なしというシンプルな `apm.yml` で十分実現できることが分かりました。逆に、複数チームの規約やレビュー基準を1つのパッケージにまとめたい場合は、`devDependencies` で社内検証用のテストスキルを分けたり、`targets` でハーネスを固定したりといった調整が必要になりそうです。

## まとめ

実際に触ってみた所感です。

- セットアップは npm 経験者であれば直感的（`init` → `apm.yml` 編集 → `install`）
- パッケージは GitHub リポジトリをそのまま参照できるため、社内のプライベートリポジトリに「スキル集」を置いて社内全体で共有する、という使い方がしやすそう
- 依存関係の中に MCP サーバーも含められるため、エージェント向けのセットアップを丸ごと「1 つの依存ツリー」として扱える
- 新規プロジェクトでは `--target` の明示が必要だったり、ロックファイルの仕組みなど、「インフラとしての落ち着き」を感じる設計
- `apm.yml` のレイアウトは「単一スキル」「複数プリミティブ」「Claude プラグイン」の3択で、配りたいものの粒度に応じて選べばよさそう

複数の AI コーディングツールを併用しているチームで、エージェント向けの指示・スキル・MCP 設定を一元管理したい場合に、検討する価値のあるツールだと感じました。

## 参考

- [APM 公式サイト](https://microsoft.github.io/apm/)
- [What is APM? - Agent Package Manager](https://microsoft.github.io/apm/concepts/what-is-apm/)
- [Quickstart - Agent Package Manager](https://microsoft.github.io/apm/quickstart/)
- [Anatomy of an APM Package](https://microsoft.github.io/apm/concepts/package-anatomy/)
- [Your First Package - Agent Package Manager](https://microsoft.github.io/apm/getting-started/first-package/)
- [GitHub - microsoft/apm](https://github.com/microsoft/apm)
- [techolve/zensical-skills](https://github.com/techolve/zensical-skills)
