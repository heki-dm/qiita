---
title: Material for MkDocsチームが作る新しい静的サイトジェネレーター「Zensical」を試してみた
tags:
  - MkDocs
  - Zensical
  - StaticSiteGenerator
  - Documentation
  - Rust
private: false
updated_at: "2026-06-13T12:00:00+09:00"
id: ""
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

ドキュメント作成ツールとして Material for MkDocs を愛用している方は多いと思いますが、その開発チームが新しい静的サイトジェネレーター「**Zensical**」を公開しました。

公式サイト: <https://zensical.org/>

今回は実際にインストールしてプロジェクトを作成・ビルドし、どんなものなのか触ってみたので、その内容をまとめます。

## Zensical とは

Zensical は、**Material for MkDocs を開発しているチームが、10 年間の経験を踏まえてゼロから作り直した**技術文書向けの静的サイトジェネレーターです。

Rust と Python で実装されており、Python パッケージとして配布されています。

## なぜ新しく作られたのか

背景には MkDocs 自体が抱える課題があります。

- MkDocs は 2024 年 8 月以降メンテナンスが止まっている
- 未解決の Issue や PR が蓄積し続けている
- サードパーティ依存によるサプライチェーンリスクになっている

こうした構造的な問題を解決するため、Zensical では著作体験（AX）・開発者体験（DX）・ユーザー体験（UX）をすべて自社でまとめて持つ「垂直統合」のアプローチが採られています。

また、`ZRX` という差分ビルドエンジンにより、再ビルド時に **4〜5 倍の高速化**が実現されているとのことです。これは後述するハンズオンでも体感できました。

## 主な特徴

- OSS として公開されている
- 既存の `mkdocs.yml` をそのまま読み込める
- Python Markdown の拡張機能はそのまま利用可能
- ファイル構成・URL・アンカーが変わらないので、既存サイトの SEO やブックマークを維持できる
- インスタントナビゲーションやコードコピー、検索ハイライトなど、Material for MkDocs でよく使われる機能の多くが標準で有効になっている

なお、エンタープライズ向けには「Zensical Spark」という有料プランも用意されているようです（詳細は公式サイトを参照してください）。

## インストールしてみる

`uv` を使うと簡単にセットアップできます。

```bash
uv init
uv add --dev zensical
```

実行すると、依存パッケージとして以下がインストールされました。

```text
+ click==8.4.1
+ deepmerge==2.0
+ jinja2==3.1.6
+ markdown==3.10.2
+ markupsafe==3.0.3
+ pygments==2.20.0
+ pymdownx==10.21.3
+ pyyaml==6.0.3
+ tomli==2.4.1
+ zensical==0.0.45
```

CLI のヘルプを見ると、コマンドは非常にシンプルです。

```bash
$ uv run zensical --help
Usage: zensical [OPTIONS] COMMAND [ARGS]...

  Zensical - A modern static site generator.

Options:
  --version  Show the version and exit.
  --help     Show this message and exit.

Commands:
  build  Build a project.
  new    Create a new template project in the current or given directory.
  serve  Build and serve a project.
```

## プロジェクトを作成する

`zensical new` でテンプレートプロジェクトを作成できます。

```bash
uv run zensical new mysite
```

生成されたファイルは以下の通りでした。

```text
mysite/zensical.toml
mysite/docs/markdown.md
mysite/docs/index.md
mysite/.github/workflows/docs.yml
```

`mkdocs.yml` の代わりに `zensical.toml` という設定ファイルが使われますが、内容はテーマ・カラーパレット・Markdown 拡張など、Material for MkDocs の `mkdocs.yml` を触ったことがある人には馴染みのある項目ばかりでした。コメントによるドキュメントも非常に充実しています。

## ビルドしてみる

```bash
cd mysite
uv run zensical build
```

結果は以下のようになりました。

```text
Build started
No issues found
Build finished in 0.68s
```

`site/` 配下に `index.html` や `search.json`、`sitemap.xml` などが出力され、静的サイトとして配信できる状態になります。

続けてもう一度ビルドしてみると、差分ビルドエンジン ZRX の効果か、明確に高速化されました。

```text
Build started
No issues found
Build finished in 0.20s
```

ファイル数が少ない簡易プロジェクトとはいえ、変更なしの再ビルドが 0.68 秒 → 0.20 秒と短縮されており、宣伝されている「再ビルド時の高速化」を実感できました。

## Material for MkDocs との互換性

公式の互換性ページによると、以下の領域でシームレスな互換性が確保されています。

