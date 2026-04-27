# OpenClaw と smolvm とは

この勉強会では、OpenClaw を「LLM を切り替えるツール」としてではなく、**チャットと AI エージェントをつなぐ Gateway** として扱う。

主に見る強みは 2 つ。

1. Slack などのチャンネルからエージェントを呼べる
3. チームで共有する Gateway を建てられる

---

## smolvm

`smol-machines/smolvm` は、ローカルで軽量な Linux VM を起動する CLI ツール。

```
コンテナ
  ↓ 共有カーネル

smolvm
  ↓ workload ごとに Linux VM
```

OpenClaw をホストマシンに直接入れず、隔離された VM の中で動かすために使う。

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

OpenClaw は、チャットアプリと AI エージェントをつなぐセルフホスト型 Gateway。

```text
Slack
  ↓
OpenClaw Gateway
  ↓
LLM / tools / approvals
```

Slack から話しかけると、OpenClaw Gateway が会話、権限、ルーティング、ツール実行を扱う。

---

## 強み 1: Slack から使える

AI エージェントを専用 CLI や専用 UI に閉じず、普段のチームチャットから呼べる。

```text
#openclaw-lab
@OpenClaw リリース前チェックリストを作って
```

Slack を入口にすると、チームメンバーは同じ場所で依頼、確認、承認、結果共有ができる。

LLM バックエンドが Ollama でも Anthropic / OpenAI でも、Slack 側の体験は大きく変わらない。

---

## 強み 3: Gateway をチームで共有できる

OpenClaw Gateway を 1 つ建てると、複数人が同じ入口からエージェントを使える。

```text
Team Slack
  ↓
Shared OpenClaw Gateway
  ↓
Agents / tools / approvals
```

チーム利用で重要になるのは、モデル切り替えよりも次の点。

- 誰が使えるか
- どのチャンネルで反応するか
- どの操作に承認が必要か
- token をどこで管理するか
- Gateway をどう監視するか

---

## smolvm + OpenClaw の組み合わせ

OpenClaw は便利なぶん、チャット、LLM、ツール実行、外部 API をつなぐ強い権限を持つ。

そこで、勉強会では OpenClaw Gateway を smolvm の中で動かす。

```text
ホストマシン
└── smolvm
    └── OpenClaw Gateway
        ├── Slack channel
        ├── LLM backend
        └── tools / approvals
```

この構成なら、Gateway と実行環境をホストから分けて試せる。まず安全に小さく動かし、その後 Slack 連携とチーム共有に進む。
