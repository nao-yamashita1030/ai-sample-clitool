# API詳細仕様

このディレクトリには、各APIの詳細仕様を個別ファイルとして管理します。

## API一覧

| API名 | IPCチャンネル | ファイル名 | 説明 |
|------|-------------|----------|------|
| startScan | ipc:start-scan | start-scan.md | リンク切れ調査を開始 |
| stopScan | ipc:stop-scan | stop-scan.md | リンク切れ調査を停止 |
| pauseScan | ipc:pause-scan | pause-scan.md | リンク切れ調査を一時停止 |
| resumeScan | ipc:resume-scan | resume-scan.md | リンク切れ調査を再開 |
| getScanResults | ipc:get-scan-results | get-scan-results.md | 調査結果を取得 |
| getScanHistory | ipc:get-scan-history | get-scan-history.md | 調査履歴を取得 |
| exportResults | ipc:export-results | export-results.md | 調査結果をエクスポート |
| getSettings | ipc:get-settings | get-settings.md | 設定を取得 |
| saveSettings | ipc:save-settings | save-settings.md | 設定を保存 |
| filterResults | ipc:filter-results | filter-results.md | 調査結果をフィルタリング |
| searchResults | ipc:search-results | search-results.md | 調査結果を検索 |

## API設計ルール

### IPCチャンネル命名規則
- 形式: `ipc:{kebab-case-name}`
- 例: `ipc:start-scan`, `ipc:get-scan-results`
- 命名規則: 動詞 + 名詞の形式（例: start-scan, get-results）

### 非同期処理
- すべてのAPIは非同期処理（Promise）を返す
- 長時間かかる処理（startScanなど）は進捗をIPCイベントで通知

### レスポンス形式
- 成功時: データオブジェクトを返す
- エラー時: Errorオブジェクトをスロー

### エラーレスポンス形式
- エラーはErrorオブジェクトとしてスロー
- エラーメッセージは日本語で分かりやすく記述
- エラーコードは必要に応じて定義

## APIバージョニング

### バージョン管理方針
- 初期リリースではバージョニングは不要
- 将来の拡張時に必要に応じてバージョンを追加

### バージョン指定方法
- IPCチャンネル名にバージョンを含める（例: `ipc:v1:start-scan`）
- または、リクエストパラメータにバージョンを含める

## 認証・認可

### 認証方式
- 認証は不要（ローカル実行のみ）

### 認可要件
- 認可は不要（ローカル実行のみ）

## ファイル命名規則

APIファイルの命名規則：
- 機能ベース: `start-scan.md`, `get-scan-results.md`
- IPCチャンネル名と一致させる（`ipc:`プレフィックスを除く）

推奨: IPCチャンネル名と一致するファイル名を使用してください。

