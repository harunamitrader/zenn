---
title: "codexがdiscordで使える。　codicodiってなんだ？"
emoji: "🖥️"
type: "tech"
topics: ["codex", "discord", "tauri", "automation"]
published: false
---

Codex CLI を使っていると、

- 別の部屋から進捗を見たい
- Discord から指示を投げたい
- でもローカル UI でも同じ会話を触りたい

と思うことがありました。

そこで作ったのが、**Codicodi（Codex Discord Connected Display）** です。  
これは、**1つの Codex CLI セッションをローカル UI と Discord の両方から共有して使うための Windows 向けローカルアプリ** です。

リポジトリはこちらです。  
[GitHub リポジトリはこちら](https://github.com/harunamitrader/codicodi)

単なる Discord Bot ではなく、**セッション管理、添付、進捗表示、停止、スケジュール実行まで含めて、ローカル UI と Discord をつなぐブリッジ** になっているのがポイントです。この記事では、今の `codicodi` で何ができるのか、導入前に何を理解しておくべきか、どう入れるのが楽かをまとめます。

![Codicodi のヘッダー画像](/images/codicodi/header.jpg)

## 良いところ

`codicodi` の価値は、単に Discord から Codex を叩けることだけではありません。

実際には次の4つがセットになっています。

1. ローカル UI と Discord で同じセッションを共有できる
2. セッションごとに model / reasoning / fast mode / working directory を切り替えられる
3. 進捗、停止、エラーが Discord 側でも追いやすい
4. cron スケジュールで既存セッションや新規セッションへ定期送信できる

つまり、**「普段は PC で触るけど、別の部屋や外出先からも同じ作業を追いたい」** という用途にかなり相性がいいです。

## 何ができるのか

できることをざっくり整理すると、次の通りです。

- ローカル UI と Discord の両方からテキストを送信できる
- ローカル UI と Discord の両方から画像とファイルを送信できる
- セッションの新規作成、切り替え、名前変更、削除ができる
- セッションごとに model / reasoning level / fast mode を切り替えられる
- セッションごとに working directory を切り替えられる
- `Restore Chat` で会話再取得や stale session 復旧ができる
- cron スケジュールで、既存セッションまたは新規セッション作成へ定期送信できる
- 実行中の Codex を途中で止められる
- 指定フォルダの変更を Discord に通知できる
- Tauri デスクトップアプリとして使える

特に今の `codicodi` で便利なのは、**ただ会話を中継するだけでなく、セッション単位の運用に必要なものがだいたい揃っている**ことです。

たとえば、

- 作業Aは `C:\Users\...\repo-a`
- 作業Bは `C:\Users\...\repo-b`

のように、セッションごとに作業フォルダを分けたまま使えます。  
さらに、定期実行したい処理は `Schedules` 画面から cron で回せます。

![ローカル UI でセッションを操作している画面](/images/codicodi/local-ui.png)
*ローカル UI では、セッション一覧、パラメータ切替、チャット入力をまとめて扱えます。*

![Discord からメッセージを送っている画面](/images/codicodi/discord-chat.png)
*Discord から送ったメッセージも同じセッションに入り、進捗や応答を追えます。*

## Discord 側で何が見えるのか

`codicodi` は Discord 側の見え方もかなり重要です。

主な操作は slash command で行います。

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
*/codex help で、今使える操作を Discord 側から確認できます。*

![/codex status の実行結果](/images/codicodi/discord-status.png)
*/codex status で、接続先や現在の状態を確認できます。*

進捗表示まわりも、最近かなり使いやすくなりました。

以前は長い `Working` が残って、見かけ上ずっと動いているように見えることがありましたが、今は **前の進捗メッセージを消して、新しい `Working` / `Completed` / `Stopped` / `Error` を一番下へ出す** 挙動にしています。

そのため、

- いま動いているのか
- もう止まったのか
- エラーで終わったのか

が Discord 上でもかなり分かりやすくなっています。`/codex stop` も、実行中のターンだけでなく未処理キューまで止めて結果を通知します。

## ローカル UI のどこが便利か

Discord 連携が目立ちますが、実際にはローカル UI 側もかなり重要です。

ローカル UI では主に次の操作ができます。

- `New Session` で新規セッション作成
- セッションカードのクリックで名前変更
- Discord チャンネルとの紐付け
- model / reasoning / fast mode の切り替え
- `Select Folder` でセッションごとの working directory 切り替え
- メッセージ送信
- ファイル追加
- drag & drop / paste 添付
- `Restore Chat`
- `Open Developer Console`
- `Schedules` 画面で cron スケジュールの追加 / 編集 / 停止 / 削除

特に `Schedules` は、**定期実行をそのまま `codicodi` の中で扱える**のが良いところです。

送信先も、

- 既存セッションに送る
- 実行のたびに新規セッションを作る

のどちらかを選べます。  
「毎朝この repo を確認する」「毎日この skill を流す」みたいな処理を、別ツールに逃がさずこのアプリ内で回せます。

## どんな人に向いているか

このアプリは、次のような人に向いています。

- Codex CLI を日常的に使っている
- ローカル UI と Discord を行き来しながら同じ作業を続けたい
- 作業ごとに working directory を切り替えたい
- 別部屋や外出先から進捗を見たい
- 定期タスクも同じ操作系の中で回したい

逆に、**不特定多数が使う共有サービス向けではありません。**

これはローカル PC 上の Codex CLI を Discord 経由で触れるようにするツールなので、安全管理はかなり重要です。便利さより先に、その前提を理解しておいたほうがいいです。

## 導入前に必ず理解しておきたいこと

これは先に書いておきます。

`codicodi` は、仕組み上 **Discord から PC 上の Codex CLI を操作できるツール** です。  
入力した文章やファイルは Codex に渡され、会話内容は Discord に送られます。

特に注意が必要なのは次です。

- `CODEX_BYPASS_APPROVALS_AND_SANDBOX=true` にすると、承認なし・sandbox 制限なしで Codex を実行できる
- `CODEX_ENABLE_SEARCH=true` にすると、外部ネットワークアクセスを許可する
- ファイル監視を有効にすると、変更されたファイルが Discord に自動投稿される
- `data/bridge.sqlite` や `data/uploads/` には会話や添付が保存される

最低限、次のルールは守ったほうがいいです。

- `.env` の Bot トークンは絶対に公開しない
- `DISCORD_ALLOWED_GUILD_IDS` は自分が管理するサーバー 1 つに限定する
- 共有サーバーや公開サーバーでは使わない
- bridge のポートを外部に公開しない
- 機密データが多いフォルダを安易に監視しない

便利ですが、安全な一般向けアプリではありません。**完全に自己責任で、安全寄りに運用する前提** が必要です。

## 導入に必要なもの

導入前に必要なのは次の4つです。

- Windows PC
- Node.js 22 LTS 以上
- Codex CLI
- Discord Bot

加えて、`.env` の用意と Discord Bot の設定が必要です。

## 導入手順

README にも書いていますが、いちばん楽なのは **Codex 自身に導入を手伝わせる方法** です。

### 方法 1. Codex に導入を手伝わせる

Codex に次のように投げるだけです。

```text
https://github.com/harunamitrader/codicodi を導入して。可能な範囲でAI側で作業を行い、必要な情報があれば質問して。手動で行う必要があるものは丁寧にやり方を教えて。Tauri デスクトップアプリとして起動できるようにデスクトップにショートカットを作成して
```

このやり方の良いところは、

- リポジトリ取得
- 依存関係の導入
- `.env` の下準備
- 起動補助スクリプトの利用

あたりをかなり AI に寄せられることです。

もちろん、Discord Bot の作成やトークン取得は人間側の作業が必要ですが、そこ以外はかなり進めやすいです。

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

少なくとも次の項目を埋めます。

- `DISCORD_BOT_TOKEN`
- `DISCORD_ALLOWED_GUILD_IDS`
- `CODEX_WORKDIR`

`CODEX_WORKDIR` は、標準の作業フォルダです。UI から選ぶ working directory や、スケジュール実行時の既定フォルダもこの配下で使う前提です。

### 4. 起動する

**Tauri デスクトップアプリ（推奨）**

```bash
.\launch-tauri-dev.ps1
# または
npm run tauri:dev
```

**ブラウザで使う場合**

```bash
npm start
```

その後、`http://127.0.0.1:3087` を開きます。

## 使い始めたら最初に試したいこと

導入できたら、まずは次の順で動作確認すると分かりやすいです。

1. ローカル UI で `New Session` を作り、テキスト送信が通るか確認する
2. Discord で `/codex status` を実行し、同じセッションに紐付いているか見る
3. `Select Folder` で working directory を切り替えて、そのフォルダで応答が返るか試す
4. 画像やファイルを添付して Codex まで届くか確認する
5. `/codex stop` を試して、Discord 側で `Stopped` が最下部に出るか確認する
6. `Schedules` 画面でテスト用の cron を1本作る

最初はシンプルに、

- メッセージ送信
- 状態確認
- 停止
- working directory 切り替え

あたりから試すのがおすすめです。

## まとめ

Codicodi は、ローカル UI と Discord を 1つの Codex セッションでつなぐためのブリッジアプリです。

便利さの中心は、単なる Discord Bot ではなく、

- セッション共有
- 添付ファイル送信
- model / reasoning / fast mode 切り替え
- セッションごとの working directory
- `Restore Chat`
- スケジュール実行
- 分かりやすい進捗 / 停止 / エラー表示

まで、**Codex を日常運用するための部品をまとめている**ところにあります。

その一方で、PC 上の Codex CLI を Discord 経由で触れる性質上、安全管理は自分でやる必要があります。まずは小さく、安全に、確認しながら使ってみてください。
