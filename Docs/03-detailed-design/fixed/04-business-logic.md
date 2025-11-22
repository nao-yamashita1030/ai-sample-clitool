# ビジネスロジック設計

## 6.1 ビジネスルール

### ルール1: リンク切れの判定ルール

- **ルール名**: リンク切れ判定ルール
- **内容**: 
  - HTTPステータスコードが400-599の範囲の場合、リンク切れと判定
  - 特に404（Not Found）は明確なリンク切れ
  - 500（Internal Server Error）もリンク切れとして扱う
  - タイムアウトやネットワークエラーもリンク切れとして扱う
  - HTTPステータスコードが200-299の場合は正常
  - HTTPステータスコードが300-399の場合はリダイレクト（正常として扱い、リダイレクト先も調査対象）
- **適用範囲**: すべてのリンクチェック処理

### ルール2: 調査対象URLの判定ルール

- **ルール名**: URL調査対象判定ルール
- **内容**: 
  - 設定で「同一ドメインのみ」が選択されている場合、開始URLと同じドメインのURLのみを調査対象とする
  - 設定で「すべて」が選択されている場合、外部ドメインのURLも調査対象とする（ただし、外部ドメイン内のリンクは再帰的に調査しない）
  - ファイルタイプが設定で指定されたもののみを調査対象とする（デフォルト: HTMLのみ）
  - 階層レベルが設定した最大階層レベルを超えるURLは調査対象外
  - 既に調査済みのURLは調査対象外（visitedUrlsで管理）
- **適用範囲**: リンク抽出後のフィルタリング処理

### ルール3: 調査終了条件ルール

- **ルール名**: 調査終了条件ルール
- **内容**: 
  - キューが空になった場合、調査を終了
  - 調査したページ数が設定した最大ページ数に到達した場合、調査を終了
  - 調査した階層レベルが設定した最大階層レベルに到達した場合、調査を終了
  - ユーザーが停止ボタンをクリックした場合、調査を終了
  - 致命的なエラーが発生した場合、調査を終了
- **適用範囲**: 調査制御処理

### ルール4: 並列処理制御ルール

- **ルール名**: 並列処理制御ルール
- **内容**: 
  - 同時に実行するHTTPリクエスト数は設定で指定された並列リクエスト数まで
  - 並列リクエスト数に達した場合、新しいリクエストは待機キューに追加
  - リクエストが完了したら、待機キューから次のリクエストを開始
  - 並列リクエスト数は1以上20以下
- **適用範囲**: 並列処理管理

### ルール5: リトライルール

- **ルール名**: リトライルール
- **内容**: 
  - ネットワークエラーやタイムアウトが発生した場合、設定で指定されたリトライ回数まで再試行
  - リトライ間隔は指数バックオフ（1秒、2秒、4秒...）
  - HTTPエラー（404、500など）はリトライしない
  - リトライ回数は0以上10以下
- **適用範囲**: HTTPリクエスト処理

## 6.2 ビジネスロジック詳細

### ロジック1: リンク切れ判定ロジック

- **目的**: HTTPレスポンスからリンク切れかどうかを判定する
- **入力**: 
  - `httpStatusCode: number | null` - HTTPステータスコード
  - `errorMessage: string | null` - エラーメッセージ
- **処理**: 
  1. `httpStatusCode`がnullの場合（ネットワークエラーなど）:
     - `errorMessage`が存在する場合、リンク切れと判定
     - それ以外は正常と判定（ただし、エラー情報は記録）
  2. `httpStatusCode`が200-299の場合:
     - 正常と判定
  3. `httpStatusCode`が300-399の場合:
     - リダイレクトと判定（正常として扱う）
     - リダイレクト先のURLをキューに追加
  4. `httpStatusCode`が400-599の場合:
     - リンク切れと判定
     - 特に404は「ページが見つかりません」、500は「サーバーエラー」として記録
- **出力**: 
  - `isBroken: boolean` - リンク切れかどうか
  - `reason: string` - リンク切れの理由（'not_found', 'server_error', 'network_error', 'timeout'など）
- **例外処理**: 
  - 予期しないエラーが発生した場合、エラーログを記録し、リンク切れとして扱う

### ロジック2: URL正規化ロジック

- **目的**: 相対URLを絶対URLに変換し、URLを正規化する
- **入力**: 
  - `url: string` - 正規化するURL（相対URLまたは絶対URL）
  - `baseUrl: string` - ベースURL（相対URLの場合の基準）
