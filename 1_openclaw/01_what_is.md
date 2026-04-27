# OpenClaw と smolvm とは

この勉強会では、OpenClaw を **Claude Code から使う MCP Gateway** として扱う。

主に見る強みは 2 つ。

1. Claude Code から MCP 経由で OpenClaw Gateway に接続できる
3. Gateway をチームで共有できる

---

## smolvm

`smol-machines/smolvm` は、ローカルで軽量な Linux VM を起動する CLI ツール。

```text
コンテナ
  ↓ 共有カーネル

smolvm
  ↓ workload ごとに Linux VM
```

OpenClaw Gateway をホストマシンに直接入れず、隔離された VM の中で動かすために使う。

### 特徴

| 機能 | 説明 |
| --- | --- |
| 高速起動 | 小さな VM を短時間で起動できる |
| ハードウェア分離 | コンテナより強い境界で workload を分ける |
| OCI image 対応 | `node:24-bookworm` などの image を使える |
| Smolfile | VM の image / network / init を 1 ファイルで宣言できる |
| 永続 VM | stop / start してもインストール済みツールを残せる |

### この勉強会での使い方

```text
Smolfile
  ↓
smolvm machine: openclaw
  ↓
Node.js + OpenClaw Gateway
```

---

## OpenClaw

OpenClaw は、MCP client やチャットアプリから接続できるセルフホスト型 Gateway。

この資料では Claude Code から接続する。

```text
Claude Code
  ↓ MCP
openclaw mcp serve
  ↓ WebSocket
OpenClaw Gateway
```

LLM は Claude Code 側の Claude を使う。OpenClaw は LLM バックエンドではなく、Gateway と MCP bridge の役割を持つ。

---

## 強み 1: Claude Code から MCP で使える

Claude Code は MCP client として `openclaw mcp serve` を起動する。

```text
Claude Code
  └── openclaw mcp serve
        └── OpenClaw Gateway
```

Claude Code から見ると、OpenClaw は MCP tools を提供する server になる。

OpenClaw Gateway から見ると、Claude Code は Gateway に接続してくる client の 1 つになる。

---

## 強み 3: Gateway をチームで共有できる

OpenClaw Gateway を 1 つ建てると、複数人が同じ Gateway に接続できる。

```text
Developer A: Claude Code
Developer B: Claude Code
Developer C: Claude Code
        ↓
Shared OpenClaw Gateway
```

チーム利用で重要になるのは、モデル切り替えよりも次の点。

- 誰が Gateway に接続できるか
- Gateway token をどこで管理するか
- どの操作に承認が必要か
- Gateway をどう監視するか
- smolvm の VM をどう再起動・更新するか

---

## smolvm + OpenClaw + Claude Code の組み合わせ

勉強会では OpenClaw Gateway を smolvm の中で動かし、Claude Code から MCP で接続する。

```text
ホストマシン
├── Claude Code
│   └── openclaw mcp serve
└── smolvm
    └── OpenClaw Gateway
        └── tools / approvals
```

この構成なら、Claude Code の LLM 体験をそのまま使いながら、OpenClaw Gateway と実行環境をホストから分けて試せる。
