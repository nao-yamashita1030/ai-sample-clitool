# API詳細仕様: searchResults

## 基本情報

- **API名**: searchResults
- **IPCチャンネル**: `ipc:search-results`
- **メソッド**: `ipcRenderer.invoke('ipc:search-results', scanHistoryId, searchOptions)`
- **バージョン**: 1.0
- **説明**: 調査結果を検索する（getScanResultsのラッパー）

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | 調査履歴ID |
| searchOptions | SearchOptions | 必須 | 検索オプション |

### SearchOptions型

```typescript
interface SearchOptions {
  url?: string;
  sourceUrl?: string;
  anchorText?: string;
}
```

## レスポンス

**戻り値**: `Promise<LinkCheckResult[]>` - 検索結果

## 関連API

- [getScanResults](./get-scan-results.md) - 調査結果を取得（検索機能を含む）


