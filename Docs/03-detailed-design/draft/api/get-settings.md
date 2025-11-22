# API詳細仕様: getSettings

## 基本情報

- **API名**: getSettings
- **IPCチャンネル**: `ipc:get-settings`
- **メソッド**: `ipcRenderer.invoke('ipc:get-settings')`
- **バージョン**: 1.0
- **説明**: アプリケーションの設定を取得する

## 認証・認可

- **認証要件**: 不要（ローカル実行のみ）
- **認可要件**: 不要
- **権限**: なし

## リクエスト

### リクエストパラメータ

パラメータなし

### リクエスト例

```typescript
// レンダラープロセス（React）
const settings = await window.electron.ipcRenderer.invoke('ipc:get-settings');
```

## レスポンス

### 成功レスポンス

**戻り値**: `Promise<Settings>`

```typescript
interface Settings {
  maxPages: number;          // 最大ページ数（デフォルト: 1000）
  maxDepth: number;          // 最大階層レベル（デフォルト: 5）
  timeout: number;           // タイムアウト時間（ミリ秒、デフォルト: 5000）
  parallelRequests: number; // 並列リクエスト数（デフォルト: 5）
  retryCount: number;       // リトライ回数（デフォルト: 3）
  domainRestriction: 'same' | 'all'; // ドメイン制限（デフォルト: 'same'）
  fileTypes: string[];      // 調査対象ファイルタイプ（デフォルト: ['html']）
}
```

### エラーレスポンス

**エラー時**: `Promise.reject(Error)`

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| DATABASE_ERROR | "データベースエラーが発生しました" | データベース接続エラー |

## バリデーション

### 入力バリデーション

パラメータなしのため、バリデーション不要

### ビジネスルールバリデーション

- 設定が存在しない場合、デフォルト値を返す

## 処理フロー

1. SettingsModel.getAll()を呼び出し
2. データベースから設定を取得
3. 設定が存在しない場合、デフォルト値を返す
4. 設定を返却

## 使用例

### 例1: 設定を取得して表示

```typescript
// レンダラープロセス
try {
  const settings = await window.electron.ipcRenderer.invoke('ipc:get-settings');
  console.log('最大ページ数:', settings.maxPages);
  console.log('並列リクエスト数:', settings.parallelRequests);
} catch (error) {
  console.error('エラー:', error.message);
}
```

### 例2: 設定を取得してフォームに反映

```typescript
// レンダラープロセス（React）
const [settings, setSettings] = useState<Settings | null>(null);

useEffect(() => {
  const loadSettings = async () => {
    const s = await window.electron.ipcRenderer.invoke('ipc:get-settings');
    setSettings(s);
  };
  loadSettings();
}, []);
```

## 注意事項

- 設定が存在しない場合、デフォルト値が返される
- 設定はJSON形式でデータベースに保存される
- 設定の変更は`saveSettings` APIを使用する

## 関連API

- [saveSettings](./save-settings.md) - 設定を保存


