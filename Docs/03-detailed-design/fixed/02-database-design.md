# データベース詳細設計

## 2.1 ER図

```
┌─────────────────────┐
│   scan_history      │
│─────────────────────│
│ id (PK)             │
│ start_url           │
│ started_at          │
│ completed_at        │
│ status              │
│ total_pages         │
│ broken_links_count  │
│ settings_json       │
└──────────┬──────────┘
           │
           │ 1:N
           │
┌──────────▼──────────┐
│ link_check_results  │
│─────────────────────│
│ id (PK)             │
│ scan_history_id(FK) │
│ url                 │
│ source_url          │
│ anchor_text         │
│ http_status_code    │
│ error_message       │
│ checked_at          │
└─────────────────────┘

┌─────────────────────┐
│   settings          │
│─────────────────────│
│ id (PK)             │
│ key                 │
│ value               │
│ updated_at          │
└─────────────────────┘
```

## 2.2 テーブル定義

### テーブル名: scan_history（調査履歴テーブル）

- **テーブル名**: `scan_history`
- **説明**: リンク切れ調査の実行履歴を保存するテーブル
- **カラム**:
  - `id`: INTEGER PRIMARY KEY AUTOINCREMENT - 調査履歴ID（主キー）
  - `start_url`: TEXT NOT NULL - 調査開始URL
  - `started_at`: DATETIME NOT NULL - 調査開始日時
  - `completed_at`: DATETIME - 調査完了日時（NULL可、調査中はNULL）
  - `status`: TEXT NOT NULL - 調査ステータス（'running', 'completed', 'paused', 'stopped', 'error'）
  - `total_pages`: INTEGER DEFAULT 0 - 調査した総ページ数
  - `broken_links_count`: INTEGER DEFAULT 0 - リンク切れの総数
  - `settings_json`: TEXT - 調査時の設定（JSON形式）
  - `created_at`: DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP - レコード作成日時
  - `updated_at`: DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP - レコード更新日時
- **主キー**: `id`
- **外部キー**: なし
- **インデックス**: 
  - `idx_scan_history_started_at`: `started_at`（降順、履歴一覧の並び替え用）
  - `idx_scan_history_status`: `status`（ステータス検索用）

### テーブル名: link_check_results（調査結果テーブル）

- **テーブル名**: `link_check_results`
- **説明**: リンク切れ調査の結果を保存するテーブル
- **カラム**:
  - `id`: INTEGER PRIMARY KEY AUTOINCREMENT - 調査結果ID（主キー）
  - `scan_history_id`: INTEGER NOT NULL - 調査履歴ID（外部キー）
  - `url`: TEXT NOT NULL - 調査対象URL（リンク切れのURL）
  - `source_url`: TEXT NOT NULL - リンク元URL（リンク切れが見つかったページのURL）
  - `anchor_text`: TEXT - アンカーテキスト（リンクの表示テキスト、NULL可）
  - `http_status_code`: INTEGER - HTTPステータスコード（NULL可、ネットワークエラー時など）
  - `error_message`: TEXT - エラーメッセージ（NULL可）
  - `checked_at`: DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP - 調査実行日時
  - `created_at`: DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP - レコード作成日時
- **主キー**: `id`
- **外部キー**: 
  - `fk_link_check_results_scan_history_id`: `scan_history_id` → `scan_history.id` ON DELETE CASCADE
- **インデックス**: 
  - `idx_link_check_results_scan_history_id`: `scan_history_id`（調査履歴での検索用）
  - `idx_link_check_results_url`: `url`（URL検索用）
  - `idx_link_check_results_http_status_code`: `http_status_code`（ステータスコードでのフィルタリング用）
  - `idx_link_check_results_checked_at`: `checked_at`（日時での並び替え用）

### テーブル名: settings（設定テーブル）

- **テーブル名**: `settings`
- **説明**: アプリケーションの設定を保存するテーブル
- **カラム**:
  - `id`: INTEGER PRIMARY KEY AUTOINCREMENT - 設定ID（主キー）
  - `key`: TEXT NOT NULL UNIQUE - 設定キー
  - `value`: TEXT - 設定値（JSON形式で複雑な設定を保存可能、NULL可）
  - `updated_at`: DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP - 設定更新日時
- **主キー**: `id`
- **外部キー**: なし
- **インデックス**: 
  - `idx_settings_key`: `key`（UNIQUE制約により自動的にインデックスが作成される）

## 2.3 インデックス設計

### インデックス: idx_scan_history_started_at

- **テーブル**: `scan_history`
- **カラム**: `started_at`
- **種類**: 通常インデックス（降順）
- **理由**: 調査履歴一覧を日時順（新しい順）で表示するため

### インデックス: idx_scan_history_status

- **テーブル**: `scan_history`
- **カラム**: `status`
- **種類**: 通常インデックス
- **理由**: 特定のステータス（実行中など）の調査を検索するため

