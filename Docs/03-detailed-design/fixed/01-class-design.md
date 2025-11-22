# クラス設計

## 1.1 クラス図

```
┌─────────────────────────────────────────────────────────┐
│                   レンダラープロセス（React）              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ MainScreen   │  │ ResultsScreen│  │ SettingsScreen│ │
│  │ Component    │  │ Component    │  │ Component    │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
│         │                  │                  │          │
│         └──────────────────┼──────────────────┘          │
│                            │                             │
│                    ┌───────▼────────┐                    │
│                    │  IPC Client     │                    │
│                    │  (Jotai Atoms) │                    │
│                    └───────┬────────┘                    │
└────────────────────────────┼─────────────────────────────┘
                             │ IPC
┌────────────────────────────▼─────────────────────────────┐
│                 メインプロセス（Node.js）                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │              IPC Handler                         │  │
│  │  - handleStartScan()                             │  │
│  │  - handleStopScan()                              │  │
│  │  - handlePauseScan()                             │  │
│  │  - handleResumeScan()                            │  │
│  │  - handleGetScanResults()                        │  │
│  │  - handleGetScanHistory()                       │  │
│  │  - handleExportResults()                         │  │
│  │  - handleGetSettings()                           │  │
│  │  - handleSaveSettings()                          │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │            ScanController                        │  │
│  │  - startScan(url: string, settings: Settings)    │  │
│  │  - stopScan()                                    │  │
│  │  - pauseScan()                                   │  │
│  │  - resumeScan()                                  │  │
│  │  - getProgress(): Progress                        │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         LinkChecker                              │  │
│  │  - checkLink(url: string): Promise<CheckResult>  │  │
│  │  - extractLinks(html: string): string[]         │  │
│  │  - isValidUrl(url: string): boolean              │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         HttpClient                               │  │
│  │  - get(url: string): Promise<Response>           │  │
│  │  - getWithRetry(url: string, retries: number)   │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         ScanHistoryModel                         │  │
│  │  - create(data: ScanHistoryData)                │  │
│  │  - findById(id: number)                         │  │
│  │  - findAll(options: FindOptions)                │  │
│  │  - update(id: number, data: Partial<...>)        │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         LinkCheckResultModel                     │  │
│  │  - create(data: LinkCheckResultData)             │  │
│  │  - findByScanHistoryId(id: number)               │  │
│  │  - findById(id: number)                          │  │
│  │  - filter(options: FilterOptions)                │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         SettingsModel                            │  │
│  │  - get(key: string): Promise<string | null>      │  │
│  │  - set(key: string, value: string)               │  │
│  │  - getAll(): Promise<Settings>                   │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │         Database (ORM)                          │  │
│  │  - Sequelize / TypeORM                          │  │
│  └──────────────────┬───────────────────────────────┘  │
│                     │                                   │
│  ┌──────────────────▼───────────────────────────────┐  │
│  │              SQLite Database                     │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## 1.2 主要クラス設計

### クラス名: ScanController

- **責務**: リンク切れ調査の制御と並列処理の管理
- **属性**:
  - `currentScan: ScanState | null` - 現在の調査状態
  - `urlQueue: Queue<string>` - 調査待ちURLキュー
  - `visitedUrls: Set<string>` - 調査済みURLセット
  - `parallelLimit: number` - 並列リクエスト数
  - `settings: ScanSettings` - 調査設定
- **メソッド**:
  - `startScan(url: string, settings: ScanSettings): Promise<void>` - 調査を開始
  - `stopScan(): Promise<void>` - 調査を停止
  - `pauseScan(): Promise<void>` - 調査を一時停止
  - `resumeScan(): Promise<void>` - 調査を再開
  - `getProgress(): Progress` - 進捗状況を取得
  - `private processUrl(url: string): Promise<void>` - 単一URLの処理
  - `private addLinksToQueue(links: string[], sourceUrl: string): void` - リンクをキューに追加
- **関係性**: 
  - LinkCheckerを使用してリンクをチェック
  - ScanHistoryModel、LinkCheckResultModelを使用してデータを保存
  - HttpClientを使用してHTTPリクエストを送信

### クラス名: LinkChecker

- **責務**: リンクのチェックとHTML解析
- **属性**:
  - `httpClient: HttpClient` - HTTPクライアント
  - `timeout: number` - タイムアウト時間
- **メソッド**:
  - `checkLink(url: string, sourceUrl: string, anchorText?: string): Promise<LinkCheckResult>` - リンクをチェック
  - `extractLinks(html: string, baseUrl: string): LinkInfo[]` - HTMLからリンクを抽出
  - `isValidUrl(url: string): boolean` - URLの妥当性チェック
  - `normalizeUrl(url: string, baseUrl: string): string` - URLを正規化
  - `shouldCheckUrl(url: string, settings: ScanSettings): boolean` - URLを調査対象とするか判定
- **関係性**: 
  - HttpClientを使用してHTTPリクエストを送信

### クラス名: HttpClient

- **責務**: HTTPリクエストの送信とエラーハンドリング
- **属性**:
  - `timeout: number` - タイムアウト時間（ミリ秒）
  - `retryCount: number` - リトライ回数
  - `userAgent: string` - User-Agent文字列
- **メソッド**:
  - `get(url: string): Promise<HttpResponse>` - GETリクエストを送信
  - `getWithRetry(url: string, retries: number): Promise<HttpResponse>` - リトライ付きGETリクエスト
  - `private handleError(error: Error): HttpError` - エラーを処理
- **関係性**: 
  - axiosライブラリを使用してHTTPリクエストを送信

### クラス名: ScanHistoryModel

- **責務**: 調査履歴のデータアクセス
- **属性**:
  - `model: SequelizeModel | TypeOrmEntity` - ORMモデル
- **メソッド**:
  - `create(data: ScanHistoryData): Promise<ScanHistory>` - 調査履歴を作成
  - `findById(id: number): Promise<ScanHistory | null>` - IDで検索
  - `findAll(options?: FindOptions): Promise<ScanHistory[]>` - 全件取得
  - `update(id: number, data: Partial<ScanHistoryData>): Promise<ScanHistory>` - 更新
  - `delete(id: number): Promise<void>` - 削除
- **関係性**: 
  - ORM（Sequelize/TypeORM）を使用してデータベースにアクセス

### クラス名: LinkCheckResultModel

- **責務**: 調査結果のデータアクセス
- **属性**:
  - `model: SequelizeModel | TypeOrmEntity` - ORMモデル
- **メソッド**:
  - `create(data: LinkCheckResultData): Promise<LinkCheckResult>` - 調査結果を作成
  - `findByScanHistoryId(scanHistoryId: number, options?: FindOptions): Promise<LinkCheckResult[]>` - 調査履歴IDで検索
  - `findById(id: number): Promise<LinkCheckResult | null>` - IDで検索
  - `filter(options: FilterOptions): Promise<LinkCheckResult[]>` - フィルタリング
  - `countByScanHistoryId(scanHistoryId: number): Promise<number>` - 件数を取得
- **関係性**: 
  - ORM（Sequelize/TypeORM）を使用してデータベースにアクセス

### クラス名: SettingsModel

- **責務**: アプリケーション設定のデータアクセス
- **属性**:
  - `model: SequelizeModel | TypeOrmEntity` - ORMモデル
- **メソッド**:
  - `get(key: string): Promise<string | null>` - 設定値を取得
  - `set(key: string, value: string): Promise<void>` - 設定値を保存
  - `getAll(): Promise<Settings>` - 全設定を取得
  - `delete(key: string): Promise<void>` - 設定を削除
- **関係性**: 
  - ORM（Sequelize/TypeORM）を使用してデータベースにアクセス

### クラス名: IPC Handler

- **責務**: レンダラープロセスとメインプロセス間のIPC通信を処理
- **属性**:
  - `scanController: ScanController` - スキャンコントローラー
  - `scanHistoryModel: ScanHistoryModel` - 調査履歴モデル
  - `linkCheckResultModel: LinkCheckResultModel` - 調査結果モデル
  - `settingsModel: SettingsModel` - 設定モデル
- **メソッド**:
  - `handleStartScan(event: IpcMainInvokeEvent, url: string, settings: Settings): Promise<ScanHistory>` - 調査開始
  - `handleStopScan(event: IpcMainInvokeEvent, scanHistoryId: number): Promise<void>` - 調査停止
  - `handlePauseScan(event: IpcMainInvokeEvent, scanHistoryId: number): Promise<void>` - 調査一時停止
  - `handleResumeScan(event: IpcMainInvokeEvent, scanHistoryId: number): Promise<void>` - 調査再開
  - `handleGetScanResults(event: IpcMainInvokeEvent, scanHistoryId: number, options?: FilterOptions): Promise<LinkCheckResult[]>` - 調査結果取得
  - `handleGetScanHistory(event: IpcMainInvokeEvent, options?: FindOptions): Promise<ScanHistory[]>` - 調査履歴取得
  - `handleExportResults(event: IpcMainInvokeEvent, scanHistoryId: number, format: 'csv'): Promise<string>` - 結果エクスポート
  - `handleGetSettings(event: IpcMainInvokeEvent): Promise<Settings>` - 設定取得
  - `handleSaveSettings(event: IpcMainInvokeEvent, settings: Settings): Promise<void>` - 設定保存
- **関係性**: 
  - ScanController、各Modelクラスを使用

## 1.3 クラス間の関係

### 依存関係

- **IPC Handler** → **ScanController**: 調査制御の委譲
- **IPC Handler** → **ScanHistoryModel, LinkCheckResultModel, SettingsModel**: データアクセスの委譲
- **ScanController** → **LinkChecker**: リンクチェックの委譲
- **ScanController** → **ScanHistoryModel, LinkCheckResultModel**: データ保存の委譲
- **LinkChecker** → **HttpClient**: HTTPリクエストの委譲
- **各Model** → **ORM (Sequelize/TypeORM)**: データベースアクセスの委譲

### 集約関係

- **ScanController** は **LinkChecker** と **HttpClient** を集約
- **IPC Handler** は **ScanController** と各 **Model** を集約

### 関連関係

- **ScanHistory** と **LinkCheckResult** は1対多の関係（データベースレベル）

## 1.4 デザインパターン

### パターン名: MVC（Model-View-Controller）

- **使用箇所**: アプリケーション全体のアーキテクチャ
- **理由**: 
  - 責務の分離を明確にする
  - テスト容易性を向上させる
  - 保守性を向上させる
- **実装方法**: 
  - **Model**: ScanHistoryModel, LinkCheckResultModel, SettingsModel（データアクセス層）
  - **View**: Reactコンポーネント（レンダラープロセス）
  - **Controller**: ScanController（ビジネスロジック層）

### パターン名: Repository パターン

- **使用箇所**: データアクセス層（各Modelクラス）
- **理由**: 
  - データアクセスロジックをカプセル化
  - データソース（SQLite）への依存を抽象化
  - テスト時にモックに置き換え可能
- **実装方法**: 
  - 各ModelクラスがRepositoryの役割を担う
  - ORMを使用してデータベースアクセスを抽象化

### パターン名: Strategy パターン

- **使用箇所**: URLの正規化、リンク抽出ロジック
- **理由**: 
  - 異なるURL形式やHTML形式に対応可能
  - 将来の拡張性を確保
- **実装方法**: 
  - URL正規化戦略、リンク抽出戦略をインターフェースで定義
  - 必要に応じて実装を切り替え可能

### パターン名: Observer パターン

- **使用箇所**: 調査進捗の通知（IPC経由）
- **理由**: 
  - 調査の進捗をリアルタイムでUIに通知
  - 疎結合な設計を実現
- **実装方法**: 
  - ElectronのIPCイベントを使用
  - またはJotaiのAtomを使用して状態を共有

### パターン名: Queue パターン

- **使用箇所**: 調査待ちURLの管理（ScanController）
- **理由**: 
  - 並列処理の制御
  - 調査順序の管理
- **実装方法**: 
  - URLキューを実装
  - 並列リクエスト数の制限を実装

## 1.5 型定義（TypeScript）

### ScanState

```typescript
interface ScanState {
  scanHistoryId: number;
  status: 'running' | 'paused' | 'stopped' | 'completed' | 'error';
  currentUrl: string | null;
  totalPages: number;
  checkedPages: number;
  brokenLinksCount: number;
}
```

### ScanSettings

```typescript
interface ScanSettings {
  maxPages: number;
  maxDepth: number;
  timeout: number;
  parallelRequests: number;
  retryCount: number;
  domainRestriction: 'same' | 'all';
  fileTypes: string[];
}
```

### LinkCheckResult

```typescript
interface LinkCheckResult {
  id?: number;
  scanHistoryId: number;
  url: string;
  sourceUrl: string;
  anchorText?: string;
  httpStatusCode?: number;
  errorMessage?: string;
  checkedAt: Date;
}
```

### Progress

```typescript
interface Progress {
  totalPages: number;
  checkedPages: number;
  brokenLinksCount: number;
  percentage: number;
  currentUrl?: string;
}
```

