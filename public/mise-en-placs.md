---
title: MISE-EN-PLACEを使ってみた
tags:
  - MISE-EN-PLACE
private: false
updated_at: '2026-01-21T22:31:15+09:00'
id: fc7cc93c0400290c49fd
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## 導入のきっかけ

Node.js のバージョン管理ツールとして Volta を使用していましたが、 Volta のサポート終了が発表されました。その際、移行先として推奨されていたのが mise でした。

Volta の公式サイトでも mise への移行を促すアナウンスがあり、これを機に mise を試してみることにしました。

## MISE-EN-PLACE とは

mise（旧称 rtx）は、複数のプログラミング言語やツールのバージョンを管理できる統合的なツールです。Node.js、Python、Ruby、Flutter など、さまざまな開発環境のバージョン管理を 1 つのツールで行うことができます。

主な特徴：

- 複数の言語・ツールを 1 つのツールで管理
- `.tool-versions`ファイルによるバージョン指定
- プロジェクトごとに異なるバージョンを自動切り替え
- asdf との互換性

公式サイト: https://mise.jdx.dev/

## インストール手順

### macOS / Linux

公式のインストールスクリプトを使用します：

```bash
curl https://mise.jdx.dev/install.sh | sh
```

インストール後、シェルの設定ファイル（`.bashrc`、`.zshrc`など）に以下を追加します：

```bash
# ~/.zshrc の場合
eval "$(~/.local/bin/mise activate zsh)"
```

設定を反映させます：

```bash
source ~/.zshrc
```

### Windows

Windows では Winget を使ってインストールできます：

```powershell
winget install jdx.mise
```

#### Windows での注意点

Windows でインストール後は、パスの設定を確認する必要があります。

```powershell
mise doctor
```

このコマンドでパスの疎通を確認し、問題がある場合は環境変数の PATH に mise の bin ディレクトリを追加する必要があります。私の環境では、パスを明示的に通さないと mise でインストールしたツールが使えませんでした。

## 実際に使ってみた

### Node.js での使用

Node.js のインストールと使用は非常にスムーズでした：

```bash
# 最新LTS版をインストール
mise install node@lts

# プロジェクトでNode.js 20を使用
mise use node@24

# グローバルで使用するバージョンを指定
mise use -g node@24
```

特に問題なく、Volta と同じような感覚で使用できました。プロジェクトごとのバージョン切り替えも自動で行われ、快適に開発できています。

### Flutter での使用

Flutter も mise でバージョン管理できます：

```bash
# Flutterをインストール
mise install flutter@latest

# プロジェクトでFlutterを使用
mise use flutter@3.x
```

**CLI ベースの開発では問題なく使用できました。** `flutter run`、`flutter build`などのコマンドは正常に動作します。

#### VS Code 拡張機能での問題

ただし、注意点があります。もともとシステムにインストールしていた Flutter SDK を削除すると、**VS Code の拡張機能（Dart/Flutter 拡張）でのアプリ実行ができなくなりました。**

VS Code の拡張機能は、mise でインストールした Flutter を認識しにくいようです。この問題については以下の対処法が考えられます：

1. VS Code の設定で明示的に Flutter SDK のパスを指定する
2. システムに従来通り Flutter SDK をインストールし、mise は特定プロジェクトでのバージョン切り替え用途に限定する

私の場合は、CLI での開発がメインだったため大きな問題にはなりませんでしたが、VS Code の統合環境を多用する場合は注意が必要です。

## まとめ

mise は複数の言語やツールを一元管理できる便利なツールです。特に Node.js での使用は非常に快適で、Volta からの移行もスムーズに行えました。

Flutter など、IDE 連携が重要なツールについては、環境によって調整が必要な場合がありますが、CLI ベースの開発であれば問題なく使用できます。

Volta からの移行を検討している方や、複数の開発環境を管理したい方にはおすすめのツールです。