### インデックス: idx_link_check_results_scan_history_id

- **テーブル**: `link_check_results`
- **カラム**: `scan_history_id`
- **種類**: 通常インデックス
- **理由**: 特定の調査履歴に紐づく結果を取得する際の検索速度向上

### インデックス: idx_link_check_results_url

- **テーブル**: `link_check_results`
- **カラム**: `url`
- **種類**: 通常インデックス
- **理由**: URLでの検索・フィルタリングの速度向上

### インデックス: idx_link_check_results_http_status_code

- **テーブル**: `link_check_results`
- **カラム**: `http_status_code`
- **種類**: 通常インデックス
- **理由**: HTTPステータスコードでのフィルタリング（404のみ表示など）の速度向上

### インデックス: idx_link_check_results_checked_at

- **テーブル**: `link_check_results`
- **カラム**: `checked_at`
- **種類**: 通常インデックス
- **理由**: 日時順での並び替えの速度向上

### インデックス: idx_settings_key

- **テーブル**: `settings`
- **カラム**: `key`
- **種類**: UNIQUEインデックス
- **理由**: 設定キーの一意性を保証し、設定取得時の検索速度向上

## 2.4 制約・トリガー

### 制約: fk_link_check_results_scan_history_id

- **テーブル**: `link_check_results`
- **制約内容**: 外部キー制約（`scan_history_id` → `scan_history.id` ON DELETE CASCADE）
- **理由**: 調査履歴を削除した際に、関連する調査結果も自動的に削除されるようにする

### 制約: settings.key UNIQUE

- **テーブル**: `settings`
- **制約内容**: UNIQUE制約（`key`カラム）
- **理由**: 同じ設定キーが重複しないようにする

### トリガー: update_scan_history_timestamp

- **テーブル**: `scan_history`
- **トリガー内容**: `updated_at`カラムを自動更新
- **実行タイミング**: UPDATE時
- **実装**: SQLiteのトリガー機能を使用（またはアプリケーション側で実装）

### トリガー: update_settings_timestamp

- **テーブル**: `settings`
- **トリガー内容**: `updated_at`カラムを自動更新
- **実行タイミング**: UPDATE時
- **実装**: SQLiteのトリガー機能を使用（またはアプリケーション側で実装）

## 2.5 データマイグレーション

### マイグレーション方針

- ORM（Sequelize/TypeORM）のマイグレーション機能を使用
- バージョン管理されたマイグレーションファイルを作成
- 開発環境と本番環境で同じマイグレーションを実行

### バージョン管理

- マイグレーションファイルは時系列で管理（例: `001_create_tables.sql`, `002_add_indexes.sql`）
- マイグレーション履歴テーブルで実行済みマイグレーションを管理
- ロールバック機能を実装（可能な範囲で）

### 初期マイグレーション

1. `scan_history`テーブルの作成
2. `link_check_results`テーブルの作成
3. `settings`テーブルの作成
4. インデックスの作成
5. 外部キー制約の設定
6. トリガーの作成（必要に応じて）

### データ型の詳細

#### SQLiteのデータ型マッピング

- **INTEGER**: 整数型（主キー、外部キー、数値カラム）
- **TEXT**: 文字列型（URL、メッセージ、JSONなど）
- **DATETIME**: 日時型（TEXTとして保存し、ISO8601形式で管理）

#### 設定値のJSON形式

`settings`テーブルの`value`カラムは、複雑な設定を保存するためにJSON形式を使用：

```json
{
  "maxPages": 1000,
  "maxDepth": 5,
  "timeout": 5000,
  "parallelRequests": 5,
  "retryCount": 3,
  "domainRestriction": "same",
  "fileTypes": ["html"]
}
```

## 2.6 データ整合性

### 参照整合性

- `link_check_results.scan_history_id`は必ず`scan_history.id`に存在する値であること
- 調査履歴を削除すると、関連する調査結果も自動的に削除される（CASCADE）

### データ整合性チェック

- `scan_history.status`は定義された値のみ許可（'running', 'completed', 'paused', 'stopped', 'error'）
- `link_check_results.http_status_code`は有効なHTTPステータスコードであること（100-599の範囲）
- URLは有効なURL形式であること（アプリケーション側でバリデーション）

## 2.7 パフォーマンス考慮事項

### 大量データへの対応

- 調査結果が数万件になる可能性があるため、インデックスを適切に設定
- ページネーション機能を実装して、一度に取得するデータ量を制限
- 古い調査結果のアーカイブ機能を検討（将来実装）

### クエリ最適化

- JOINクエリを使用する際は、インデックスを活用
- 集計クエリ（COUNT、SUMなど）は適切にインデックスを使用
- 不要なデータの取得を避ける（SELECT * の使用を避ける）

