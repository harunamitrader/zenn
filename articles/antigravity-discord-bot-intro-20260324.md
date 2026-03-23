---
title: "Antigravity Discord Botってなんだ？"
emoji: "🤖"
type: "tech"
topics: ["antigravity", "discord", "ai", "automation"]
published: true
---

Antigravity を使っていると、「外出先や別の部屋から操作したい」「Discord から指示を投げられたら便利なのに」と思うことがありました。

そこで作ったのが、**Antigravity Discord Bot** です。これは、Discord から Antigravity を遠隔操作するための自作ボットです。Chrome DevTools Protocol を使って Antigravity の状態にアクセスし、メッセージ送信や承認操作、スクリーンショット取得などを行います。

リポジトリはこちらです。  
[GitHub リポジトリはこちら](https://github.com/harunamitrader/antigravity-discord-bot)

かなり便利な一方で、性質としては **PC を Discord 経由で触れるようにするツール** でもあるので、導入前にできることと危険性の両方を理解しておくのが大事です。この記事では、何ができるかと、実際の導入手順を中心に丁寧にまとめます。

正直言うと、危険性もあるし、グレーなツールなので、こういうものを GitHub で公開したり記事を書くのはどうなのかな……という気持ちはある。いや、使ってもらわなくて全然いいんだけど、作ったものはとりあえず見てもらいたいじゃん？（承認欲求w）

![Antigravity Discord Bot のヘッダー画像](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/antigravity-discord-bot-header.jpg)

## この Bot でできること

まず、できることをざっくり整理すると次の通りです。

- Discord メッセージをそのまま Antigravity に送って生成を始められる
- 画像やテキストファイルを添付して Antigravity に渡せる
- Antigravity 側で出た選択肢や確認を Discord から見やすく受け取れる
- 現在の画面をキャプチャして Discord に返せる
- 直前の応答を再取得したり、新規チャットを切ったりできる
- プロジェクト内のファイル変更を検知して Discord に通知できる
- 接続先ウィンドウを切り替えて、複数ウィンドウ運用に対応できる
- Bot 自体の再起動まで Discord 側から行える

単なる「Discord から文章を投げるだけ」のボットではなく、**Antigravity 側の操作フローまでかなり踏み込んでいる**のがポイントです。

普段の使い方はかなりシンプルで、**基本的な AI チャットは Discord からメッセージを送るだけで OK** です。そのうえで、必要になったときだけスラッシュコマンドで補助機能を使える、という構成にしています。

また、Antigravity 側で選択肢や確認が出たときに、Discord 側でも状況を追いやすいようにしているのもポイントです。

![Discord から普通のメッセージを送って Antigravity と会話している画面](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/antigravity-discord-bot-chat-basic.jpg)

![Antigravity 側の選択肢が Discord に通知されている画面](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/antigravity-discord-bot-options.jpg)

## 主な /コマンド

基本の会話は普通の Discord メッセージで始められます。スラッシュコマンドは、必要なときに使う補助機能という位置づけです。

- `/help` : 使えるコマンド一覧を表示
- `/status` : 接続中ウィンドウやモードの状態を確認
- `/model` : 利用モデルの確認と切り替え
- `/conversation` : Planning / Fast の切り替え
- `/newchat` : 新規チャットを開始
- `/stop` : 生成を停止
- `/screenshot` : 現在の画面を取得
- `/last_response` : 直前の回答を取り直す
- `/window` : 接続先ウィンドウの確認・切り替え
- `/restart` : Bot プロセスを再起動

![help コマンドで使える補助機能を確認している画面](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/antigravity-discord-bot-help.jpg)

![status コマンドで接続中のタイトルやモデル、モードを確認している画面](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/antigravity-discord-bot-status.jpg)

## どんな人に向いているか

この Bot は、次のような人に向いています。

- Antigravity を日常的に使っている
- Discord から簡単に指示を送りたい
- 別部屋や外出先から作業の様子を見たい
- 複数ウィンドウで Antigravity を並行運用したい

逆に、**安全管理に自信がない人には向きません。** Bot トークンやユーザー ID 制限をミスすると、第三者が PC を触れる状態に近づくからです。

## 導入前に必ず理解しておきたいこと

これはかなり重要なので、便利さより先に書きます。

この Bot は、仕組み上 **Discord から PC を遠隔操作できるバックドアに近いツール** です。Bot の操作権限を奪われることは、**PC の乗っ取りにかなり近い** と考えたほうがいいです。

つまり、Bot トークンの漏洩、ユーザー ID 制限の設定ミス、運用中の油断が、そのまま危険な操作につながる可能性があります。ファイルの削除、機密情報の流出、認証情報の漏洩、任意コード実行のリスクがあります。

さらに、これは **Antigravity の非公式自動化ツール** です。規約違反と判断された場合、Google アカウントが予告なく一時停止や永久停止になるリスクもあります。ここは軽く見ないほうがいいです。

最低限、次のルールは必須だと考えてください。

- `.env` の Bot トークンは絶対に公開しない
- `DISCORD_ALLOWED_USER_ID` は自分だけに厳格に制限する
- 個人情報や重要データのあるメイン環境では使わない
- できれば独立した PC や仮想環境で使う
- バックアップなしで触らない
- 停止されても困らないアカウントや環境で試す

便利ですが、安全な一般向けツールではありません。試すなら、**完全に自己責任で、安全を優先して使う前提** が必要です。

## 導入に必要なもの

導入前に必要なのは次の 3 つです。

- Node.js 18 以上
- Antigravity
- Discord Bot

加えて、Antigravity は **デバッグポート付きで起動** する必要があります。

## 導入手順

README に書いている通り、いちばん楽なのは **Antigravity 自身に導入作業を手伝わせる方法** です。

### 方法 1. まずは Antigravity に導入を手伝わせる

Antigravity の AI チャットに、次のように投げる方法です。

```text
https://github.com/harunamitrader/antigravity-discord-bot を導入して。可能な範囲でAI側で作業を行い、必要な情報があれば質問して。手動で行う必要があるものは丁寧にやり方を教えて。
```

このやり方の良いところは、リポジトリ取得、依存関係の導入、`.env` の用意、起動補助のような部分を、Antigravity 側にかなり寄せられることです。

導入後に必要なら、README にある通り

```text
デバッグモード用ショートカットとantigravity-discord-botの起動用ショートカットをデスクトップに作成して
```

と追加で頼むのもありです。

ただし、Discord Bot の作成、トークン取得、ユーザー ID の確認などは人間側で確認が必要になります。なので、完全自動ではなく、**AI に案内役と半自動セットアップをさせる方法** と考えるのが近いです。

### 方法 2. 手動でセットアップする

手動でも入れられますが、分からなくなったら、**Antigravity にそのまま聞く** のがいちばん早いです。

### 1. Discord Bot を作る

まず [Discord Developer Portal](https://discord.com/developers/applications) で Bot を作り、トークンを取得して、自分のサーバーへ招待します。`MESSAGE CONTENT INTENT` を含む必要な Intent も忘れずに有効化します。

### 2. 自分の Discord ユーザー ID を取る

Discord の開発者モードをオンにして、自分のユーザー ID をコピーします。これは `.env` の `DISCORD_ALLOWED_USER_ID` に入れます。

### 3. リポジトリを取得して依存関係を入れる

```bash
git clone https://github.com/harunamitrader/antigravity-discord-bot.git
cd antigravity-discord-bot
npm install
```

その後、`.env.example` をコピーして `.env` を作り、`DISCORD_BOT_TOKEN` と `DISCORD_ALLOWED_USER_ID` を中心に必要な値を入れます。

### 4. Antigravity をデバッグモードで起動する

Antigravity のショートカットを複製し、リンク先の末尾に次を追加して起動します。

```text
--remote-debugging-port=9222
```

### 5. Browser CDP Port の競合を避ける

Antigravity 本体を 9222 で起動するなら、antigravity本体の設定で `Browser CDP Port` は `9333` など別の値に変えておくのがおすすめです。`9222` のままだと競合が起きることがあります。

![Antigravity の Browser CDP Port を 9333 に変更した設定画面](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/antigravity-discord-bot-browser-cdp-port.jpg)

### 6. Bot を起動する

起動はシンプルで、通常は次で動きます。

```bash
node discord_bot.js
```

または `start_bot.bat` を使います。

## 使い始めたら最初に試したいこと

導入できたら、まずは次の順で動作確認すると分かりやすいです。

1. 普通のメッセージを送って生成が始まるか確認する
2. `/status` で接続状態を確認する
3. `/screenshot` で現在画面が取れるか試す
4. `/newchat` と `/last_response` を試す

最初はシンプルに、メッセージ送信と状態確認から試すのがおすすめです。

## 複数ウィンドウ運用もできる

最近追加した機能の中では、マルチインスタンス対応も気に入っています。

`.env.bot2` を用意して 2 つ目の Bot を起動すると、別の Antigravity ウィンドウを割り当てて並行運用できます。`/window` で接続先を確認・切り替えられるので、1 台の PC で複数作業を回したいときに便利です。

## まとめ

Antigravity Discord Bot は、Discord から Antigravity を遠隔操作するための自作ツールです。

便利さの中心は、単なるメッセージ中継ではなく、**承認操作、スクリーンショット取得、ウィンドウ切り替え、ファイル監視、再起動** まで一つの操作系としてまとめたところにあります。

その一方で、かなり強い権限を持つツールでもあります。だからこそ、導入は丁寧に、運用は安全寄りにするのがおすすめです。

もしこの記事を読んで試すなら、最初は小さく、安全に、確認しながら使ってみてください。
