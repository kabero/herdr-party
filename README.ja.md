# herdr-party 🎉

[English](README.md) | 日本語

[Herdr](https://herdr.dev) 上で動いている Claude Code を、右サイドのペインにパーティー会場として描くプラグインです。
セッション一つが棒人間、space（ワークスペース）が会場のステージになります。
誰が働いていて、誰が承認を待っていて、誰が終わって呼んでいるかが、ひと目で分かります。

```
       2 dancing   1 waiting   1 done   2 chilling      00:09:29
                       Matsu 13m   Shizu
┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
   Sora [12]   Aoto [3]  Matsu [7]  ┆ Subaru [1]
     (54s)                (13m)      ┆    (2h)
      \o/         o        !o!       ┆    \o/
       |         /|\        |        ┆     |
      / \        / \       / \       ┆    / \
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
            master ✱3                 feat/x ✱1
                       herdr-party
```

## インストール

Herdr のプラグインとして登録します。ローカルの checkout をリンクする場合:

```bash
git clone https://github.com/kabero/herdr-party ~/ghq/github.com/kabero/herdr-party
herdr plugin link ~/ghq/github.com/kabero/herdr-party
herdr plugin action list --plugin kabe.herdr-party
```

GitHub から直接入れる場合は `herdr plugin install kabero/herdr-party` です。

プレフィックスキーに割り当てるには `~/.config/herdr/config.toml` に追加して `herdr server reload-config` します。

```toml
[[keys.command]]
key = "prefix+space"
type = "plugin_action"
command = "kabe.herdr-party.toggle"
description = "party pane"
```

プラグインが提供するもの:

| 種類 | id | 動き |
| --- | --- | --- |
| action | `kabe.herdr-party.toggle` | 現在のタブの右側に会場を開く。開いていれば閉じる |
| action | `kabe.herdr-party.open` / `close` | 開くだけ / 閉じるだけ |
| pane | `party` | Herdr 管理のペインとして開く（`herdr plugin pane open --plugin kabe.herdr-party --entrypoint party --placement split`） |
| pane | `peek` | ポップアップで会場を覗く（`--entrypoint peek`、`q` で閉じる） |

## 使い方

プラグインを使わずに、Herdr のペイン内（`HERDR_ENV=1`）で直接実行することもできます。

```bash
./bin/herdr-party            # 右側に幅 30% のペインを開く。すでに開いていれば閉じる（トグル）
./bin/herdr-party --open     # 開くだけ
./bin/herdr-party --close    # 閉じるだけ
./bin/herdr-party --width 0.4
./bin/herdr-party --focus    # 開いたペインにフォーカスを移す
./bin/herdr-party -- --all   # claude 以外のエージェントも招待
./bin/herdr-party -- --list  # 棒人間ではなく一覧表示
```

依存: `herdr`, `jq`（ランチャー）, `python3`（ビューア、標準ライブラリのみ）

## 見方

### 棒人間（セッション）

| 棒人間 | 記号 | 状態 |
| --- | --- | --- |
| `\o/` 踊っている（虹色に光る） | ◐ | working |
| ` o ` 立っている | ● | idle |
| `!o!` 慌てている | ◆ | blocked（承認や質問を待っている） |
| `\o ` 手を振っている | ✔ | done（バックグラウンドで終わって、まだ見ていない） |
| ` ? ` | ? | unknown |

- 各 guest にはセッション ID から決まるローマ字の名前が付きます。同じセッションは再起動しても同じ名前です。
- 名前の横に、そのセッションに出した指示の回数が `Sota [19]` のように出ます。Claude Code の会話ログ（`~/.claude/projects/*/<セッション ID>.jsonl`）のユーザー発言（ツール結果を除く）を差分で数えるので、ビューアを開き直しても正確です。ログが見つからないセッションには出しません。
- 承認待ち・完了・稼働中の guest は、名前の下にその状態になってからの経過時間が `(54s)` のように出ます。変化を見た時刻は `~/.local/state/herdr-party/state-since.json`（プラグインのペインとして開いた場合は `HERDR_PLUGIN_STATE_DIR`）に保存するので、ビューアを開き直しても同じ状態が続いていれば経過時間を引き継ぎます。一度も変化を見ていない guest には時間を出しません。
- 自分がいるセッションの足元は、ステージが明るい黄色になります（スポットライト）。フォーカス中の space はステージの名前が太字です。

### ステージ（space）

- ステージは会場幅の 8 割を基本の幅とし、出演者はステージの中央に寄って立ちます。1 行の台と、その下に space の名前だけの簡素な形です。
- 1 列に収まらないときはステージを会場いっぱいまで広げ、それでも余る人は後ろの列（上側）に並びます。
- ステージは git のリポジトリ単位です。同じリポジトリの worktree（Herdr では別の space）は 1 つのステージに立ち、worktree ごとの区画を仕切り `┆` で分けます。各区画の下にそのブランチと変更数（`master ✱3 ↑1` の形。コミットしていない変更の数とリモートとの差）が出て、ステージ名はリポジトリ名です。区画をクリックするとその worktree の space に移ります。
- 区画が横に収まらないときは、区画ごとに別のステージ（名前は同じリポジトリ名）に分けます。git 管理外の space は、space のラベルがそのままステージ名になります。
- ステージの色は space ごとに変わります。

### 誰に何を頼んだか

ステージ名の下（ロビーの人はその列の下）に、全員について `Sota ▸ スポットライト狭くして` のような 1 行が出ます。そのセッションに最後に投げたプロンプトの 1 行目を会話ログから差分で拾うので、新しい指示を送った瞬間に変わります。スラッシュコマンドはコマンド名と引数で出ます。

### ロビー

しばらく idle の人（既定 30 分以上。`--lobby-after 分` で変更、0 で無効）はステージを降りて、会場の下のロビーに集まります。薄い色で描かれ、名前の下に idle の長さが出て、長い順に並びます。ステージにはいま動きのある人だけが残るので視界がすっきりします。idle の長さは、ビューアが見た状態の変化から数え、ビューアを開く前から idle だった人は Claude Code の会話ログの最終記録時刻から数えます。ロビーの人もホバー、選択、クリックはできます。

### 上部の集計

1 行目は状態ごとの人数と時計です。誰かが踊っていると「dancing」の文字が虹色に流れます。承認待ちや完了の guest がいるときは、2 行目に名前と待ち時間を、承認待ち → 完了、待ちが長い順に並べます。

## 操作

| 操作 | 動き |
| --- | --- |
| 棒人間にマウスを乗せる | 吹き出しが出る。1 行目は状態・名前・経過時間・最後に見てからの時間、2 行目は最後に頼んだこと（`❯`）、3 行目は最後の応答の冒頭（`⏺`）か承認待ちの内容（`?`） |
| 棒人間を左クリック | そのセッションのペインに Herdr のフォーカスが移る（`herdr agent focus`） |
| ステージを左クリック | その space に移る（`herdr workspace focus`） |
| `←` `→`（`h` `l` `j` `k` `↑` `↓` も可） | guest を選ぶ。選んだ guest は名前がピンクになり、吹き出しが出る |
| `Tab` | 承認待ち・完了の guest へ順にジャンプ |
| `Enter` / `o` | 選んだ guest のセッションを開く。何も選んでいなければ、いま最も対応が必要な guest（承認待ち → 完了、待ちが長い順の先頭）へ飛ぶ。その guest は頭上に上下する `▼` が出て名前が状態色の太字になり、上部の一覧では `⏎` が付く |
| `Esc` | 選択を解除 |
| `?` | 操作の説明を出す / 消す |
| `q` | 終了（ペインも閉じる） |

プラグインのアクションでトグルすると、会場が追従して別の space にいても、そこにある会場を閉じます。

## 追従

ペインは 1 つのタブに属するので、そのままだとタブや space を移動したときに会場は隠れます。そこで既定では会場が付いてきます。別のタブの `tab.focused` イベントを見たら、自分のペインをそのタブへ `pane move` で引っ越し、同じ幅（`--width`、既定 30%）の右側の分割として現れます。タブの間を移動するだけなので、表示内容、経過時間、選択はそのまま残ります。会場のペイン自体をクリックしたときは動きません。付いてきてほしくなければ `--no-follow` を渡します。

```bash
./bin/herdr-party -- --no-follow   # 開いたタブから動かない
```

## サイドバーに常駐させる

ペインを動かしたくない場合は、Herdr のサイドバーでどこにいても見えるようにする手もあります。
ビューアに `--report` を付けるか、画面なしの `--report-only` を動かすと、各セッションのペインに名前とポーズがメタデータとして報告されます。

```bash
./bin/herdr-party -- --report          # 会場のペインを開きつつサイドバーにも報告
./bin/herdr-party-agents --report-only  # 画面なしで報告だけ（バックグラウンド向け）
```

`~/.config/herdr/config.toml` の agent 行に `$party_pose` と `$party_name` を足すと、サイドバーの各 agent に棒人間と名前が並びます。

```toml
[ui.sidebar.agents]
rows = [["state_icon", "machine", "workspace", "tab"], ["$party_pose", "$party_name", "agent"]]
```

報告は 8 秒で失効するので、ビューアを止めればサイドバーは元に戻ります。

## 仕組みのメモ

- 吹き出しの「頼んだこと」「応答」「承認待ちの内容」は、状態が変わったときに `herdr agent read` で画面テキストを読んで取り出します（`❯` 行、`⏺` で始まる応答行、承認ダイアログの箱の先頭）。「最後に見てから」は `pane.focused` イベントから数えます。
- ビューアは Herdr の socket API で `pane.focused` `pane.agent_status_changed` `pane.created` などのイベントを購読し、来た瞬間に反映します。フォーカスの移動はイベントの pane_id から即座にスポットライトへ反映し、それ以外は `herdr agent list` と `herdr workspace list` を取り直します。保険として 2 秒ごとの取得も続け、描画は 0.5 秒ごとです。
- xterm のマウス移動追跡（1003 モード）を有効にするので、Herdr の `mouse_capture` が有効でもホバーとクリックがペインに届きます。
- Herdr 0.9.0 以前のクライアントは、API で space を切り替えた直後にクリックしたペインの space へ引き戻すことがあります（0.9.1 で修正。changelog #3760 #4153 #4171）。0.9.0 以前ではボタンを離してから 1.2 秒待って切り替え、そのあと 2 秒間は戻されていないか監視して、戻されていたら切り替え直します。0.9.1 以降は待ちを 0.2 秒にしています。クリックで確実に移動したいなら `herdr update` で 0.9.1 以降にしてください。
- 環境変数 `HERDR_PARTY_LOG` にファイルパスを入れて起動すると、クリックとフォーカスの経過がそのファイルに残ります。
