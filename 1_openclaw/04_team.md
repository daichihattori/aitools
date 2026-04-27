# チームで共有する Gateway を建てる

この章では、個人のローカル実験から一歩進めて、チームで共有する OpenClaw Gateway の考え方を整理する。

Claude Code MCP 接続までできると、OpenClaw は「自分の手元だけで動く Gateway」から「複数人が接続できる共有 Gateway」になる。

```text
Developer A: Claude Code
Developer B: Claude Code
Developer C: Claude Code
        ↓ MCP
Shared OpenClaw Gateway
        ↓
smolvm sandbox
```

---

## 1. 共有 Gateway で変わること

個人利用とチーム利用では、見るべきポイントが変わる。

| 観点 | 個人利用 | チーム共有 |
| --- | --- | --- |
| Claude Code | 自分の MCP 設定だけ | 各メンバーが同じ Gateway に接続 |
| Gateway | localhost / LAN で十分 | 常時起動、再起動、ログ確認が必要 |
| token | 手元の環境変数 | Secret 管理が必要 |
| 接続元 | 自分の端末だけ | 複数メンバーの端末 |
| 実行 | 試行錯誤 | 承認、監査、権限分離 |

---

## 2. 最小の共有構成

勉強会では、まず次の構成を目標にする。

```text
各メンバーの端末
  └── Claude Code
      └── openclaw mcp serve

講師または共有マシン
  └── smolvm machine: openclaw
      └── OpenClaw Gateway
```

Gateway は外部に広く公開しない。LAN / VPN / Tailscale など信頼できる範囲からだけ接続する。

---

## 3. 共有時の接続例

Gateway 側は共有マシンで起動する。

VM 内で実行する。

```bash
export OPENCLAW_GATEWAY_TOKEN=shared-gateway-token

openclaw gateway \
  --port 18789 \
  --allow-unconfigured \
  --verbose
```

このローカル構成では、各メンバーは同じマシン上の Gateway に `smolvm machine exec` 経由で接続する。

```bash
claude mcp add --transport stdio openclaw \
  -- smolvm machine exec --name openclaw -- \
    openclaw mcp serve \
    --url ws://127.0.0.1:18789 \
    --token shared-gateway-token \
    --claude-channel-mode on
```

別マシンのメンバーからも接続させる場合は、`--bind lan` と Control UI の allowed origins など追加設定が必要になる。この勉強会ではまずローカル共有までに留める。

実運用では token の直書きを避け、`--token-file` を使う。

```bash
claude mcp add --transport stdio openclaw \
  -- smolvm machine exec --name openclaw -- \
    openclaw mcp serve \
    --url ws://127.0.0.1:18789 \
    --token-file ~/.openclaw/gateway.token \
    --claude-channel-mode on
```

---

## 4. 運用チェック

Gateway を起動しているターミナルで、エラーが出ずにプロセスが残っていることを確認する。

ログ。

```bash
smolvm machine exec --name openclaw -- openclaw logs --follow
```

Claude Code 側の MCP 設定。

```bash
claude mcp get openclaw
claude mcp list
```

---

## 5. チーム導入時の注意点

### Gateway を公開しすぎない

`--bind lan` を使う場合は、必ず token / VPN / firewall / Control UI allowed origins を組み合わせる。

インターネットに直接公開するより、Tailscale などの private network 経由にするほうが扱いやすい。

### token を分ける

勉強会用、検証用、本番用の Gateway token は分ける。

同じ token を複数環境で使い回さない。

### Claude Code の MCP 設定を共有しすぎない

`.mcp.json` などで設定を共有する場合も、token はファイルや環境変数に逃がす。

`--token shared-gateway-token` のような値をリポジトリに入れない。

---

## まとめ

OpenClaw のチーム利用で重要なのは、モデル選択よりも Gateway の共有方法と権限設計。

- Claude Code を MCP client として使う
- OpenClaw Gateway をチームで共有する
- smolvm で Gateway と実行環境を隔離する
- token と実行権限を分ける
- approval を前提にする

この形にすると、OpenClaw は「各自がローカルで動かす実験ツール」から「チームの Gateway」になる。
