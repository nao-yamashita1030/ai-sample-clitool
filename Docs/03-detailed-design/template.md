# 詳細設計書

このディレクトリには、詳細設計の各項目を分割して管理します。

## ディレクトリ構造

```
03-detailed-design/
├── template.md              # このファイル（目次・概要）
├── 01-class-design.md      # クラス設計
├── 02-database-design.md   # データベース詳細設計
├── 03-process-flow.md      # 処理フロー
├── 04-business-logic.md    # ビジネスロジック設計
├── 05-security.md          # セキュリティ詳細設計
├── 06-error-handling.md    # エラーハンドリング詳細
├── 07-performance.md      # パフォーマンス詳細設計
├── 08-test-design.md       # テスト設計
├── draft/                   # 検討中の内容
│   ├── api/               # API詳細仕様（検討中）
│   │   ├── README.md      # API一覧・目次
│   │   ├── template.md    # APIテンプレート
│   │   └── [api-name].md  # 各APIの詳細仕様
│   └── screens/           # 画面詳細設計（検討中）
│       ├── README.md      # 画面一覧・目次
│       ├── template.md    # 画面テンプレート
│       └── [screen-name].md # 各画面の詳細設計
└── fixed/                 # 確定した内容
    ├── api/               # API詳細仕様（確定）
    │   ├── README.md      # API一覧・目次
    │   └── [api-name].md  # 各APIの詳細仕様
    └── screens/           # 画面詳細設計（確定）
        ├── README.md      # 画面一覧・目次
        └── [screen-name].md # 各画面の詳細設計
```

## 各ファイルの説明

### メインファイル

- **template.md** (このファイル): 詳細設計の目次と概要
- **01-class-design.md**: クラス設計
- **02-database-design.md**: データベース詳細設計
- **03-process-flow.md**: 処理フロー
- **04-business-logic.md**: ビジネスロジック設計
- **05-security.md**: セキュリティ詳細設計
- **06-error-handling.md**: エラーハンドリング詳細
- **07-performance.md**: パフォーマンス詳細設計
- **08-test-design.md**: テスト設計

### API詳細仕様（draft/api/ と fixed/api/ ディレクトリ）

- **draft/api/README.md**: API一覧と目次（検討中）
- **draft/api/template.md**: APIテンプレート
- **draft/api/[api-name].md**: 各APIの詳細仕様（検討中）
- **fixed/api/README.md**: API一覧と目次（確定）
- **fixed/api/[api-name].md**: 各APIの詳細仕様（確定）

### 画面詳細設計（draft/screens/ と fixed/screens/ ディレクトリ）

- **draft/screens/README.md**: 画面一覧と目次（検討中）
- **draft/screens/template.md**: 画面テンプレート
- **draft/screens/[screen-name].md**: 各画面の詳細設計（検討中）
- **fixed/screens/README.md**: 画面一覧と目次（確定）
- **fixed/screens/[screen-name].md**: 各画面の詳細設計（確定）

## 作成手順

1. 各テンプレートファイルを `draft/` ディレクトリにコピー
2. プロジェクトに合わせて内容を記入
3. APIや画面は `draft/api/` や `draft/screens/` 内に個別ファイルとして作成
4. 確定したら `fixed/api/` や `fixed/screens/` に移動またはコピー
5. 各APIや画面を個別にfixしたか確認し、確定したものは `fixed/` に移動
