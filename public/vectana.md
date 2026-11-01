---
title: 依存パッケージの脆弱性とEOLをまとめて監視する「Vectana」を作った
tags:
  - FastAPI
  - Next.js
  - OSv
  - 脆弱性
  - OSS
private: false
updated_at: '2026-11-01T20:09:38+09:00'
id: e2572853d432380c9a39
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## はじめに

複数のプロジェクトを並行して見ていると、こんなことがよくあります。

- 「あのライブラリに脆弱性が出たらしいけど、うちのどのプロジェクトが使ってる？」
- 「この案件の Python や Node.js、いつまでサポートされるんだっけ？」

Dependabot などのリポジトリ単位のツールは便利ですが、会社・部署・協力会社をまたいで「全プロジェクトの状況を一か所で見たい」となると、ちょうどいいものが見つかりませんでした。

そこで、プロジェクトが使っているパッケージとランタイムを登録しておくと、**脆弱性**と**サポート終了（EOL）** を自動で監視して通知してくれるアプリ「**Vectana**（ベクタナ）」を作りました。

- ドキュメント: <https://docs.vectana.techolve.org>
- GitHub: <https://github.com/techolve/vectana>
- Docker Hub: <https://hub.docker.com/r/heki0824/vectana>

![Vectana のダッシュボード](https://docs.vectana.techolve.org/screenshots/ja/dashboard.png)

## Vectana でできること

| 機能                | 内容                                                                                                                       |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| 脆弱性の検知        | 登録したパッケージのバージョンを [OSV](https://osv.dev/) に定期照会し、影響する脆弱性と修正版を表示                        |
| EOL の検知          | フレームワーク・言語・DB・OS のサポート期限を [endoflife.date](https://endoflife.date/) で確認し、期限切れ・期限間近を表示 |
| lockfile の取り込み | `package-lock.json` や `uv.lock`、`pubspec.lock`、`mise.toml` をドロップするだけで登録                                     |
| 通知                | 新しく見つかったものを、メールと Discord / Slack にまとめて送信                                                            |
| 組織と権限          | グループ > 会社 > 部署の構成と、BP（協力会社）を含めた権限で見える範囲を分ける                                             |
| セルフホスト        | Docker Compose で起動。AGPL-3.0 で公開                                                                                     |

画面は日本語と英語に対応しています。

### 仕組み

1. プロジェクトを作り、lockfile とランタイム（言語・DB・OS など）を登録する
2. Vectana が OSV と endoflife.date に問い合わせて、脆弱性と EOL を判定する
3. 新しく見つかったもの・状態が変わったものを、メールと Webhook でまとめて通知する
4. 以降は定期的に（初期設定では 6 時間ごと）自動でチェックし続ける

## 使ってみる

### lockfile をドロップして登録する

パッケージの登録は、プロジェクトの「パッケージ」タブに lockfile をドロップするのがいちばん楽です。複数ファイルを一度に取り込めます。

![パッケージタブ](https://docs.vectana.techolve.org/screenshots/ja/project-packages.png)

| ファイル                                                       | 種類                         |
| -------------------------------------------------------------- | ---------------------------- |
| `package-lock.json` / `npm-shrinkwrap.json` / `pnpm-lock.yaml` | npm                          |
| `uv.lock` / `poetry.lock` / `requirements*.txt`                | PyPI                         |
| `pubspec.lock`                                                 | Pub（Dart / Flutter）        |
| `mise.toml` / `.tool-versions`                                 | ランタイム（EOL 判定に使う） |

普段 Flutter / Next.js / Django を触っているので、まずはこのあたりの lockfile に対応しました。Maven / Go / crates.io / RubyGems / Packagist / NuGet は手入力で登録できます。

同じファイル名で取り込み直すと**差分だけ**が反映されます。依存を更新したら lockfile を入れ直すだけで、古いバージョンの検知結果が消えて最新の状態になります。

:::note warn
差分はファイル名で判断しているので、モノレポで `frontend/package-lock.json` と `admin/package-lock.json` のように同じ名前のファイルがある場合は、プロジェクトを分けて登録してください。
:::

### mise.toml からランタイムの EOL を拾う

言語や DB のように lockfile に出てこないものは「ランタイム」として登録します。手で入力することもできますが、[mise](https://mise.jdx.dev/) の `mise.toml` や asdf の `.tool-versions` をドロップすれば、`[tools]` の中身がそのままランタイムとして登録されます。

```toml:mise.toml
[tools]
python = "3.14"
node = ["24", "22"]
postgres = { version = "18" }
java = "temurin-21"
uv = "latest"
```

- `node` → Node.js、`postgres` → PostgreSQL のように、endoflife.date の製品名へ自動で対応付け
- `temurin-21` や `corretto-17` のようなベンダー付きの Java にも対応
- `latest` や `lts` のようにバージョンが決まらないものや、endoflife.date にデータがないもの（`uv` など）は、理由を表示してスキップ

![EOL の一覧](https://docs.vectana.techolve.org/screenshots/ja/eol.png)

EOL の状態は「サポート終了」「まもなく終了」「サポート中」「不明」の 4 つです。「まもなく終了」にする日数は管理画面で変えられます（初期値は 90 日）。

### Discord / Slack に通知する

プロジェクトにアサインされたメンバーにはメールが届きます。チームのチャンネルに流したい場合は、プロジェクトごとに Discord / Slack の Webhook を登録します。

![通知（Webhook）タブ](https://docs.vectana.techolve.org/screenshots/ja/project-webhooks.png)

通知まわりは「うるさくならないこと」を意識して、次のルールにしています。

- 1 回のチェックで見つかったものは、宛先ごとに 1 通にまとめる
- 同じ脆弱性を同じ宛先に 2 回通知しない
- EOL は「まもなく終了」になったとき、「サポート終了」になったときの 1 回ずつ
- 「確認済み」「無視」にした脆弱性は通知しない
- 通知する重大度のしきい値（例: HIGH 以上）を、個人・Webhook ごとに設定できる

### 会社・部署・協力会社で見える範囲を分ける

自分の仕事柄、グループ会社や協力会社（BP）のメンバーと一緒にプロジェクトを進めることが多いので、組織と権限はそれに合わせて作りました。

```txt
グループ（Vectana 全体）
├── 親会社
│   ├── 開発本部
│   │   ├── 第1開発部
│   │   └── 第2開発部
│   └── 営業部
├── 関連会社
└── BP（協力会社）
```

| 誰             | 見えるプロジェクト                                             |
| -------------- | -------------------------------------------------------------- |
| グループ管理者 | すべて                                                         |
| 会社管理者     | 自分の会社のすべて                                             |
| 部署長         | 担当部署とその配下の部署のプロジェクト（＋アサインされたもの） |
| メンバー / BP  | アサインされたプロジェクトだけ                                 |

見えないプロジェクトは一覧に出ず、URL を直接開いても「見つかりません」になります。

## 技術構成

| パス                | 内容                                                     |
| ------------------- | -------------------------------------------------------- |
| `api/`              | FastAPI。API と定期チェック用の worker を含む            |
| `web/`              | Next.js（静的書き出し）。ビルドしたものを FastAPI が配信 |
| `docker/Dockerfile` | アプリのイメージ（1 つ）                                 |
| `compose.prod.yaml` | 本番用（Docker Hub のイメージを使う）                    |

最近 FastAPI を触り始めていたので、API は FastAPI で書きました。フロントは Next.js を静的書き出しして FastAPI から配信しているので、本番で動かすイメージは 1 つだけです。

そのイメージを役割ごとに 4 つのコンテナとして起動します。

| サービス  | 役割                                           |
| --------- | ---------------------------------------------- |
| `db`      | PostgreSQL 18                                  |
| `migrate` | 起動時に DB を最新の構造にして終了             |
| `app`     | 画面と API（`127.0.0.1:8000`）                 |
| `worker`  | 起動時と定期的に全パッケージをチェックして通知 |

開発環境のツールとタスクは mise で管理しています。

```bash
mise trust && mise install   # Python / Node.js / uv / git-cliff
mise run setup               # api と web の依存関係
mise run services            # PostgreSQL と Mailpit を Docker で起動
mise run migrate             # DB を最新にする
mise run dev                 # API (:8000) と web (:3100) を起動
```

## セルフホストしてみる

Docker と Docker Compose v2 が使える Linux サーバーがあれば動きます（メモリ 1GB 以上推奨）。

```bash
mkdir -p /srv/vectana && cd /srv/vectana
curl -fsSLo compose.yaml https://raw.githubusercontent.com/techolve/vectana/main/compose.prod.yaml
curl -fsSLo .env https://raw.githubusercontent.com/techolve/vectana/main/.env.example
chmod 600 .env
```

`.env` には最低限これだけ設定します。

```bash:.env
VECTANA_VERSION=1.0.0
POSTGRES_PASSWORD=（openssl rand -hex 24 の出力）
SECRET_KEY=（openssl rand -hex 32 の出力）
WEB_URL=https://vectana.example.com
GROUP_NAME=Example Group
```

起動して、最初の管理者を作ります。

```bash
docker compose up -d
curl http://127.0.0.1:8000/api/health
# {"status":"ok","version":"1.0.0"}

docker compose exec app python -m app.cli create-admin \
  --email you@example.com --name "管理者" --company "Example Inc."
```

`app` は `127.0.0.1:8000` でしか待ち受けないので、外部には Caddy や nginx、Cloudflare Tunnel を通して HTTPS で公開します。メールは SMTP のほか、Cloudflare Email Service にも対応しています。詳しくはドキュメントの「[セルフホスティング](https://docs.vectana.techolve.org/self-hosting/install)」を見てください。

## ドキュメントは Blume で作った

ドキュメントサイトは、以前記事にした「Blume」で作りました。

- [AI時代のドキュメントツール「Blume」を触ってみた](https://qiita.com/heki-dm/items/9a216a79e5ad3c0f06e7)

Markdown を書くだけで日本語・英語のサイトになり、`llms.txt` / `llms-full.txt` や各ページの `.md` 版、Agent Skills も自動で出てくれます。AI エージェントに「Vectana のドキュメントを読んで設定して」と頼みやすくなるので、セルフホスト向けのアプリとは相性がいいと感じています。

## おわりに

Vectana は AGPL-3.0 で公開しています。複数のプロジェクトの依存関係やランタイムのサポート期限を管理しきれていない方は、ぜひ試してみてください。Issue や PR も歓迎です。

- ドキュメント: <https://docs.vectana.techolve.org>
- GitHub: <https://github.com/techolve/vectana>

### データの出典

- 脆弱性情報: [OSV.dev](https://osv.dev/)（Google が運営）。各アドバイザリは提供元のライセンスに従います（多くは CC BY 4.0）
- EOL 情報: [endoflife.date](https://endoflife.date/)（MIT License）
