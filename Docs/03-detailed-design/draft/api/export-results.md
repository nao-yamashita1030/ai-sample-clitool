# API詳細仕様: exportResults

## 基本情報

- **API名**: exportResults
- **IPCチャンネル**: `ipc:export-results`
- **メソッド**: `ipcRenderer.invoke('ipc:export-results', scanHistoryId, format, filePath)`
- **バージョン**: 1.0
- **説明**: 調査結果をCSV形式でエクスポートする

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | エクスポートする調査履歴ID |
| format | 'csv' | 必須 | エクスポート形式（現在はCSVのみ） |
| filePath | string | 必須 | 保存先ファイルパス |

## レスポンス

**戻り値**: `Promise<string>` - 保存されたファイルパス

## エラーレスポンス

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| NOT_FOUND | "調査履歴が見つかりません" | 指定した調査履歴が存在しない |
| FILE_ERROR | "ファイルの保存に失敗しました" | ファイル保存エラー |

## CSV形式

```csv
URL,リンク元URL,アンカーテキスト,HTTPステータスコード,エラーメッセージ,調査日時
https://example.com/broken,https://example.com/page1,リンクテキスト,404,,2024-01-01T00:00:00Z
```

## 関連API

- [getScanResults](./get-scan-results.md) - 調査結果を取得


