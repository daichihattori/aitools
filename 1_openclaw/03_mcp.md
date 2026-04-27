# Claude Code から MCP 接続する

この章では Claude Code から OpenClaw Gateway に MCP で接続する。

この構成では、LLM は Claude Code 側の Claude。OpenClaw は LLM バックエンドではなく、Claude Code から呼び出される MCP server / Gateway として動く。

```text
Claude Code
  ↓ MCP stdio
smolvm machine exec ... openclaw mcp serve
  ↓ VM 内 WebSocket
OpenClaw Gateway（smolvm 内）
```

---

## 1. Gateway を起動しておく

前章の手順で `openclaw` VM に入り、Gateway を起動する。

VM 内で実行する。

```bash
export OPENCLAW_GATEWAY_TOKEN=smolvm-local-token

mkdir -p /tmp/openclaw

nohup openclaw gateway run \
  --port 18789 \
  --verbose \
  > /tmp/openclaw/gateway.log 2>&1 < /dev/null &

tail -f /tmp/openclaw/gateway.log
```

ログに `ready` が出ればよい。`tail -f` は `Ctrl-C` で止めてもよい。Gateway は background で動き続ける。

---

## 2. Claude Code に MCP server を登録する

ホスト側で実行する。勉強会では、手元の環境だけに隠れる `local` scope ではなく、プロジェクトで確認しやすい `project` scope に登録する。

すでに同名の設定がある場合は削除する。

```bash
claude mcp remove openclaw
```

登録する。

```bash
claude mcp add --scope project --transport stdio openclaw \
  -- smolvm machine exec --name openclaw -- \
    openclaw mcp serve \
    --url ws://localhost:18789 \
    --token smolvm-local-token \
    --claude-channel-mode on
```

`--` より前は Claude Code 側の設定、`--` より後ろは Claude Code が起動する MCP server コマンド。

ここで起動されるのは、VM 内の `openclaw mcp serve`。この bridge が同じ VM 内の Gateway に `ws://localhost:18789` で WebSocket 接続する。

この形にすると、Gateway を LAN 公開しなくてよい。ホスト側に OpenClaw CLI を入れる必要もない。

登録内容を確認する。

```bash
claude mcp get openclaw
claude mcp list
```

---

## 3. 接続確認

Claude Code を起動する。

```bash
claude
```

Claude Code 内で MCP 状態を見る。

```text
/mcp
```

`openclaw` が connected になっていれば接続成功。

---

## 4. 使ってみる

Claude Code で OpenClaw の MCP tools が見えているか確認する。

```text
OpenClaw 経由で使える会話やツールを一覧して
```

Gateway に routed session がある場合、OpenClaw MCP bridge は conversation / message / approval 系の tools を Claude Code に公開する。

代表的な tools:

| tool | 役割 |
| --- | --- |
| `conversations_list` | Gateway が知っている会話を一覧する |
| `messages_read` | 会話履歴を読む |
| `events_poll` / `events_wait` | 新しいイベントを待つ |
| `messages_send` | 同じ route に返信する |
| `permissions_list_open` | 未処理の approval を見る |
| `permissions_respond` | approval に応答する |

---

## トラブルシューティング

### `openclaw` が connected にならない

Gateway が起動しているか確認する。

Gateway を起動しているターミナルで、エラーが出ずにプロセスが残っているか確認する。

MCP server の登録内容を見る。

```bash
claude mcp get openclaw
```

登録内容の `--url` は VM 内 localhost を指す。

```text
ws://localhost:18789
```

### `Missing config` と出る

Gateway 用の最小設定を入れる。

```bash
openclaw config set gateway.mode local
openclaw config set gateway.port 18789 --strict-json
```

### `non-loopback Control UI requires...` と出る

Gateway 起動時に `--bind lan` を付けている。Claude Code MCP の手順では LAN bind は不要なので外す。

### tools は見えるが会話が空

`openclaw mcp serve` は Gateway が持っている routed session を MCP tools として見せる。Gateway 側にまだ会話や route がない場合、一覧は空になる。

この章のゴールは、まず Claude Code と OpenClaw Gateway の MCP 接続を成立させること。

---

## ここまでできたこと

- smolvm 内で OpenClaw Gateway を起動した
- Claude Code に project scope で `openclaw mcp serve` を登録した
- Claude Code から MCP 経由で OpenClaw Gateway に接続した

次のステップでは、この Gateway をチームで共有する構成にする。
