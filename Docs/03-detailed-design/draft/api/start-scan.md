# API詳細仕様: startScan

## 基本情報

- **API名**: startScan
- **IPCチャンネル**: `ipc:start-scan`
- **メソッド**: `ipcRenderer.invoke('ipc:start-scan', url, settings)`
- **バージョン**: 1.0
- **説明**: リンク切れ調査を開始する

## 認証・認可

- **認証要件**: 不要（ローカル実行のみ）
- **認可要件**: 不要
- **権限**: なし

## リクエスト

### リクエストパラメータ

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| url | string | 必須 | 調査を開始するURL |
| settings | ScanSettings | 任意 | 調査設定（省略時はデフォルト設定を使用） |

### ScanSettings型

```typescript
interface ScanSettings {
  maxPages?: number;          // 最大ページ数（デフォルト: 1000）
  maxDepth?: number;          // 最大階層レベル（デフォルト: 5）
  timeout?: number;           // タイムアウト時間（ミリ秒、デフォルト: 5000）
  parallelRequests?: number; // 並列リクエスト数（デフォルト: 5）
  retryCount?: number;       // リトライ回数（デフォルト: 3）
  domainRestriction?: 'same' | 'all'; // ドメイン制限（デフォルト: 'same'）
  fileTypes?: string[];      // 調査対象ファイルタイプ（デフォルト: ['html']）
}
```

### リクエスト例

```typescript
// レンダラープロセス（React）
const scanHistory = await window.electron.ipcRenderer.invoke(
  'ipc:start-scan',
  'https://example.com',
  {
    maxPages: 500,
    maxDepth: 3,
    timeout: 10000,
    parallelRequests: 10,
    retryCount: 2,
    domainRestriction: 'same',
    fileTypes: ['html', 'htm']
  }
);
```

## レスポンス

### 成功レスポンス

**戻り値**: `Promise<ScanHistory>`

```typescript
interface ScanHistory {
  id: number;
  startUrl: string;
  startedAt: string;        // ISO8601形式
  completedAt: string | null;
  status: 'running' | 'paused' | 'stopped' | 'completed' | 'error';
  totalPages: number;
  brokenLinksCount: number;
  settingsJson: string;     // JSON文字列
  createdAt: string;         // ISO8601形式
  updatedAt: string;         // ISO8601形式
}
```

### エラーレスポンス

**エラー時**: `Promise.reject(Error)`

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| INVALID_URL | "無効なURLです: {url}" | URL形式が不正 |
| ALREADY_RUNNING | "既に調査が実行中です" | 他の調査が実行中 |
| DATABASE_ERROR | "データベースエラーが発生しました" | データベース接続エラー |

## バリデーション

### 入力バリデーション

- **url**: 
  - URL形式が正しいか（正規表現でチェック）
  - プロトコルがhttpまたはhttpsか
  - ドメイン部分が存在するか
  - URLの長さが2048文字以内か
- **settings.maxPages**: 1以上10000以下
- **settings.maxDepth**: 1以上10以下
- **settings.timeout**: 1000ms以上60000ms以下
- **settings.parallelRequests**: 1以上20以下
- **settings.retryCount**: 0以上10以下

### ビジネスルールバリデーション

- 現在実行中の調査がないこと
- URLがアクセス可能であること（オプション、事前チェック）

## 処理フロー

1. リクエストパラメータのバリデーション
2. 現在実行中の調査がないかチェック
3. 調査履歴レコードを作成（status: 'running'）
4. ScanController.startScan()を呼び出し
5. 調査を非同期で開始
6. 調査履歴レコードを返却
7. 進捗は`ipc:scan-progress`イベントで通知

## 進捗通知

調査の進捗は`ipc:scan-progress`イベントで通知されます：

```typescript
// レンダラープロセス
window.electron.ipcRenderer.on('ipc:scan-progress', (event, progress) => {
  console.log('進捗:', progress);
});

interface Progress {
  scanHistoryId: number;
  totalPages: number;
  checkedPages: number;
  brokenLinksCount: number;
  percentage: number;
  currentUrl: string | null;
}
```

## 使用例

### 例1: 基本的な使用

```typescript
// レンダラープロセス
try {
  const scanHistory = await window.electron.ipcRenderer.invoke(
    'ipc:start-scan',
    'https://example.com'
  );
  console.log('調査開始:', scanHistory.id);
} catch (error) {
  console.error('エラー:', error.message);
}
```

### 例2: カスタム設定で開始

```typescript
// レンダラープロセス
const settings = {
  maxPages: 2000,
  maxDepth: 7,
  timeout: 15000,
  parallelRequests: 8,
  retryCount: 5,
  domainRestriction: 'all',
  fileTypes: ['html', 'htm', 'php']
};

const scanHistory = await window.electron.ipcRenderer.invoke(
  'ipc:start-scan',
  'https://example.com',
  settings
);
```

## 注意事項

- 調査は非同期で実行されるため、このAPIは即座に返却される
- 進捗は`ipc:scan-progress`イベントで通知される
- 調査を停止する場合は`stopScan` APIを使用する
- 調査を一時停止する場合は`pauseScan` APIを使用する
- 同じURLで複数の調査を同時に実行することはできない

## 関連API

- [stopScan](./stop-scan.md) - 調査を停止
- [pauseScan](./pause-scan.md) - 調査を一時停止
- [resumeScan](./resume-scan.md) - 調査を再開
- [getScanResults](./get-scan-results.md) - 調査結果を取得


