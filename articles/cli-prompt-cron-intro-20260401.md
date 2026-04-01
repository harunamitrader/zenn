---
title: "cli-prompt-cron AIチャットだけで導入から定期実行設定まで"
emoji: "⏰"
type: "tech"
topics: ["ai", "automation", "cli", "tools"]
published: false
---

> この記事は AI と一緒に下書きしています。

- 毎朝ほしいニュースを集めたり、
- 週1で定型タスクを投げたり
- 1時間に一回AIで何かを作成したい
みたいな場面がありませんか？

でも、実際にそこまでやろうとすると、

- cron を自分で書く
- 実行コマンドを整える
- 権限の扱いを考える
- ログを見る仕組みを作る

と、急に「運用のための作業」が増えます。

そこで作ったのが **cli-prompt-cron** です。  
これは、AI CLI 向けに **定期実行** を足すためのツールなのですが、コンセプトは、**導入から定期実行の設定まで全部、自然言語プロンプトで行うこと* です。

リポジトリはこちらです。  
[GitHub リポジトリはこちら](https://github.com/harunamitrader/cli-prompt-cron)

## 良いところ

このツールの価値は、単に cron を回せることだけではありません。

実際には次の2つがセットになっています。

1. AI CLI で定期実行が設定できる
2. その設定や運用を AI チャットに手伝わせやすい

たとえば、

```text
cli-prompt-cron の skills/SKILL.md を読んで、毎分「AIに関するトリビアを1つtxtに追記して」というプロンプトをcodexに送信するジョブを作って。
```

のように頼むと、ジョブ JSON の形や権限プロファイルを毎回人間が意識しなくても進めやすいです。

![AIチャットからジョブ作成を依頼している画面](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/cli-prompt-cron-chat-setup-20260401.png)

*AI チャットからそのままジョブ作成を依頼できるのが、cli-prompt-cron のかなり大きな強みです。*

## 何ができるのか

できることをざっくり整理すると、次の通りです。

- Gemini CLI / Claude Code / Codex を送信先として選べる
- `safe / edit / plan / full` の4段階で権限を付与
- 毎日、毎週、毎分などの cron 実行ができる
- ジョブの停止 / 再開 / 削除ができる
- ライブログと実行結果をブラウザで確認できる
- 実行中ジョブの強制停止ができる
- `skills/SKILL.md` を読ませることで、AI チャットからジョブ管理しやすい

**CLI の権限差は `safe / edit / plan / full` に寄せ**ています。

内部ではツールごとに別のオプションへ変換していますが、使う側は4つのラベルだけで管理します。

## ダッシュボードで何が見えるのか

`cli-prompt-cron` は cron を裏で回すだけではなく、**ブラウザダッシュボード** も持っています。

![cli-prompt-cron のダッシュボード全体](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/cli-prompt-cron-ui-overview-20260401.png)

*ジョブ一覧、ライブログ、実行結果を一画面で確認できます。*

この画面で確認できるのは主に次のものです。

- ジョブ一覧
- 次回実行時刻
- 実行中ジョブ
- ライブログ
- 実行結果

CLI ツール系って、便利でも「状態が見えにくい」ことが多いので、ここがあるだけでかなり安心感があります。

ジョブカードもただの一覧ではなく、

- `logId`
- 送信先 CLI
- 権限プロファイル
- prompt
- 次回実行

がまとまって見えます。

![ジョブカードの表示例](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/cli-prompt-cron-job-card-20260401.png)

*ジョブカードだけでも、何をどこへどの権限で投げるのかがかなり把握しやすいです。*

実行中ジョブについても、別セクションで見えるようになっています。

![実行中カードの表示例](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/cli-prompt-cron-running-card-20260401.png)

*いま何が走っているか、経過時間はどれくらいか、必要ならすぐ止められるかが分かります。*

結果もファイル一覧だけで終わらず、その場で開いて確認できます。

![実行結果モーダルの表示例](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/cli-prompt-cron-result-modal-20260401.png)

*出力確認のたびに別ファイルを開かなくていいのは、地味ですがかなり楽です。*

## 導入も AI にかなり任せやすい

このツールは、自分で手順を追ってももちろん入れられます。

```bash
git clone https://github.com/harunamitrader/cli-prompt-cron.git
cd cli-prompt-cron
npm install
npm start
```

ただ、ここでも AI チャットを使うとかなり楽です。

たとえば AI CLI に対して、

```text
https://github.com/harunamitrader/cli-prompt-cron をクローンして、npm install して
```

と頼めば、導入の最初のステップをかなりそのまま進められます。

さらに起動も、
npm install 時にデスクトップへショートカットが自動作成されるのでそこから起動できますし、

```text
cli-prompt-cron のデーモンとダッシュボードを起動して
```

のように頼むことでも起動できます。

**ツールを入れる作業そのものも AI に**頼めます。

## ジョブ作成も AI チャットで進めやすい

導入後の本番はむしろここです。

多くのツールは、インストールしたあとに

- JSON の形式を覚える
- cron 式を書く
- どの CLI へ送るか決める
- 権限をどうするか考える

という段階で手が止まりやすいです。

`cli-prompt-cron` では、`skills/SKILL.md` を AI に読ませる前提があるので、ここを自然言語で進めやすくしています。

たとえば、

```text
cli-prompt-cron の skills/SKILL.md を読んで、毎朝9時にGeminiに「GitHubのトレンドをまとめて」というプロンプトを送信するジョブを追加して。
```

のように言えば、ジョブファイル作成まで行えます。

ジョブ定義自体は、今は次のような形です。

```json
{
  "targetCli": "gemini",
  "permissionProfile": "safe",
  "prompt": "今日のニュースをまとめて",
  "cron": "0 9 * * *",
  "timezone": "Asia/Tokyo",
  "active": true
}
```

![ジョブ JSON の最小例](https://raw.githubusercontent.com/harunamitrader/zenn/main/images/cli-prompt-cron-job-json-20260401.png)

*保存される JSON も読みやすくしています。実コマンドを丸ごと覚えなくていいです。*

JSON に実行コマンドを直書きせず、

- `targetCli`
- `permissionProfile`
- `prompt`

などの情報をを持たせます。

実際のコマンドは `cli-prompt-cron` 側が実行時に組み立てます。  
このおかげで、**AI にジョブ作成を頼むときも、複雑な CLI オプションを毎回説明しなくて済む**ようになっています。

## どういう人に向いているか

このツールは、次のような人にかなり向いています。

- AI CLI を普段から使っている
- 定型 prompt を定期実行したい
- cron まわりの設定を毎回手で書きたくない
- AI にインストールやジョブ設定を手伝わせたい
- Gemini / Claude / Codex をまとめて扱いたい


## 注意点

もちろん、便利さだけではありません。

まず、`full` 権限はかなり強いです。  
実際の挙動は各 CLI 本体の権限モデルに依存するので、最初から全部 `full` にするのはあまりおすすめしません。

個人的には、

- まずは `safe`
- 必要が出たら `edit`

くらいから始めるのが扱いやすいと思います。

また、このツールが安全にしてくれるのは「設定の整理」であって、「危ないタスクを安全に変えること」ではありません。  
危ない prompt を毎分実行したら、危ない処理が毎分回るだけです。

そこは普通の cron と同じで、最後はジョブ内容の設計次第です。

## まとめ

`cli-prompt-cron` のいちばん面白いところは、**AI CLI に定期実行を足せる**こと以上に、**その設定作業まで AI チャットに手伝わせやすい**ことだと思っています。

つまり、

- ツールをクローンしてインストールする
- 起動する
- `skills/SKILL.md` を読ませる
- 定期実行ジョブを作る
- ダッシュボードで状態を見る

という流れを、かなり自然に AI と一緒に進められます。

「AI を使うツール」ではなく、**AI にツール運用も手伝わせる**方向で作業を進めたい人には、相性がいいです。

もし試すなら、まずは `safe` の小さなジョブを1本作って、AI チャットからどこまで設定を進められるかを体験してみるのがおすすめです。
