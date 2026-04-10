# well-harness

Devin（AIエージェント）が読むためのルール・制約ファイルを管理するリポジトリです。

## 概要

このリポジトリは、Devinがtamurai-dev配下の各プロジェクトで実装作業を行う際に参照すべきルールやガイドラインを一元管理する場所です。  
openclaw-workspaceにおけるHEARTBEAT.mdやAGENTS.mdと同様に、エージェントが読み込むためのMarkdownファイルをリポジトリルート直下に配置します。

## ファイル構造

```
well-harness/
├── README.md          # このファイル（リポジトリの説明）
├── CONSTRAINTS.md     # Devinが実装時に守るべきルール・制約
├── INGEST.md          # ファイル→Markdown変換ルール（MarkItDown）
├── INTENT.md          # 自然言語ワークフロー宣言（Phase 0: DECLARE）
├── BLUEPRINT.md       # AI駆動の分解・設計図（Phase 1: DECOMPOSE）
└── ...                # 今後追加されるDevin向けMDファイル
```

## ファイル一覧

| ファイル | 説明 |
|---------|------|
| `CONSTRAINTS.md` | 実装上の制約・ルール。Devinはセッション開始時にこのファイルを読み、記載されたルールに従う |
| `INGEST.md` | 各種ファイル形式（PDF, Word, Excel, PPT, 音声, YouTube等）をMarkdownに変換するためのルール。Microsoft MarkItDownを使用 |
| `INTENT.md` | 人間が自然言語でフローを宣言するための起点（Phase 0: DECLARE）。品質チェック後、BLUEPRINT.mdに引き渡す |
| `BLUEPRINT.md` | AIが宣言を構造化し、スキルモード・データモデル・技術スタックに分解する設計図（Phase 1: DECOMPOSE） |

## 運用ルール

- 各MDファイルはリポジトリの**ルート直下**に配置する（openclaw-workspaceのHEARTBEAT.mdと同じ構造）
- ファイル名は大文字で、役割が明確にわかる名前にする（例: `CONSTRAINTS.md`, `INGEST.md`, `INTENT.md`, `BLUEPRINT.md`）
- Devinは作業開始時にこのリポジトリの該当ファイルを参照し、ルールに従って作業する
