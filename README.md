# My Dify Study Project

本プロジェクトは、[langgenius/dify](https://github.com/langgenius/dify) をベースに作成した学習用リポジトリです。

- **ベースとしたバージョン:** v0.12.0 (2026年2月時点)
- **目的:** 内部構造の理解および自身の勉強環境用に一部カスタマイズ

## プロジェクト構成

```
dify/
├── api/          # Backend API (Python / Flask)
├── web/          # Frontend (Next.js 15 / React 19 / TypeScript)
├── docker/       # Docker Compose によるデプロイ・ミドルウェア構成
├── workflow/     # Dify ワークフロー定義ファイル (DSL/YAML)
├── dev/          # 開発用スクリプト・テスト設定
├── sdks/         # SDK
├── scripts/      # ユーティリティスクリプト
└── Makefile      # 開発環境セットアップ・品質チェック用コマンド
```

## ワークフロー一覧

`workflow/` ディレクトリには、Dify の DSL (YAML) 形式でエクスポートしたワークフロー定義が格納されています。Dify の管理画面からインポートして利用できます。

### 1. 規定集チャット (`corporate-rules.yml`)

| 項目       | 内容                                                                      |
|------------|---------------------------------------------------------------------------|
| **種別**   | Advanced Chat                                                             |
| **モデル** | Gemini 2.5 Flash                                                          |
| **概要**   | 社内規定（就業規則・経費精算）に関する質問に RAG で回答するチャットボット |

**フロー:**
1. ユーザーが「就業規則」または「経費精算」を選択
2. IF/ELSE で分岐し、対応するナレッジベースを検索
3. 検索結果をコンテキストとして LLM が日本語で回答
4. 回答できない場合は「担当者へ直接確認してください」と案内

### 2. 画像認識ワークフロー (`image-analysis.yml`)

| 項目       | 内容                                                                           |
|------------|--------------------------------------------------------------------------------|
| **種別**   | Workflow                                                                       |
| **モデル** | Gemini 2.5 Flash (Vision)                                                      |
| **概要**   | マーケティング資料・広告画像を解析し、商品名・メインメッセージ等を JSON で抽出 |

**フロー:**
1. ユーザーが画像ファイルをアップロード
2. LLM (Vision) が画像を解析し、構造化出力 (JSON) で抽出
3. コード実行ノードで結果を整形
4. HTTP リクエストで外部 Webhook (n8n 等) へ送信

### 3. テーマ株おすすめくん (`stock-recommend.yml`)

| 項目       | 内容                                                                     |
|------------|--------------------------------------------------------------------------|
| **種別**   | Advanced Chat                                                            |
| **モデル** | GPT-5 Mini + Tavily Search                                               |
| **概要**   | キーワードに関連する日本株のおすすめ情報を、Web 検索結果をもとにレポート |

**フロー:**
1. ユーザーがキーワードを入力 (20文字以内)
2. コード実行ノードで文字数バリデーション
3. Tavily Search で「キーワード + 日本株 おすすめ」を検索 (finance / advanced)
4. LLM が検索結果を整理し、銘柄ごとに企業概要・関連性・注意点を出力
5. 投資助言ではなく情報提供であることを明示

## セットアップ

### 前提条件

- Docker / Docker Compose
- Node.js 20+ / pnpm
- Python 3.12+ / uv

### 1. ワンコマンドセットアップ

```bash
make dev-setup
```

これにより以下が実行されます:

1. **Docker ミドルウェア起動** — PostgreSQL, Redis, Weaviate 等のミドルウェアコンテナを起動
2. **Web 環境準備** — `web/.env` の作成と `pnpm install`
3. **API 環境準備** — `api/.env` の作成、`uv sync --dev`、DB マイグレーション

### 2. 各サービスの起動

**Backend API:**

```bash
cd api
uv run flask run --host 0.0.0.0 --port 5001
```

**Frontend Web:**

```bash
cd web
pnpm dev
```

### 3. ワークフローのインポート

1. Dify 管理画面にログイン
2. 「スタジオ」→「DSL からインポート」を選択
3. `workflow/` 内の YAML ファイルを選択してインポート
4. ナレッジベースや API キー等の環境依存設定を調整

### コード品質チェック

```bash
# Backend
make lint          # ruff format + check + import lint + dotenv lint
make type-check    # basedpyright による型チェック
make test          # ユニットテスト実行

# Frontend
cd web
pnpm lint:fix      # ESLint
pnpm type-check:tsgo
pnpm test
```

### クリーンアップ

```bash
make dev-clean     # Docker コンテナ停止 + ボリューム削除
```