- **処理**: 
  1. URLが絶対URL（http://またはhttps://で始まる）の場合:
     - そのまま使用
  2. URLが相対URLの場合:
     - baseUrlを基準に絶対URLに変換
     - 例: baseUrlが`https://example.com/page1/`、urlが`../page2.html`の場合、`https://example.com/page2.html`に変換
  3. URLの正規化:
     - フラグメント（#以降）を削除
     - クエリパラメータは保持
     - 末尾のスラッシュを統一（設定に基づく）
     - 大文字小文字を統一（ドメイン部分は小文字に）
- **出力**: 
  - `normalizedUrl: string` - 正規化された絶対URL
- **例外処理**: 
  - URL形式が不正な場合、エラーをスロー

### ロジック3: リンク抽出ロジック

- **目的**: HTMLからリンク（`<a>`タグ）を抽出する
- **入力**: 
  - `html: string` - HTML文字列
  - `baseUrl: string` - ベースURL（相対URLを絶対URLに変換するため）
- **処理**: 
  1. HTMLパーサー（cheerioなど）を使用してHTMLを解析
  2. すべての`<a>`タグを検索
  3. 各`<a>`タグから以下を抽出:
     - `href`属性の値（URL）
     - アンカーテキスト（`<a>`タグ内のテキスト）
  4. 抽出したURLを正規化（URL正規化ロジックを使用）
  5. 重複URLを除去
  6. 無効なURLを除外
- **出力**: 
  - `links: LinkInfo[]` - 抽出したリンク情報の配列
    - `url: string` - リンクURL
    - `anchorText: string | null` - アンカーテキスト
    - `sourceUrl: string` - リンク元URL（baseUrl）
- **例外処理**: 
  - HTMLの解析に失敗した場合、空の配列を返す
  - エラーログを記録

### ロジック4: URLフィルタリングロジック

- **目的**: 抽出したリンクから、調査対象となるURLをフィルタリングする
- **入力**: 
  - `links: LinkInfo[]` - 抽出したリンク情報の配列
  - `startUrl: string` - 開始URL
  - `settings: ScanSettings` - 調査設定
  - `currentDepth: number` - 現在の階層レベル
  - `visitedUrls: Set<string>` - 調査済みURLセット
- **処理**: 
  1. 各リンクについて以下をチェック:
     - **ドメイン制限チェック**: 
       - `settings.domainRestriction === 'same'`の場合、リンクURLのドメインが開始URLのドメインと一致するかチェック
       - 一致しない場合はスキップ
     - **ファイルタイプチェック**: 
       - リンクURLの拡張子を取得
       - `settings.fileTypes`に含まれているかチェック
       - 含まれていない場合はスキップ（拡張子がない場合はHTMLとして扱う）
     - **階層レベルチェック**: 
       - `currentDepth + 1`が`settings.maxDepth`を超えていないかチェック
       - 超えている場合はスキップ
     - **調査済みチェック**: 
       - `visitedUrls`に含まれているかチェック
       - 含まれている場合はスキップ
  2. すべてのチェックを通過したリンクのみを返す
- **出力**: 
  - `filteredLinks: LinkInfo[]` - フィルタリング後のリンク情報の配列
- **例外処理**: 
  - URLの解析に失敗した場合、そのURLをスキップ
  - エラーログを記録

### ロジック5: 進捗計算ロジック

- **目的**: 調査の進捗状況を計算する
- **入力**: 
  - `totalPages: number` - 調査対象の総ページ数（キューに追加されたURL数）
  - `checkedPages: number` - 調査済みページ数
  - `brokenLinksCount: number` - リンク切れの数
- **処理**: 
  1. 進捗率を計算: `percentage = (checkedPages / totalPages) * 100`
  2. 総ページ数が0の場合は進捗率を0%とする
  3. 進捗率が100%を超える場合は100%に制限
- **出力**: 
  - `progress: Progress` - 進捗情報
    - `totalPages: number` - 総ページ数
    - `checkedPages: number` - 調査済みページ数
    - `brokenLinksCount: number` - リンク切れの数
    - `percentage: number` - 進捗率（0-100）
    - `currentUrl: string | null` - 現在調査中のURL
- **例外処理**: 
  - 計算エラーが発生した場合、デフォルト値を返す

### ロジック6: 統計情報計算ロジック

- **目的**: 調査結果の統計情報を計算する
- **入力**: 
  - `results: LinkCheckResult[]` - 調査結果の配列
