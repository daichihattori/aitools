# チームで共有する Gateway を建てる

この章では、個人のローカル実験から一歩進めて、チームで共有する OpenClaw Gateway の考え方を整理する。

Slack 連携までできると、OpenClaw は「個人の CLI ツール」ではなく「チームのチャットに常駐する Gateway」になる。

```
Team Slack
  ↓
Shared OpenClaw Gateway
  ↓
Agents / tools / approvals
  ↓
smolvm sandbox
```

---

## 1. 共有 Gateway で変わること

個人利用とチーム利用では、見るべきポイントが変わる。

| 観点 | 個人利用 | チーム共有 |
| --- | --- | --- |
| 認証 | 自分だけ | Slack ユーザー、チャンネル、approver |
| Gateway | localhost で十分 | 常時起動、再起動、ログ確認が必要 |
| token | 手元の環境変数 | Secret 管理が必要 |
| チャンネル | DM 中心 | public / private channel と thread |
| 実行 | 試行錯誤 | 承認、監査、権限分離 |

---

## 2. 最小の共有構成

勉強会では、まず次の構成を目標にする。

```text
Slack workspace
  └── #openclaw-lab
      └── OpenClaw bot

講師または共有マシン
  └── smolvm machine: openclaw
      └── OpenClaw Gateway
```

Gateway は外部に広く公開しない。Slack Socket Mode を使えば、Slack 側から Gateway に直接 HTTP 到達できる必要はない。

Control UI を開く場合も、LAN / VPN / Tailscale など信頼できる範囲に閉じる。

---

## 3. Slack の権限設計

最初はチャンネルを限定する。

```text
#openclaw-lab
```

運用ルール：

- bot を invite するチャンネルを限定する
- チャンネルでは mention 必須にする
- DM は pairing または allowlist にする
- 実行系の操作は approval を挟む
- token を Slack や資料に貼らない

この章の主題は「何でも自動化できる」ではなく、「チームが安全に使える入口を作る」こと。

---

## 4. Gateway の起動例

勉強会用の簡易起動。

```bash
smolvm machine exec --name openclaw -it -- sh -lc '
  source ~/.openclaw-slack-env
  openclaw gateway --port 18789 --bind lan --verbose
'
```

実運用では固定 token の直書きを避け、ホスト側の secret 管理、systemd / launchd / supervisor、Tailscale などを組み合わせる。

---

## 5. 運用チェック

Gateway の健康状態。

```bash
smolvm machine exec --name openclaw -- openclaw status
smolvm machine exec --name openclaw -- openclaw gateway status
```

チャンネル状態。

```bash
smolvm machine exec --name openclaw -- openclaw channels status --probe
```

ログ。

```bash
smolvm machine exec --name openclaw -- openclaw logs --follow
```

---

## 6. チーム導入時の注意点

### Gateway を公開しすぎない

`--bind lan` や remote access を使う場合は、必ず token / password / VPN / firewall を組み合わせる。

Slack Socket Mode だけなら、Gateway の HTTP endpoint をインターネットに公開する必要はない。

### Slack の文脈は信頼しすぎない

チャンネル名、トピック、過去ログ、添付ファイルはすべてプロンプト注入の入口になり得る。

エージェントに実行権限を与える場合は、approval と allowlist を前提にする。

### token を分ける

勉強会用、検証用、本番用の Slack App / Gateway token / LLM API key は分ける。

同じ token を複数環境で使い回さない。

---

## まとめ

OpenClaw のチーム利用で重要なのは、モデル選択よりも Gateway の共有方法と権限設計。

- Slack を入口にする
- Gateway をチームで共有する
- smolvm で実行環境を隔離する
- token と実行権限を分ける
- approval を前提にする

この形にすると、OpenClaw は「各自がローカルで動かす実験ツール」から「チームの作業入口」になる。
