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
└── ...                # 今後追加されるDevin向けMDファイル
```

## ファイル一覧

| ファイル | 説明 |
|---------|------|
| `CONSTRAINTS.md` | 実装上の制約・ルール。Devinはセッション開始時にこのファイルを読み、記載されたルールに従う |

## 運用ルール

- 各MDファイルはリポジトリの**ルート直下**に配置する（openclaw-workspaceのHEARTBEAT.mdと同じ構造）
- ファイル名は大文字で、役割が明確にわかる名前にする（例: `CONSTRAINTS.md`, `GUIDELINES.md`）
- Devinは作業開始時にこのリポジトリの該当ファイルを参照し、ルールに従って作業する
