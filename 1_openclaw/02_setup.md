# インストールとセットアップ

この章では `smol-machines/smolvm` で OpenClaw Gateway 用の VM を作る。

ポイントは **Smolfile 1 枚で環境を宣言する**こと。参加者は同じ Smolfile から同じ Node.js + OpenClaw 環境を起動できる。

```
ホストマシン
└── smolvm machine: openclaw
    └── Node.js 24
        └── OpenClaw Gateway :18789
```

---

## 1. smolvm をインストールする

現在の `smol-machines/smolvm` を使う。

```bash
curl -sSL https://smolmachines.com/install.sh | bash
```

インストール後にコマンドが見えることを確認する。

```bash
smolvm --help
```

> macOS では Hypervisor.framework、Linux では KVM を使う。Linux の場合は `/dev/kvm` が使える環境で実行する。

---

## 2. OpenClaw 用の Smolfile を作る

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
| `net = true` | npm install と Gateway 通信のためにネットワークを有効化 |
| `[dev].init` | VM 作成時に OpenClaw CLI を入れてバージョン確認する |

> 勉強会ではまず `net = true` で進める。Slack 連携まで動いたあと、必要に応じて `allow_hosts` で外向き通信を絞る。

---

## 3. VM を作成・起動する

Smolfile から永続 VM を作る。

```bash
smolvm machine create openclaw -s Smolfile
```

VM を起動する。

```bash
smolvm machine start --name openclaw
```

中に OpenClaw が入っていることを確認する。

```bash
smolvm machine exec --name openclaw -- node --version
smolvm machine exec --name openclaw -- openclaw --version
```

以降の OpenClaw 操作は VM 内で実行する。まず VM に入る。

```bash
smolvm machine exec --name openclaw -it -- sh
```

---

## 4. OpenClaw の初期設定について

この資料では `openclaw onboard` は使わない。

`onboard` は対話式の初期設定ウィザードなので、smolvm 経由の PTY と相性が悪い環境では画面が崩れることがある。勉強会では再現性を優先し、必要な値を環境変数で渡して Gateway を直接起動する。

LLM バックエンドの API キーを使う場合は、Gateway 起動時の環境変数や Control UI で設定する。OpenClaw の主題はチャンネル連携とチーム共有なので、Ollama / Anthropic / OpenAI の切り替えは後続章で軽く触れる。

---

## 5. Gateway を起動する

Gateway は `18789` 番ポートで起動する。

ここからは VM 内で実行する。

```bash
export OPENCLAW_GATEWAY_TOKEN=smolvm-local-token

openclaw gateway \
  --port 18789 \
  --bind lan \
  --allow-unconfigured \
  --verbose
```

`--bind lan` を使うので、認証トークンを必ず設定する。勉強会では説明を簡単にするため固定値にしているが、実運用では `openclaw doctor --generate-gateway-token` などで生成した値を使う。

`Missing config. Run openclaw setup...` と出る場合は `--allow-unconfigured` が付いているか確認する。ここでは Slack 連携前の空設定で Gateway を起動するため、このオプションを付ける。

別ターミナルで VM の IP を確認する。

```bash
smolvm machine list
```

表示された IP を使って、ホスト側のブラウザで Control UI を開く。

```text
http://<VM_IP>:18789/
```

認証を求められたら、次のトークンを入力する。

```text
smolvm-local-token
```

---

## 6. 動作確認

Gateway を起動したまま、別ターミナルから状態を確認する。

```bash
smolvm machine exec --name openclaw -- openclaw gateway status
smolvm machine exec --name openclaw -- openclaw status
```

Control UI が開けて、Gateway が running になっていればセットアップ完了。

---

## 7. 停止・再開

Gateway はフォアグラウンドで起動しているので、起動中のターミナルで `Ctrl-C` すると止まる。

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

openclaw gateway \
  --port 18789 \
  --bind lan \
  --allow-unconfigured \
  --verbose
```

不要になったら削除する。

```bash
smolvm machine delete openclaw
```

---

## ここまでできたこと

- `smol-machines/smolvm` をインストールした
- Smolfile 1 枚で Node.js + OpenClaw 環境を宣言した
- OpenClaw Gateway を VM 内で起動した
- ホスト側ブラウザから Control UI に接続した

次のステップでは、この Gateway を Slack と接続する。
