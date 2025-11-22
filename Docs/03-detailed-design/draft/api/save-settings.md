# API詳細仕様: saveSettings

## 基本情報

- **API名**: saveSettings
- **IPCチャンネル**: `ipc:save-settings`
- **メソッド**: `ipcRenderer.invoke('ipc:save-settings', settings)`
- **バージョン**: 1.0
- **説明**: アプリケーションの設定を保存する

## リクエスト

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|------|------|
| settings | Settings | 必須 | 保存する設定 |

### Settings型

```typescript
interface Settings {
  maxPages: number;
  maxDepth: number;
  timeout: number;
  parallelRequests: number;
  retryCount: number;
  domainRestriction: 'same' | 'all';
  fileTypes: string[];
}
```

## レスポンス

**戻り値**: `Promise<void>`

## エラーレスポンス

| エラーコード | エラーメッセージ | 説明 |
|------------|----------------|------|
| VALIDATION_ERROR | "設定値が無効です" | バリデーションエラー |
| DATABASE_ERROR | "データベースエラーが発生しました" | データベース接続エラー |

## バリデーション

- maxPages: 1以上10000以下
- maxDepth: 1以上10以下
- timeout: 1000ms以上60000ms以下
- parallelRequests: 1以上20以下
- retryCount: 0以上10以下

## 関連API

- [getSettings](./get-settings.md) - 設定を取得

