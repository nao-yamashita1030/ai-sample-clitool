# API詳細仕様: getScanHistory

## 基本情報

- **API名**: getScanHistory
- **IPCチャンネル**: `ipc:get-scan-history`
- **メソッド**: `ipcRenderer.invoke('ipc:get-scan-history', options)`
- **バージョン**: 1.0
- **説明**: 調査履歴の一覧を取得する

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| options | GetHistoryOptions | 任意 | 取得オプション |

### GetHistoryOptions型

```typescript
interface GetHistoryOptions {
  limit?: number;              // 取得件数（デフォルト: 50）
  offset?: number;              // オフセット（デフォルト: 0）
  sort?: 'startedAt' | 'createdAt'; // ソートフィールド（デフォルト: 'startedAt'）
  order?: 'asc' | 'desc';      // ソート順（デフォルト: 'desc'）
}
```

## レスポンス

**戻り値**: `Promise<ScanHistory[]>`

```typescript
interface ScanHistory {
  id: number;
  startUrl: string;
  startedAt: string;
  completedAt: string | null;
  status: 'running' | 'paused' | 'stopped' | 'completed' | 'error';
  totalPages: number;
  brokenLinksCount: number;
  settingsJson: string;
  createdAt: string;
  updatedAt: string;
}
```

## 関連API

- [getScanResults](./get-scan-results.md) - 調査結果を取得

