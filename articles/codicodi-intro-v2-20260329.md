---
title: "Codicodiってなんだ？"
emoji: "🖥️"
type: "tech"
topics: ["codex", "discord", "tauri", "automation"]
published: false
---

Codex CLI を使っていると、「別の部屋から進捗を見たい」「Discord から指示を投げたい」「ローカルでも同じセッションを触りたい」と思うことがありました。

そこで作ったのが、**Codicodi（Codex Discord Connected Display）** です。これは、ローカル UI と Discord の両方から同じ Codex セッションを操作できる Windows 向けブリッジアプリです。Node.js で bridge サーバーを動かし、Tauri でデスクトップアプリ化しています。

リポジトリはこちらです。
[GitHub リポジトリはこちら](https://github.com/harunamitrader/codicodi)

単なる Discord Bot ではなく、**ローカル UI と Discord を 1 つのセッションで同期させるラッパー** というのがポイントです。片方で送ったメッセージはもう片方にも届くし、進捗もリアルタイムで共有されます。この記事では、何ができるかと、実際の導入手順をまとめます。

![Codicodi のヘッダー画像](/images/codicodi/header.jpg)

## Codicodi でできること

まず、できることをざっくり整理すると次の通りです。

- ローカル UI と Discord の両方からテキスト・画像・ファイルを送信できる
- セッションの作成・切替・名前変更・削除ができる
- model / reasoning level / fast mode を切り替えられる
- 実行中の Codex を途中で止められる
- 進捗や途中メッセージがローカルと Discord の両方に表示される
- 指定フォルダのファイル変更を検知して Discord に通知できる
- Tauri デスクトップアプリとしてネイティブに動く

普段の使い方はシンプルで、**ローカル UI か Discord のどちらかからメッセージを送るだけ** です。セッション管理やパラメータ変更は、必要なときに Discord のスラッシュコマンドか UI のボタンで操作します。

![ローカル UI でセッションを操作している画面](/images/codicodi/local-ui.png)
*ローカル UI — セッション一覧、モデル選択、チャット入力が1画面に*

![Discord からメッセージを送っている画面](/images/codicodi/discord-chat.png)
*Discord から画像を送信し、Codex が応答している様子*

## 主な /codex コマンド

Discord 側の操作はスラッシュコマンドで行います。

- `/codex help` : 使えるコマンド一覧を表示
- `/codex session` : セッション一覧を表示
- `/codex new` : 新規セッションを作成
- `/codex status` : 接続状態を確認
- `/codex model` : モデルの確認・切替
- `/codex reasoning` : reasoning level の切替
- `/codex fast on|off` : fast mode の切替
- `/codex stop` : 実行中のセッションを停止
- `/codex rename name:新しい名前` : セッション名を変更

![/codex help の実行結果](/images/codicodi/discord-help.png)
*/codex help — 使えるコマンド一覧*

## どんな人に向いているか

この Bot は、次のような人に向いています。

- Codex CLI を日常的に使っている
- 別部屋や外出先から進捗を確認したい
- ローカルと Discord を行き来しながら作業したい
- セッションを閉じても復旧できる環境がほしい

逆に、**公開サーバーでの運用や多人数共有には向きません。** ローカル PC 上の Codex CLI を Discord 経由で操作できる性質上、セキュリティの管理は自分で行う必要があります。

## 導入前に必ず理解しておきたいこと

これは先に書いておきます。

Codicodi は、仕組み上 **Discord から PC 上の Codex CLI を操作するツール** です。入力した文章やファイルは Codex に渡され、会話内容は Discord に送信されます。

特に注意が必要な設定があります。

- `CODEX_BYPASS_APPROVALS_AND_SANDBOX=true` にすると、Codex が承認なしでコマンドを実行します
- `CODEX_ENABLE_SEARCH=true` にすると、外部ネットワークアクセスが有効になります
- ファイル監視を有効にすると、変更されたファイルが自動で Discord に飛びます

最低限、次のルールは守ってください。

- `.env` の Bot トークンは絶対に公開しない
- `DISCORD_ALLOWED_GUILD_IDS` は自分のサーバー 1 つに限定する
- 公開サーバーでは使わない
- `data/bridge.sqlite` や `data/uploads/` には会話や添付が保存されることを把握しておく
- bridge のポートを外部に公開しない

便利ですが、安全な一般向けツールではありません。**完全に自己責任で、安全を優先して使う前提** が必要です。

## 導入に必要なもの

導入前に必要なのは次の 4 つです。

- Windows PC
- Node.js 22 LTS 以上
- Codex CLI（`codex --help` が通ること）
- Discord Bot（Message Content Intent 有効化必須）

## 導入手順

Antigravity Discord Bot のときと同じで、いちばん楽なのは **Codex 自身に導入作業を手伝わせる方法** です。

### 方法 1. Codex に導入を手伝わせる

Codex に次のように投げるだけです。

```text
https://github.com/harunamitrader/codicodi を導入して。可能な範囲でAI側で作業を行い、必要な情報があれば質問して。手動で行う必要があるものは丁寧にやり方を教えて。
```

リポジトリ取得、依存関係の導入、`.env` の用意あたりは Codex にかなり任せられます。ただし、Discord Bot の作成やトークン取得は人間側での作業が必要です。

### 方法 2. 手動でセットアップする

手動でも導入できます。分からなくなったら Codex に聞くのがいちばん早いです。

### 1. Discord Bot を作る

[Discord Developer Portal](https://discord.com/developers/applications) で Bot を作成し、トークンを取得します。`MESSAGE CONTENT INTENT` を含む必要な Intent を有効化し、`bot` + `applications.commands` Scope で自分のサーバーに招待します。

### 2. リポジトリを取得して依存関係を入れる

```bash
git clone https://github.com/harunamitrader/codicodi.git
cd codicodi
npm install
```

### 3. `.env` を用意する

```bash
cp .env.example .env
```

少なくとも次の 3 つを埋めます。

- `DISCORD_BOT_TOKEN` : Bot のトークン
- `DISCORD_ALLOWED_GUILD_IDS` : 自分の Discord サーバー ID（1 つだけ）
- `CODEX_WORKDIR` : Codex の作業ディレクトリ

### 4. 起動する

**Tauri デスクトップアプリ（推奨）:**

```bash
.\launch-tauri-dev.ps1
# または
npm run tauri:dev
```

**ブラウザで使う場合:**

```bash
npm start
# http://127.0.0.1:3087 にアクセス
```

## 使い始めたら最初に試したいこと

導入できたら、まずは次の順で動作確認すると分かりやすいです。

1. ローカル UI で `New Session` を作り、テキスト送信が通るか確認する
2. Discord で `/codex status` を実行し、同じセッションに紐付いているか見る
3. 画像やファイルを添付して Codex まで届くか確認する
4. `/codex fast on` や `/codex reasoning` でパラメータ変更が反映されるか試す

最初はシンプルに、メッセージ送信と状態確認から始めるのがおすすめです。

![/codex status の実行結果](/images/codicodi/discord-status.png)
*/codex status — セッションの接続状態を確認*

## まとめ

Codicodi は、ローカル UI と Discord を 1 つの Codex セッションでつなぐためのブリッジアプリです。

便利さの中心は、単なる Discord Bot ではなく、**セッション管理、ファイル送信、パラメータ切替、進捗共有をローカルと Discord の両方から行える** ところにあります。

その一方で、PC 上の Codex CLI を外部から操作する性質上、安全管理は自分でやる必要があります。まずは小さく、安全に、確認しながら使ってみてください。
