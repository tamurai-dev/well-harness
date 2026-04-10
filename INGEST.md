# INGEST.md

各種ファイル形式（PDF, Word, Excel, PowerPoint, 画像, 音声, YouTube URL等）を  
**カスタムパーサー不要・レイアウト破損なし・テキスト混乱なし**で  
クリーンなMarkdownに変換するためのルールです。

---

## ツール

[Microsoft MarkItDown](https://github.com/microsoft/markitdown)（MIT License）を使用する。

- **バージョン**: v0.1.5以上（`pip install 'markitdown[all]'`）
- **Python**: 3.10以上必須
- **MCP連携**: `markitdown-mcp` でClaude Desktop等のLLMアプリと統合可能

---

## インストール

### 全機能インストール（推奨）

```bash
pip install 'markitdown[all]'
```

### 必要な機能のみインストール

```bash
# 例: PDF, Word, PowerPointのみ
pip install 'markitdown[pdf, docx, pptx]'
```

### 利用可能なオプション依存

| オプション | 対応形式 |
|-----------|---------|
| `[all]` | 全形式 |
| `[pdf]` | PDF |
| `[docx]` | Word (.docx) |
| `[pptx]` | PowerPoint (.pptx) |
| `[xlsx]` | Excel (.xlsx) |
| `[xls]` | 旧Excel (.xls) |
| `[outlook]` | Outlook (.msg) |
| `[audio-transcription]` | 音声ファイル (.wav, .mp3) |
| `[youtube-transcription]` | YouTube URL |
| `[az-doc-intel]` | Azure Document Intelligence |

---

## 対応ファイル形式

| 形式 | 拡張子 / 入力 | 変換内容 |
|------|-------------|---------|
| PDF | `.pdf` | テキスト抽出、見出し・リスト・テーブル保持 |
| Word | `.docx` | 文書構造（見出し、段落、表）をMarkdownに変換 |
| Excel | `.xlsx`, `.xls` | 各シートをMarkdownテーブルに変換 |
| PowerPoint | `.pptx` | スライドごとに見出し＋本文として変換 |
| 画像 | `.jpg`, `.png` 等 | EXIFメタデータ抽出 + OCR（LLM連携時） |
| 音声 | `.wav`, `.mp3` | EXIFメタデータ抽出 + 音声書き起こし（LLM連携時） |
| YouTube | URL | 字幕・トランスクリプト取得 |
| HTML | `.html` | HTMLタグをMarkdownに変換 |
| テキスト系 | `.csv`, `.json`, `.xml` | 構造を保持してMarkdownに変換 |
| ZIP | `.zip` | 内部ファイルを再帰的に変換 |
| EPUB | `.epub` | 電子書籍の本文をMarkdownに変換 |

---

## 使い方

### CLI（コマンドライン）

```bash
# 基本（標準出力）
markitdown path-to-file.pdf

# ファイルに出力
markitdown path-to-file.pdf -o output.md

# パイプ入力
cat path-to-file.pdf | markitdown
```

### Python API

```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("document.pdf")
print(result.text_content)
```

### LLM連携（画像OCR・音声書き起こし）

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o")

# 画像のOCR + 説明
result = md.convert("screenshot.png")
print(result.text_content)

# 音声の書き起こし
result = md.convert("meeting.wav")
print(result.text_content)
```

### OCRプラグイン（PDF/Word/PPT内の画像テキスト抽出）

```bash
pip install markitdown-ocr
```

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.text_content)
```

### Docker

```bash
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < input.pdf > output.md
```

---

## Devinでの運用ルール

### 基本方針

1. **ユーザーからファイルが提供されたら**、まず `markitdown` で変換を試みる
2. **カスタムパーサーを自作しない** — MarkItDownが対応している形式は必ずMarkItDownを使う
3. **変換結果を確認してから次の処理に進む** — 空の出力やエラーがないかチェックする

### ファイル受領時のフロー

```
ファイル受領
  ↓
拡張子 / MIMEタイプを確認
  ↓
MarkItDownの対応形式か？
  ├─ Yes → markitdown で変換
  │         ↓
  │       出力を検証（空でない・構造が保持されている）
  │         ↓
  │       変換済みMarkdownを後続処理に渡す
  └─ No  → ユーザーに対応不可を通知、代替手段を提案
```

### 変換品質チェックリスト

変換後、以下を確認する：

- [ ] 見出し（`#`, `##`, `###`）が元の文書構造を反映している
- [ ] テーブルがMarkdownテーブル形式で正しくレンダリングされる
- [ ] リスト（箇条書き・番号付き）が保持されている
- [ ] 日本語テキストが文字化けしていない
- [ ] 空のセクションや意味不明なテキストブロックがない
- [ ] 画像の代替テキスト / キャプションが含まれている（LLM連携時）

### エラー時の対応

| エラー | 原因 | 対応 |
|--------|------|------|
| `UnsupportedFormatException` | 未対応の形式 | ユーザーに通知、手動変換を提案 |
| 出力が空 | パスワード保護、破損ファイル等 | ファイルの状態を確認、ユーザーに報告 |
| 文字化け | エンコーディング問題 | `charset_normalizer` で再試行 |
| テーブル崩れ | 複雑なセル結合 | Azure Document Intelligence の利用を検討 |

### 高品質変換が必要な場合

標準変換で品質が不足する場合、以下を段階的に試す：

1. **OCRプラグイン有効化** — `markitdown-ocr` で画像内テキストも抽出
2. **Azure Document Intelligence** — `markitdown -d -e "<endpoint>"` で高精度変換
3. **LLM後処理** — 変換後のMarkdownをLLMに渡して構造を修正

---

## 注意事項

- MarkItDownの出力は**LLM/テキスト解析パイプライン向け**に最適化されている。人間が読むための高忠実度ドキュメント変換が必要な場合は、Pandoc等の別ツールを検討する
- `convert_stream()` はバイナリファイルオブジェクトのみ対応（v0.1.0以降）。`io.StringIO` は不可
- プラグインはデフォルトで無効。使用時は `enable_plugins=True` を明示する
- YouTube URLの変換には字幕データが必要。字幕がない動画は変換できない
