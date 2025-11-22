# API詳細仕様: stopScan

## 基本情報

- **API名**: stopScan
- **IPCチャンネル**: `ipc:stop-scan`
- **メソッド**: `ipcRenderer.invoke('ipc:stop-scan', scanHistoryId)`
- **バージョン**: 1.0
- **説明**: 実行中のリンク切れ調査を停止する

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | 停止する調査履歴ID |

## レスポンス

**戻り値**: `Promise<void>`

## エラーレスポンス

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| NOT_FOUND | "調査履歴が見つかりません" | 指定した調査履歴が存在しない |
| INVALID_STATUS | "調査が実行中ではありません" | 調査が実行中でない |

## 処理フロー

1. 調査履歴の存在確認
2. 調査のステータスを'stopped'に更新
3. 実行中のリクエストをキャンセル
4. 処理を停止

## 関連API

- [startScan](./start-scan.md) - 調査を開始
- [pauseScan](./pause-scan.md) - 調査を一時停止