- **ビルド設定**: 既存の `mkdocs.yml` をそのまま利用可能
- **コンテンツと前処理**: Python Markdown とその拡張機能がそのまま動作
- **プロジェクト構造と URL**: ファイル配置・URL・アンカーが変わらず、SEO やブックマークが保持される
- **テンプレートと CSS/JS**: MiniJinja への対応に伴う軽微な調整のみで、HTML 構造と CSS 変数は互換

移行は 4 つのフェーズで段階的に進められる予定で、フェーズ 1 では「最大限の互換性」、フェーズ 3 で Material for MkDocs や主要プラグインとの機能パリティを目指しているとのことです。

## 公式ドキュメントをAIエージェント向けスキルにしてみた

Zensical の公式ドキュメントは前述の通りコメントが充実していて読みやすいのですが、AI コーディングエージェントに「Zensical のドキュメントサイトを編集して」と頼んだ場合、エージェントが Zensical 固有の構文や設定方法を知らないと、MkDocs 時代の古い情報を参照してしまったり、的外れな提案をしてしまうことがあります。

そこで、Zensical の公式ドキュメントを章ごとに **Agent Skill** として切り出した [zensical-skills](https://github.com/techolve/zensical-skills) というリポジトリを作成しました。

### 構成

公式ドキュメントのカテゴリ(Get Started / Usage / Setup / Extensions / Authoring など)に対応する形で、ページ単位で Skill を分割しています。

```text
.apm/
├── agents/
│   └── zensical.agent.md
└── skills/
    ├── zensical-docs-get-started/
    │   └── SKILL.md
    ├── zensical-docs-usage-cli/
    │   └── SKILL.md
    ├── zensical-docs-setup-navigation/
    │   └── SKILL.md
    ├── zensical-docs-authoring-admonitions/
    │   └── SKILL.md
    ...（全48スキル）
```

各 `SKILL.md` には、対応する公式ドキュメントの URL、対象とする使用シーン、カバーすべき内容、サンプルコードがまとめられています。例えば `zensical-docs-get-started` の場合は以下のような内容です。

```markdown
---
name: zensical-docs-get-started
description: Summarize and apply Zensical get-started guidance for installation and initial environment setup.
---

# Zensical Docs: Get started

Use this skill when the user asks how to install Zensical, choose an installation method, or prepare a local environment.

## Scope

This skill is based on:

- https://zensical.org/docs/get-started/

## What to cover
...
```

加えて、Zensical ドキュメントサイトの編集に特化した `zensical.agent.md` というサブエージェント定義も用意し、「Zensical 関連の質問にのみ答える」「ユーザーのプロジェクト外のファイルは編集しない」といった制約を持たせています。

### APM パッケージとして配布する

別記事で紹介した [APM (Agent Package Manager)](https://microsoft.github.io/apm/) の仕組みに乗せて、このスキル集を 1 つの APM パッケージとして公開しました。利用する側は、自分の `apm.yml` に依存として追加するだけで導入できます。

```yaml
dependencies:
  apm:
  - techolve/zensical-skills
```

```bash
apm install --target claude
```

これを実行すると、48個のスキルとエージェント定義が `.claude/skills/` 配下に展開され、Claude Code が Zensical ドキュメントの編集時にこれらのスキルを参照できるようになります。

実際に Zensical で構築している社内ドキュメントサイトのリポジトリに導入したところ、ナビゲーションや拡張機能の設定変更を依頼した際に、`zensical.toml` の正しいキー名や Zensical 固有の構文を踏まえた提案が返ってくるようになりました。公式ドキュメントが整理されている分、Skill 化もしやすく、ツール固有のドキュメントを AI エージェントの知識として配布する一つのパターンとして手応えを感じています。

## まとめ

実際に触ってみた所感としては、

- `uv` でのセットアップが非常にスムーズ
- 生成される `zensical.toml` がコメント付きで分かりやすく、Material for MkDocs の知識がそのまま活きる
- 簡易プロジェクトでも再ビルドの高速化を体感できた
- 公式ドキュメントが整理されている分、AI エージェント向けの Skill としても切り出しやすい

という印象でした。MkDocs のメンテナンスが停滞している中、既存の Material for MkDocs プロジェクトからの移行先として、今後注目していきたいツールです。

## 参考

- [Zensical 公式サイト](https://zensical.org/)
- [Get started - Zensical Documentation](https://zensical.org/docs/get-started/)
- [Compatibility - Zensical](https://zensical.org/compatibility/)
- [Zensical - A modern static site generator - Material for MkDocs Blog](https://squidfunk.github.io/mkdocs-material/blog/2025/11/05/zensical/)
- [techolve/zensical-skills](https://github.com/techolve/zensical-skills)
