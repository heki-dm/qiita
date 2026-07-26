---
title: AI時代のドキュメントツール「Blume」を触ってみた
tags:
  - astro
  - documentation
  - StaticSiteGenerator
  - AI
  - Blume
private: false
updated_at: '2026-07-26T10:29:37+09:00'
id: 9a216a79e5ad3c0f06e7
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## はじめに

先日、Material for MkDocs チームが作る新しい静的サイトジェネレーター「Zensical」を触ってみた記事を書きました。

- [Material for MkDocsチームが作る新しい静的サイトジェネレーター「Zensical」を試してみた](https://qiita.com/heki-dm/items/a7e37c13a0b677709851)

その後もドキュメントツールを追っていたところ、今度は「**Blume**」という、AI エージェント連携を前面に押し出したドキュメントジェネレーターを見つけました。

公式サイト: <https://useblume.dev/>

Zensical が「Material for MkDocs の資産を引き継ぎつつ高速化する」方向性だったのに対し、Blume は Astro / Vite ベースでゼロから作られており、コンセプトも毛色が違います。実際にセットアップしてビルドまで試してみたので、その内容をまとめます。

## Blume とは

Blume は、"**fast, AI-ready, markdown-first docs**" を謳うドキュメントサイトジェネレーターです。フォルダに Markdown を置くだけで、本番品質のドキュメントサイトを構築できます。

- 完全無料・オープンソース(MIT ライセンス)
- ゼロコンフィグで開始可能(ナビゲーションは自動推論)
- Astro / Vite 基盤で高速動作
- Node / Bun / Deno に対応

個人プロジェクトの小さな README サイトから、数千ページ規模の API ドキュメントまでスケールする設計とのことです。

### 一番の特徴は「AI-ready」であること

Blume を触っていて最も印象的だったのは、生成されたサイトが最初から AI エージェントや LLM に読まれることを前提に作られている点です。

- 各ページの Markdown ソースをそのまま `/{route}.md` として配信(コンテンツネゴシエーション)
- `llms.txt` / `llms-full.txt` を自動生成(サイト全体の機械可読インデックス)
- `agent-readability.json` によって、AI クローラーに「このサイトはどう読めばいいか」を明示
- Vercel AI Gateway や OpenRouter 経由で、ページ内 AI アシスタントを組み込み可能
- Model Context Protocol(MCP)サーバーを標準搭載

つまり「人間が読むサイト」と「AI エージェントが読み込む知識源」を、同じ Markdown ソースから同時に作る、という設計思想になっています。

### そのほかの機能

- Mermaid 図、KaTeX 数式などの Markdown 拡張
- ダークモード、フルテキスト検索、OGP 画像、SEO、36 言語のローカライズ、RTL 言語対応
- サイトマップ・RSS・JSON-LD の自動生成
- 30 種類以上のアクセシブルな MDX コンポーネント(Card、Steps、Tabs、CodeGroup、Diff など)をインポートなしで利用可能
- OpenAPI / AsyncAPI 仕様を Scalar で統合した API リファレンス生成
- ファイルシステムだけでなく GitHub、Sanity、Notion などをコンテンツソースとして接続可能(カスタムバックエンドアダプタにも対応)

## インストールしてみる

`npx blume init` でプロジェクトを初期化できます。

```bash
npx blume init . --yes
```

出力は以下の通りでした。

```text
[blume] ✔ Created docs/index.mdx
[blume] ✔ Created package.json
[blume] ✔ Created blume.config.ts
[blume] ✔ Added .blume/, dist/ to .gitignore

 ╭─────────────────╮
 │    npm install  │
 │    npm run dev  │
 ╰─────────────────╯
```

生成されたファイルはシンプルで、以下の3つだけです。

```text
docs/index.mdx
package.json
blume.config.ts
```

### 設定ファイル

`blume.config.ts` は TypeScript で書かれており、`defineConfig` に型補完が効きます。

```typescript
import { defineConfig } from "blume";

export default defineConfig({
  title: "My Docs",
  description: "Documentation powered by Blume.",
});
```

コンテンツソースの切り替えや、AI アシスタント用のモデル設定などもこのファイルに追記していく形のようです。

### 生成された Markdown

`docs/index.mdx` には最初からフロントマターが入っています。

```mdx
---
title: Introduction
description: Welcome to your new Blume docs.
---

# Introduction

Welcome to **Blume** — markdown-first docs powered by Astro and Vite.

Edit `docs/index.mdx` to get started, then run `blume dev`.
```

## ビルドしてみる

`npm install` の後、`npm run build`(内部的には `blume build`)を実行します。

```bash
npm install
npm run build
```

ビルドログの一部です。

```text
[blume] ◐ Building 1 page(s) (static output)
09:28:09 [content] Synced content
09:28:09 [build] output: "static"
09:28:09 [build] mode: "static"
...
 generating static routes 
   ├─ /404.html
   ├─ /blume-search.json
   ├─ /index.md
   ├─ /index.mdx
   ├─ /index.html
 ✓ Completed in 86ms.

[blume] ✔ Generated llms.txt and llms-full.txt
[blume] ✔ Generated robots.txt
[blume] ✔ Generated agent-readability.json
[blume] ✔ Emitted _headers (UTF-8 Content-Type for raw endpoints)

 ╭───────────────────────────────────────╮
 │  Output     static                    │
 │  Adapter    none                      │
 │  Search     orama                     │
 │  Robots     yes                       │
 │  Agent JSON yes                       │
 │  LLM files  yes                       │
 ╰───────────────────────────────────────╯

[blume] ✔ Built to /path/to/dist
```

たった1ページのプロジェクトでも、`index.html` だけでなく `index.md`(生 Markdown)、`llms.txt`、`agent-readability.json` までまとめて出力されるのが、Zensical など他の SSG との明確な違いだと感じました。

### 生成された AI 向けファイルの中身

`dist/llms.txt` はサイト全体を要約したインデックスになっていました。

```text
# My Docs

> Documentation powered by Blume.

## Docs

- [Introduction](/): Welcome to your new Blume docs.
```

`dist/index.md` には、ページの生 Markdown ソースがそのまま出力されています(HTML ページと同じ内容を `.md` として別途配信する形)。

```markdown
---
title: Introduction
description: Welcome to your new Blume docs.
---

# Introduction

Welcome to **Blume** — markdown-first docs powered by Astro and Vite.

Edit `docs/index.mdx` to get started, then run `blume dev`.
```

`dist/agent-readability.json` は、AI クローラー向けにサイトの構造を宣言する JSON でした。

```json
{
  "artifacts": {
    "markdown": {
      "contentNegotiation": "text/markdown",
      "pattern": "/{route}.md"
    },
    "llmsFullTxt": "/llms-full.txt",
    "llmsTxt": "/llms.txt"
  },
  "description": "Documentation powered by Blume.",
  "generator": "blume@1.1.4",
  "name": "My Docs",
  "site": null,
  "contentUsage": {
    "search": true,
    "ai-input": true,
    "ai-train": true
  }
}
```

`contentUsage` で `ai-input` や `ai-train` の可否をサイト側が明示できるようになっているのも、AI クロール前提のツールらしい設計だと思いました。

## MCP サーバーも標準搭載している

Blume で個人的に一番注目したのが、**MCP(Model Context Protocol)サーバーをドキュメントサイト自体が持てる**という点です。Claude Code のようなコーディングエージェントが、ドキュメントを検索・取得するためのツールを直接呼び出せるようになります。

公式ドキュメント: <https://useblume.dev/docs/configuration/ai>

`blume.config.ts` に以下のように書くだけで有効化できます。

```typescript
import { defineConfig } from "blume";

export default defineConfig({
  title: "My Docs",
  ai: {
    mcp: {
      enabled: true,
      route: "/mcp",
    },
  },
});
```

設定項目は以下の通りです。

| オプション | 説明 | デフォルト |
| --- | --- | --- |
| `enabled` | MCP サーバー生成の有効化 | `false` |
| `route` | エンドポイントのマウント先 | `/mcp` |
| `name` | クライアントに表示される名前 | サイトタイトル |
| `instructions` | エージェント向けのシステムヒント(任意) | - |

有効化すると、`search_docs` / `get_page` / `list_pages` / `get_navigation` といったツールが MCP クライアント側から呼び出せるようになるとのことです。ただし MCP サーバーはサーバーサイドの機能なので、静的出力ではなく `deployment.output: "server"` とアダプタの指定が必要です。

```typescript
export default defineConfig({
  deployment: {
    output: "server",
    adapter: "vercel",
    site: "https://docs.example.com",
  },
});
```

似た仕組みとして、ページ内チャットで読者の質問に答える「Ask AI」アシスタントも用意されています。こちらは Vercel AI Gateway(デフォルト)、OpenRouter、Inkeep、あるいは OpenAI 互換エンドポイントをバックエンドとして選べます。

```typescript
ai: {
  ask: {
    enabled: true,
    provider: "openrouter",
    model: "anthropic/claude-sonnet-4-5",
  },
}
```

「AI にサイトを検索させる導線(MCP)」と「サイト上で AI に質問できる導線(Ask AI)」の両方を、同じ `ai` 設定ブロックで賄えるようになっているのが Blume らしいところだと感じました。

### `blume doctor` で診断

サイトの健全性をチェックする `doctor` コマンドも用意されています。

```bash
$ npm run doctor

[blume] ℹ Pages: 1
[blume] ℹ Output: static
[blume] ℹ Search: orama
[blume] ✔ No problems found.
```

## Zensical との比較で感じたこと

同じ「Markdown からドキュメントサイトを作る」ツールでも、Zensical と Blume では狙っている方向性がかなり異なると感じました。

| 観点 | Zensical | Blume |
| --- | --- | --- |
| 基盤 | Rust + Python | Astro + Vite(TypeScript) |
| 立ち位置 | Material for MkDocs の後継・高速リビルド | ゼロから作られた AI-ready ドキュメントツール |
| 設定 | `zensical.toml`(mkdocs.yml 互換) | `blume.config.ts`(型付き TS) |
| AI 対応 | 特になし | llms.txt / MCP サーバー / ページ内アシスタントを標準搭載 |
| 移行のしやすさ | 既存 MkDocs プロジェクトからの移行に強い | 新規プロジェクト向け、コンポーネントも豊富 |

既存の MkDocs 資産を活かしたいなら Zensical、これから新規にドキュメントサイトを立ち上げつつ AI エージェントからの参照も見据えるなら Blume、という住み分けになりそうです。

### 試しに Zensical のページを Blume に持っていってみた

せっかく直前に Zensical を触っていたので、`zensical new` で生成された Markdown をそのまま Blume の `docs/` に置いてビルドできるか試してみました。

プレーンな Markdown だけで書かれたページ(見出し・リスト・表・コードブロックなど)は、フロントマターを `title` に直すだけでそのままビルドが通り、ルーティングも自動で `/markdown/` として認識されました。標準的な Markdown の範囲であれば、ファイルを移してくるだけで十分だという印象です。

一方で、`!!! note` のような admonition や `=== "Python"` のコンテンツタブといった **MkDocs(PyMdown Extensions)固有の記法**は、Blume 側では素直にビルドが通りませんでした。Blume は MDX ベースなので、これらは `<Note>` や `<Tabs>` のような MDX コンポーネントに書き換える必要があります。

```text
 ERROR  Build failed with 1 error:
[plugin @mdx-js/rolldown] .../features-test.mdx
MDXError: Could not parse expression with oxc: Expected , or ) but found : (mdx-jsx:unexpected-character)
```

admonition だけならまだしも、コンテンツタブ・脚注・数式・アイコン記法までページ全体にちりばめられていると、1ページずつ手で書き換えるのはそれなりに骨が折れそうです。ページ数が多い実サイトを丸ごと移行するとなると、地道な変換作業になることは覚悟しておいた方がよさそうです。

とはいえ、変換ルール自体は「`!!! note` は `<Note>`」「`=== "Python"` は `<Tabs>` + `<Tab>`」のように機械的に定義できるものです。前回の Zensical の記事で作った [zensical-skills](https://github.com/techolve/zensical-skills) のように、**Blume 側の MDX コンポーネント仕様を Agent Skill として与えておけば、AI コーディングエージェントに変換作業自体を任せられる**はずです。「MkDocs 記法 → Blume の MDX コンポーネント」という対応表さえ Skill 化してしまえば、大量のページがあっても人手で1つずつ書き換える必要はなくなりそうだと感じました。

## まとめ

実際に触ってみた所感としては、

- `npx blume init` から `npm run build` まで、数分でセットアップが完了する手軽さ
- 1ページだけの最小構成でも `llms.txt` や `agent-readability.json` が自動生成される、AI 前提の設計思想
- `blume.config.ts` が TypeScript で書けるため、エディタの型補完がそのまま効く
- `blume doctor` のようなヘルスチェックコマンドが最初から用意されている
- MkDocs 系ツールからの移行は、プレーンな Markdown はそのまま流用できるものの、admonition やタブなどの独自記法はページ数が多いと地道な書き換えが必要になりそう。ただし変換ルール自体は機械的に定義できるので、Agent Skill 化して AI エージェントに任せれば現実的に回せそう

という印象でした。ドキュメントサイトが「人間向けのウェブページ」であると同時に「AI エージェントが読み込む知識ソース」でもある、という前提に立ったツール設計は、今後のドキュメントツール全般のトレンドになっていきそうだと感じています。

## 参考

- [Blume 公式サイト](https://useblume.dev/)
- [Material for MkDocsチームが作る新しい静的サイトジェネレーター「Zensical」を試してみた](https://qiita.com/heki-dm/items/a7e37c13a0b677709851)
