# herdr-party 🎉

[Herdr](https://herdr.dev) 上で動いている Claude Code を、右サイドのペインにパーティー会場として描くプラグインです。
セッション一つが棒人間、space（ワークスペース）が会場のステージになります。
誰が働いていて、誰が承認を待っていて、誰が終わって呼んでいるかが、ひと目で分かります。

```
     ◐ 2 dancing   ◆ 1 waiting   ✔ 1 done   ● 2 chilling      00:09:29
                     ◆ Matsu 13m   ✔ Shizu
┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
     Sora    Aoto   Matsu   Shizu   Subaru
    (54s)           (13m)            (2h)
     \o/      o      !o!     \o      \o/
      |      /|\      |       |\      |
     / \     / \     / \     / \     / \
  ▄▄▄▄▄▄▄▄▄▄▄█████▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
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
- 承認待ちと完了の guest には、ホバーしなくても頭上に小さな吹き出しが出ます。ビューアが状態の変化を見てからの経過時間（`3m` など）が入り、承認待ちは明滅します。変化を見た時刻は `~/.local/state/herdr-party/state-since.json`（プラグインのペインとして開いた場合は `HERDR_PLUGIN_STATE_DIR`）に保存するので、ビューアを開き直しても同じ状態が続いていれば経過時間を引き継ぎます。一度も変化を見ていない guest には吹き出しを出しません。
- 自分がいるセッションの足元は、ステージが明るい黄色になります（スポットライト）。フォーカス中の space はステージの名前が太字です。

### ステージ（space）

- ステージは会場幅の 8 割を基本の幅とし、出演者はステージの中央に寄って立ちます。1 行の台と、その下に space の名前だけの簡素な形です。
- 1 列に収まらないときはステージを会場いっぱいまで広げ、それでも余る人は後ろの列（上側）に並びます。
- ステージの色は space ごとに変わります。

### 上部の集計

1 行目は状態ごとの人数と時計です。承認待ちや完了の guest がいるときは、2 行目に名前と待ち時間を、承認待ち → 完了、待ちが長い順に並べます。

## 操作

| 操作 | 動き |
| --- | --- |
| 棒人間にマウスを乗せる | タイトルの吹き出しが出る |
| 棒人間を左クリック | そのセッションのペインに Herdr のフォーカスが移る（`herdr agent focus`） |
| ステージを左クリック | その space に移る（`herdr workspace focus`） |
| `←` `→`（`h` `l` `j` `k` `↑` `↓` も可） | guest を選ぶ。選んだ guest は名前がピンクになり、吹き出しが出る |
| `Tab` | 承認待ち・完了の guest へ順にジャンプ |
| `Enter` / `o` | 選んだ guest のセッションを開く |
| `Esc` | 選択を解除 |
| `?` | 操作の説明を出す / 消す |
| `q` | 終了（ペインも閉じる） |

## サイドバーに常駐させる

ペインは space やタブを移動すると隠れます。どこにいても見えるようにするには、Herdr のサイドバーを使います。
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

- ビューアは `herdr agent list` と `herdr workspace list` を 2 秒ごとに取得し、0.5 秒ごとに描画します。
- xterm のマウス移動追跡（1003 モード）を有効にするので、Herdr の `mouse_capture` が有効でもホバーとクリックがペインに届きます。
- Herdr 0.9.0 以前のクライアントは、API で space を切り替えた直後にクリックしたペインの space へ引き戻すことがあります（0.9.1 で修正。changelog #3760 #4153 #4171）。0.9.0 以前ではボタンを離してから 1.2 秒待って切り替え、そのあと 2 秒間は戻されていないか監視して、戻されていたら切り替え直します。0.9.1 以降は待ちを 0.2 秒にしています。クリックで確実に移動したいなら `herdr update` で 0.9.1 以降にしてください。
- 環境変数 `HERDR_PARTY_LOG` にファイルパスを入れて起動すると、クリックとフォーカスの経過がそのファイルに残ります。

## 今後のアイデア

- Herdr の socket API にある `pane.agent_status_changed` イベントを購読してポーリングをやめる
- `pane report-metadata` でサイドバーにも小さな棒人間を出し、どの space にいても見えるようにする
