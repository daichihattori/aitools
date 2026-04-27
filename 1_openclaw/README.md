# OpenClaw 勉強会

smolvm 上で OpenClaw Gateway を動かし、Claude Code から MCP 経由で接続する。

## 全体構成図

![OpenClaw と smolvm の構成図](architecture.svg)

D2 ソース: [architecture.d2](architecture.d2)

## 構成

| ファイル | 内容 |
| --- | --- |
| [01_what_is.md](01_what_is.md) | OpenClaw と smolvm の全体像 |
| [02_setup.md](02_setup.md) | smol-machines/smolvm + Smolfile で Gateway を起動 |
| [03_mcp.md](03_mcp.md) | Claude Code から OpenClaw Gateway に MCP 接続 |
| [04_team.md](04_team.md) | チームで共有する Gateway の考え方 |

## 進め方

1. まず [01_what_is.md](01_what_is.md) で全体像を掴む
2. [02_setup.md](02_setup.md) で smolvm + OpenClaw Gateway をセットアップ
3. [03_mcp.md](03_mcp.md) で Claude Code から MCP 接続する
4. [04_team.md](04_team.md) で共有 Gateway と運用を整理する

## 必要なもの

- macOS または Linux
- smol-machines/smolvm
- Claude Code
- Node.js 22.14 以上（ホスト側で OpenClaw MCP bridge を動かす場合）
- OpenClaw CLI
