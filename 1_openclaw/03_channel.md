# Slack と接続する

この章では OpenClaw Gateway を Slack App と接続する。

OpenClaw の強みは、LLM を切り替えられることではなく、普段使っているチャットから Gateway 経由でエージェントを呼べることにある。ここでは Slack を入口にする。

```
Slack
  ↓ Socket Mode
OpenClaw Gateway（smolvm 内）
  ↓
LLM / tools / approvals
```

---

## 1. Slack App を作る

Slack API の App 管理画面を開く。

```text
https://api.slack.com/apps
```

`Create New App` を押して、`From a manifest` を選ぶ。

勉強会では Socket Mode を使う。外部公開 URL や ngrok が不要なので、ローカルの smolvm 上でも扱いやすい。

---

## 2. App-Level Token を作る

Slack App の `Basic Information` を開き、`App-Level Tokens` で token を作る。

必要な scope:

```text
connections:write
```

作成された `xapp-...` 形式の token を控える。

```text
SLACK_APP_TOKEN=xapp-...
```

---

## 3. Bot Token を作る

`OAuth & Permissions` を開き、Bot Token Scopes を設定する。

最小構成では、メッセージを読んで返信するための scopes を入れる。

```text
app_mentions:read
channels:history
channels:read
chat:write
groups:history
groups:read
im:history
im:read
im:write
mpim:history
mpim:read
reactions:read
reactions:write
users:read
```

設定後、`Install to Workspace` を実行する。

作成された `xoxb-...` 形式の Bot User OAuth Token を控える。

```text
SLACK_BOT_TOKEN=xoxb-...
```

---

## 4. Socket Mode と Events を有効にする

`Socket Mode` を開いて有効化する。

次に `Event Subscriptions` を開き、Bot Events を追加する。

```text
app_mention
message.channels
message.groups
message.im
message.mpim
reaction_added
reaction_removed
```

チャンネルで使う場合は、対象チャンネルに Slack App を invite する。

```text
/invite @OpenClaw
```

---

## 5. OpenClaw に Slack token を渡す

前章で作った smolvm の `openclaw` VM に入る。

```bash
smolvm machine exec --name openclaw -it -- sh
```

VM 内で Slack token と Gateway token を設定して起動する。

```bash
export SLACK_APP_TOKEN="xapp-..."
export SLACK_BOT_TOKEN="xoxb-..."
export OPENCLAW_GATEWAY_TOKEN="smolvm-local-token"

openclaw gateway \
  --port 18789 \
  --bind lan \
  --allow-unconfigured \
  --verbose
```

Gateway 起動ログに Slack channel の接続ログが出ればよい。

トークンをコマンド履歴に残したくない場合は、VM 内で `.env` 相当のファイルを作って読み込む。

```bash
smolvm machine exec --name openclaw -it -- sh
```

VM 内で実行する。

```bash
cat > ~/.openclaw-slack-env <<'EOF'
export SLACK_APP_TOKEN="xapp-..."
export SLACK_BOT_TOKEN="xoxb-..."
export OPENCLAW_GATEWAY_TOKEN="smolvm-local-token"
EOF

chmod 600 ~/.openclaw-slack-env
source ~/.openclaw-slack-env

openclaw gateway \
  --port 18789 \
  --bind lan \
  --allow-unconfigured \
  --verbose
```

---

## 6. Slack から呼び出す

DM で bot に話しかける。

```text
こんにちは。今どんなツールが使える？
```

チャンネルで使う場合は mention する。

```text
@OpenClaw 今日の議事録テンプレートを作って
```

OpenClaw が返信すれば、Slack と Gateway の接続は成功。

---

## 7. 状態確認

別ターミナルから VM 内の状態を確認する。

```bash
smolvm machine exec --name openclaw -- openclaw channels status --probe
smolvm machine exec --name openclaw -- openclaw status
```

ログを見る。

```bash
smolvm machine exec --name openclaw -- openclaw logs --follow
```

---

## LLM バックエンドについて

OpenClaw では Ollama / Anthropic / OpenAI / OpenRouter などを使えるが、この勉強会では主役にしない。

Slack 連携の観点では、重要なのは次の分離。

| 関心 | 役割 |
| --- | --- |
| Slack | ユーザーの入口 |
| OpenClaw Gateway | 会話、認可、ルーティング、チャンネル管理 |
| LLM バックエンド | 応答生成や推論 |
| smolvm | Gateway と実行環境の隔離 |

モデルを切り替えても、Slack 側の使い方は大きく変わらない。OpenClaw の価値は、チャット入口と Gateway 運用を共通化できるところにある。

---

## トラブルシューティング

### Slack から返事がない

まず状態を見る。

```bash
smolvm machine exec --name openclaw -- openclaw channels status --probe
```

確認すること：

- `SLACK_APP_TOKEN` が `xapp-...` で始まっている
- `SLACK_BOT_TOKEN` が `xoxb-...` で始まっている
- Socket Mode が有効
- Bot Events が登録されている
- チャンネルに bot を invite している

### チャンネルでは反応しないが DM では反応する

チャンネルに app が入っていないか、mention が必要な設定になっている可能性が高い。

```text
/invite @OpenClaw
```

その上で `@OpenClaw ...` と mention して試す。

### token を資料に残したくない

勉強会資料やリポジトリには token を書かない。参加者ごとに Slack App を作るか、講師側で一時 workspace / 一時 app を用意する。

---

## ここまでできたこと

- Slack App を作った
- Socket Mode で OpenClaw Gateway と接続した
- Slack DM / channel から OpenClaw を呼べるようにした

次のステップでは、この Gateway をチームで共有する構成にする。
