# example App (Flask, Vanilla JS)

ローカルHTTP環境で使うexample Webアプリです。

## 構成

- フロントエンド: Vanilla JS + Semantic UI
- バックエンド: Flask (Blueprint構成)
- exampleモデル: Hugging Face 

## ディレクトリ

- `app/__init__.py`: `create_app()` とロギング設定
- `app/routes/web.py`: 画面表示Blueprint
- `app/routes/api.py`: example API Blueprint
- `app/services/example_service.py`: DeepSeek example実行ロジック
- `app/templates/index.html`: 2カラムUI
- `app/static/app.js`: 貼り付け/実行/表示/コピー/保存

## 事前準備

1. `uv` をインストール
2. Python 3.11 以上を利用

## セットアップ (uv)

```bash
uv sync
```

## 起動

```bash
uv run python run.py
```

ブラウザで `http://127.0.0.1:5000` を開いて利用します。

## モデル保存と再DL防止

- モデル保存先: `./models/example`
- 初回起動時にモデルがない場合だけDL
- 2回目以降はローカルフォルダから読み込み

## API

### `POST /api/example`

- form-data の `image` に画像ファイルを添付
- レスポンス例:

```json
{
  "ok": true,
  "model": "example",
  "raw_result": {},
  "text": "..."
}
```

### `GET /api/health`

```json
{
  "ok": true,
  "model_loaded": false
}
```

## ログ

- 出力先: `logs/app.log`
- ローテーション: 5MB x 3世代



