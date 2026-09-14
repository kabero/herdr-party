# herdr-party 🎉

[Herdr](https://herdr.dev) 上で動いている Claude Code を、右サイドの専用ペインにパーティー会場として描く小さなプラグインです。
セッション一つが棒人間、space（ワークスペース）がパーティー会場の机になります。

```
   ◐ 1 dancing   ◆ 1 waiting   ● 1 chilling      00:09:29
┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈

               ╭──────────────────────╮
               │ ● sensei.nvim 設計   │
               ╰────────┬─────────────╯
               Asahi   Akane   Ichiro
                \o/     o      !o!
                 |     /|\      |
                / \    / \     / \
       ┏━━━━━━━━○━━━━━━━○━━━━━━━○━━━━━━━━━━━━━━━┓
       ┃                herdr-party             ┃
       ┗━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━┛
         ┃                                   ┃
```

（Akane にマウスを乗せた状態）

## 使い方

Herdr のペイン内（`HERDR_ENV=1`）で実行します。

```bash
./bin/herdr-party            # 右側に幅 30% のペインを開いてビューアを起動
./bin/herdr-party --width 0.4
./bin/herdr-party --focus    # 開いたペインにフォーカスを移す
./bin/herdr-party -- --all   # claude 以外のエージェントも招待
./bin/herdr-party -- --list  # 棒人間ではなく一覧表示
```

ビューアのペインでは `q` で終了できます。ビューアは xterm のマウス移動追跡（1003 モード）を有効にするので、Herdr の `mouse_capture` が有効でもホバーがペインに届きます。

## 構成

- `bin/herdr-party` — `herdr pane split --direction right` で右ペインを作り、ビューアを起動するランチャー
- `bin/herdr-party-agents` — `herdr agent list` と `herdr workspace list` を一定間隔（デフォルト 2 秒）でポーリングし、0.5 秒ごとにアニメーション描画するビューア（Python 3、標準ライブラリのみ）

依存: `herdr`, `jq`（ランチャー）, `python3`（ビューア）

## キーバインドに登録する

`~/.config/herdr/config.toml` に追加すると、プレフィックスキーから起動できます。

```toml
[[keys.command]]
key = "prefix+alt+p"
type = "shell"
command = "/path/to/herdr-party/bin/herdr-party"
```

## 表示の見方

| 棒人間 | 名札 | 状態 |
| --- | --- | --- |
| `\o/` 踊っている（虹色に光る） | ◐ | working |
| ` o ` 立っている | ● | idle |
| `!o!` 慌てている | ◆ | blocked（承認や質問待ち） |
| `\o ` 手を振っている | ✔ | done（バックグラウンドで完了、未確認） |
| ` ? ` | ? | unknown |

机は会場幅の 8 割を基本の長さとし、客は机の中央に寄って座ります。天板には座席ごとに皿 ○ が並び、フォーカス中の space の机はラベルが太字になります。机の周りに入り切らない客は机の下側（脚の代わりに皿が並ぶ側）に回ります。

各 guest にはセッション ID から決まるローマ字の名前が付きます（同じセッションは再起動しても同じ名前）。棒人間にマウスを乗せると、頭上（机の下側の客は足元）に吹き出しが出て、セッションのタイトルが読めます。棒人間を左クリックすると `herdr agent focus` でそのセッションのペインに、机をクリックすると `herdr workspace focus` でその space に Herdr のフォーカスが移ります。ふだんは棒人間と名前だけなので、人数が増えても崩れません。

## 今後のアイデア

- Herdr の socket API にある `pane.agent_status_changed` イベントを購読してポーリングをやめる
- ビューア内で行を選んでそのエージェントにフォーカスする
