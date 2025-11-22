# API詳細仕様: resumeScan

## 基本情報

- **API名**: resumeScan
- **IPCチャンネル**: `ipc:resume-scan`
- **メソッド**: `ipcRenderer.invoke('ipc:resume-scan', scanHistoryId)`
- **バージョン**: 1.0
- **説明**: 一時停止中のリンク切れ調査を再開する

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | 再開する調査履歴ID |

## レスポンス

**戻り値**: `Promise<void>`

## エラーレスポンス

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| NOT_FOUND | "調査履歴が見つかりません" | 指定した調査履歴が存在しない |
| INVALID_STATUS | "調査が一時停止中ではありません" | 調査が一時停止中でない |

## 処理フロー

1. 調査履歴の存在確認
2. 調査のステータスを'running'に更新
3. 並列処理を再開
4. キューからURLを取得して処理を継続

## 関連API

- [pauseScan](./pause-scan.md) - 調査を一時停止
- [stopScan](./stop-scan.md) - 調査を停止


