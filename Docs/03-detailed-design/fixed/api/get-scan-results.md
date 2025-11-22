# API詳細仕様: getScanResults

## 基本情報

- **API名**: getScanResults
- **IPCチャンネル**: `ipc:get-scan-results`
- **メソッド**: `ipcRenderer.invoke('ipc:get-scan-results', scanHistoryId, options)`
- **バージョン**: 1.0
- **説明**: 指定した調査履歴の調査結果を取得する

## 認証・認可

- **認証要件**: 不要（ローカル実行のみ）
- **認可要件**: 不要
- **権限**: なし

## リクエスト

### リクエストパラメータ

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| scanHistoryId | number | 必須 | 調査履歴ID |
| options | GetResultsOptions | 任意 | 取得オプション（フィルタリング、ページネーションなど） |

### GetResultsOptions型

```typescript
interface GetResultsOptions {
  filter?: {
    httpStatusCode?: number | number[];  // HTTPステータスコードでフィルタ
    hasError?: boolean;                  // エラーがあるかどうか
  };
  search?: {
    url?: string;                        // URLで検索
    sourceUrl?: string;                   // リンク元URLで検索
    anchorText?: string;                  // アンカーテキストで検索
  };
  sort?: {
    field: 'url' | 'sourceUrl' | 'checkedAt' | 'httpStatusCode';
    order: 'asc' | 'desc';
  };
  pagination?: {
    page: number;                         // ページ番号（1から開始）
    pageSize: number;                     // 1ページあたりの件数
  };
}
```

### リクエスト例

```typescript
// レンダラープロセス（React）
const results = await window.electron.ipcRenderer.invoke(
  'ipc:get-scan-results',
  1,  // scanHistoryId
  {
    filter: {
      httpStatusCode: [404, 500]
    },
    sort: {
      field: 'checkedAt',
      order: 'desc'
    },
    pagination: {
      page: 1,
      pageSize: 50
    }
  }
);
```

## レスポンス

### 成功レスポンス

**戻り値**: `Promise<GetResultsResponse>`

```typescript
interface GetResultsResponse {
  results: LinkCheckResult[];
  total: number;              // 総件数（フィルタリング前）
  filtered: number;           // フィルタリング後の件数
  page: number;               // 現在のページ番号
  pageSize: number;           // 1ページあたりの件数
  totalPages: number;         // 総ページ数
}

interface LinkCheckResult {
  id: number;
  scanHistoryId: number;
  url: string;
  sourceUrl: string;
  anchorText: string | null;
  httpStatusCode: number | null;
  errorMessage: string | null;
  checkedAt: string;          // ISO8601形式
}
```

### エラーレスポンス

**エラー時**: `Promise.reject(Error)`

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| NOT_FOUND | "調査履歴が見つかりません: {scanHistoryId}" | 指定した調査履歴が存在しない |
| DATABASE_ERROR | "データベースエラーが発生しました" | データベース接続エラー |
| INVALID_OPTIONS | "無効なオプションです" | オプションの形式が不正 |

## バリデーション

### 入力バリデーション

- **scanHistoryId**: 
  - 正の整数であること
  - 存在する調査履歴IDであること
- **options.pagination.page**: 1以上
- **options.pagination.pageSize**: 1以上1000以下
- **options.sort.field**: 定義された値のみ許可
- **options.sort.order**: 'asc'または'desc'

### ビジネスルールバリデーション

- 指定した調査履歴が存在すること
- ページ番号が総ページ数を超えていないこと

## 処理フロー

1. リクエストパラメータのバリデーション
2. 調査履歴の存在確認
3. LinkCheckResultModel.findByScanHistoryId()を呼び出し
4. フィルタリング処理（HTTPステータスコード、エラー有無など）
5. 検索処理（URL、リンク元URL、アンカーテキスト）
6. ソート処理
7. ページネーション処理
8. 結果を返却

## 使用例

### 例1: すべての結果を取得

```typescript
// レンダラープロセス
const response = await window.electron.ipcRenderer.invoke(
  'ipc:get-scan-results',
  1  // scanHistoryId
);
console.log('結果数:', response.filtered);
```

### 例2: 404エラーのみを取得

```typescript
// レンダラープロセス
const response = await window.electron.ipcRenderer.invoke(
  'ipc:get-scan-results',
  1,
  {
    filter: {
      httpStatusCode: 404
    }
  }
);
```

### 例3: ページネーション付きで取得

```typescript
// レンダラープロセス
const response = await window.electron.ipcRenderer.invoke(
  'ipc:get-scan-results',
  1,
  {
    pagination: {
      page: 1,
      pageSize: 20
    },
    sort: {
      field: 'checkedAt',
      order: 'desc'
    }
  }
);
```

### 例4: URLで検索

```typescript
// レンダラープロセス
const response = await window.electron.ipcRenderer.invoke(
  'ipc:get-scan-results',
  1,
  {
    search: {
      url: 'example.com'
    }
  }
);
```

## 注意事項

- 大量の結果がある場合、ページネーションを使用することを推奨
- フィルタリングと検索は同時に使用可能
- ソートは1つのフィールドのみ対応
- デフォルトのソート順は`checkedAt`の降順

## 関連API

- [getScanHistory](./get-scan-history.md) - 調査履歴を取得
- [filterResults](./filter-results.md) - 調査結果をフィルタリング
- [searchResults](./search-results.md) - 調査結果を検索

