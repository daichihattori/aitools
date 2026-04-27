# OpenClaw 勉強会

smolvm 上で OpenClaw Gateway を動かし、Slack からチームで使える AI エージェント入口を作る。

## 全体構成図

![OpenClaw と smolvm の構成図](architecture.svg)

D2 ソース: [architecture.d2](architecture.d2)

## 構成

| ファイル | 内容 |
| --- | --- |
| [01_what_is.md](01_what_is.md) | OpenClaw と smolvm の全体像 |
| [02_setup.md](02_setup.md) | smol-machines/smolvm + Smolfile で Gateway を起動 |
| [03_channel.md](03_channel.md) | Slack と OpenClaw Gateway を接続 |
| [04_team.md](04_team.md) | チームで共有する Gateway の考え方 |

## 進め方

1. まず [01_what_is.md](01_what_is.md) で全体像を掴む
2. [02_setup.md](02_setup.md) で smolvm + OpenClaw Gateway をセットアップ
3. [03_channel.md](03_channel.md) で Slack から呼び出す
4. [04_team.md](04_team.md) で共有 Gateway と運用を整理する

## 必要なもの

- macOS または Linux
- smol-machines/smolvm
- Slack workspace の管理権限、または検証用 workspace
- OpenClaw で使う LLM バックエンド
- 必要に応じて Anthropic / OpenAI / OpenRouter などの API key
