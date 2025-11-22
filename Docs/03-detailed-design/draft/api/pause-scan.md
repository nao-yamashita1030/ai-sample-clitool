# API詳細仕様: pauseScan

## 基本情報

- **API名**: pauseScan
- **IPCチャンネル**: `ipc:pause-scan`
- **メソッド**: `ipcRenderer.invoke('ipc:pause-scan', scanHistoryId)`
- **バージョン**: 1.0
- **説明**: 実行中のリンク切れ調査を一時停止する

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | 一時停止する調査履歴ID |

## レスポンス

**戻り値**: `Promise<void>`

## エラーレスポンス

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| NOT_FOUND | "調査履歴が見つかりません" | 指定した調査履歴が存在しない |
| INVALID_STATUS | "調査が実行中ではありません" | 調査が実行中でない |

## 処理フロー

1. 調査履歴の存在確認
2. 調査のステータスを'paused'に更新
3. 新しいリクエストの開始を停止
4. 実行中のリクエストは完了まで待機

## 関連API

- [resumeScan](./resume-scan.md) - 調査を再開
- [stopScan](./stop-scan.md) - 調査を停止

