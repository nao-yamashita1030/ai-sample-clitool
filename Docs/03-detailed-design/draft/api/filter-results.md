# API詳細仕様: filterResults

## 基本情報

- **API名**: filterResults
- **IPCチャンネル**: `ipc:filter-results`
- **メソッド**: `ipcRenderer.invoke('ipc:filter-results', scanHistoryId, filterOptions)`
- **バージョン**: 1.0
- **説明**: 調査結果をフィルタリングする（getScanResultsのラッパー）

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | 調査履歴ID |
| filterOptions | FilterOptions | 必須 | フィルタリングオプション |

### FilterOptions型

```typescript
interface FilterOptions {
  httpStatusCode?: number | number[];
  hasError?: boolean;
}
```

## レスポンス

**戻り値**: `Promise<LinkCheckResult[]>` - フィルタリング後の結果

## 関連API

- [getScanResults](./get-scan-results.md) - 調査結果を取得（フィルタリング機能を含む）