- **処理**: 
  1. 総リンク数を計算: `totalLinks = results.length`
  2. リンク切れの数を計算: `brokenLinks = results.filter(r => r.httpStatusCode >= 400 || r.errorMessage).length`
  3. 正常なリンクの数を計算: `validLinks = totalLinks - brokenLinks`
  4. リンク切れ率を計算: `brokenRate = (brokenLinks / totalLinks) * 100`
  5. HTTPステータスコード別の集計:
     - 200-299: 正常
     - 300-399: リダイレクト
     - 400-499: クライアントエラー（特に404）
     - 500-599: サーバーエラー
     - null: ネットワークエラーなど
- **出力**: 
  - `statistics: Statistics` - 統計情報
    - `totalLinks: number` - 総リンク数
    - `brokenLinks: number` - リンク切れの数
    - `validLinks: number` - 正常なリンクの数
    - `brokenRate: number` - リンク切れ率（%）
    - `statusCodeDistribution: Record<number, number>` - ステータスコード別の分布
- **例外処理**: 
  - 計算エラーが発生した場合、デフォルト値を返す

## 6.3 計算ロジック

### 計算1: 進捗率の計算

- **計算式**: `percentage = (checkedPages / totalPages) * 100`
- **入力**: 
  - `checkedPages: number` - 調査済みページ数
  - `totalPages: number` - 総ページ数
- **出力**: 
  - `percentage: number` - 進捗率（0-100の範囲）
- **精度**: 
  - 小数点以下1桁まで表示
  - 0%未満は0%、100%超過は100%に制限

### 計算2: リンク切れ率の計算

- **計算式**: `brokenRate = (brokenLinks / totalLinks) * 100`
- **入力**: 
  - `brokenLinks: number` - リンク切れの数
  - `totalLinks: number` - 総リンク数
- **出力**: 
  - `brokenRate: number` - リンク切れ率（0-100の範囲）
- **精度**: 
  - 小数点以下2桁まで表示
  - 0%未満は0%、100%超過は100%に制限
  - 総リンク数が0の場合は0%を返す

### 計算3: 階層レベルの計算

- **計算式**: `depth = countPathSegments(startUrl, currentUrl)`
- **入力**: 
  - `startUrl: string` - 開始URL
  - `currentUrl: string` - 現在のURL
- **処理**: 
  1. 開始URLのパス部分を取得（例: `/page1/page2/`）
  2. 現在URLのパス部分を取得（例: `/page1/page2/page3/`）
  3. 開始URLのパスを基準に、現在URLのパスが何階層深いか計算
  4. パスセグメントの差分を計算
- **出力**: 
  - `depth: number` - 階層レベル（0から開始）
- **精度**: 
  - 整数値
  - 0以上

### 計算4: リトライ間隔の計算（指数バックオフ）

- **計算式**: `interval = baseInterval * Math.pow(2, retryCount)`
- **入力**: 
  - `retryCount: number` - リトライ回数（0から開始）
  - `baseInterval: number` - ベース間隔（ミリ秒、デフォルト: 1000）
- **出力**: 
  - `interval: number` - リトライ間隔（ミリ秒）
- **精度**: 
  - 整数値（ミリ秒）
  - 例: 1回目: 1000ms, 2回目: 2000ms, 3回目: 4000ms, 4回目: 8000ms
  - 最大間隔は60000ms（60秒）に制限

## 6.4 ビジネスルールの実装詳細

### ルール実装: ドメイン抽出

```typescript
function extractDomain(url: string): string {
  try {
    const urlObj = new URL(url);
    return urlObj.hostname;
  } catch (error) {
    throw new Error(`Invalid URL: ${url}`);
  }
}
```

### ルール実装: 階層レベル計算

```typescript
function calculateDepth(startUrl: string, currentUrl: string): number {
  try {
    const startUrlObj = new URL(startUrl);
    const currentUrlObj = new URL(currentUrl);
    
    // ドメインが異なる場合は0を返す（外部リンク）
    if (startUrlObj.hostname !== currentUrlObj.hostname) {
      return 0;
    }
    
    const startPath = startUrlObj.pathname.split('/').filter(p => p);
    const currentPath = currentUrlObj.pathname.split('/').filter(p => p);
    
    // 共通部分を除いた差分の階層数を計算
    let commonLength = 0;
    for (let i = 0; i < Math.min(startPath.length, currentPath.length); i++) {
      if (startPath[i] === currentPath[i]) {
        commonLength++;
      } else {
        break;
      }
    }
    
    return currentPath.length - commonLength;
  } catch (error) {
    return 0; // エラー時は0を返す
  }
}
```

