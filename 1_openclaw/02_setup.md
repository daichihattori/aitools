# インストールとセットアップ

この章では `smol-machines/smolvm` の VM 内で OpenClaw Gateway を起動する。

この勉強会では、OpenClaw Gateway を **Claude Code から MCP で接続する先**として使う。Gateway は VM 内の `localhost:18789` で待ち受けさせる。

```text
ホストマシン
├── Claude Code
└── smolvm machine: openclaw
    └── OpenClaw Gateway
        └── localhost:18789
```

---

## 1. smolvm をインストールする

現在の `smol-machines/smolvm` を使う。

```bash
curl -sSL https://smolmachines.com/install.sh | bash
```

確認する。

```bash
smolvm --help
```

---

## 2. Smolfile を作る

作業ディレクトリを作る。

```bash
mkdir openclaw-smolvm
cd openclaw-smolvm
```

`Smolfile` を作る。

```toml
image = "node:24-bookworm"
net = true

[dev]
init = [
  "npm install -g openclaw@latest",
  "openclaw --version"
]
```

この Smolfile で指定していること：

| 設定 | 意味 |
| --- | --- |
| `image = "node:24-bookworm"` | Node.js 24 入りの Debian 系イメージを使う |
| `net = true` | `npm install -g openclaw` のためにネットワークを有効化 |
| `[dev].init` | VM 作成時に OpenClaw CLI を入れてバージョン確認する |

---

## 3. VM を作成・起動する

Smolfile から VM を作る。

```bash
smolvm machine create openclaw -s Smolfile
```

VM を起動する。

```bash
smolvm machine start --name openclaw
```

OpenClaw CLI が入っていることを確認する。

```bash
smolvm machine exec --name openclaw -- openclaw --version
```

以降の OpenClaw 操作は VM 内で実行する。まず VM に入る。

```bash
smolvm machine exec --name openclaw -it -- sh
```

---

## 4. OpenClaw の最小 config を入れる

Gateway に必要な値を `openclaw config set` で入れる。

VM 内で実行する。

```bash
openclaw config set gateway.mode local
openclaw config set gateway.port 18789 --strict-json
```

確認する。

```bash
openclaw config get gateway.mode
openclaw config get gateway.port
```

`local` と `18789` が出ればよい。

---

## 5. Gateway を起動する

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

ログに `ready` が出れば Gateway は起動している。

```text
[gateway] ready
```

`tail -f` はログを見るためだけのコマンド。`Ctrl-C` で止めても Gateway は background で動き続ける。

---

## 6. 停止・再開

Gateway だけ止める場合は VM 内で実行する。

```bash
pkill -f "openclaw gateway"
```

VM 自体を止める。

```bash
smolvm machine stop --name openclaw
```

再開する。

```bash
smolvm machine start --name openclaw
smolvm machine exec --name openclaw -it -- sh
```

VM 内で Gateway を起動する。

```bash
export OPENCLAW_GATEWAY_TOKEN=smolvm-local-token

mkdir -p /tmp/openclaw

nohup openclaw gateway run \
  --port 18789 \
  --verbose \
  > /tmp/openclaw/gateway.log 2>&1 < /dev/null &

tail -f /tmp/openclaw/gateway.log
```

不要になったら削除する。

```bash
smolvm machine delete openclaw
```

---

## ここまでできたこと

- Smolfile から `openclaw` VM を作った
- VM 内に OpenClaw CLI を入れた
- `gateway.mode=local` と `gateway.port=18789` を設定した
- OpenClaw Gateway を VM 内 `localhost:18789` で起動した

次のステップでは、この Gateway に Claude Code から MCP 接続する。
