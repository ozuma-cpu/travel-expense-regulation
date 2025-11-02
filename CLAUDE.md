# プロジェクト設定

## 基本設定
```yaml
プロジェクト名: 出張報告書自動生成ツール
開始日: 2025-11-02
技術スタック:
  frontend:
    - React 18 + TypeScript 5
    - MUI v6
    - Zustand (状態管理)
    - React Router v6
    - React Query
    - Vite 5
    - react-papaparse (CSV処理)
    - jsPDF + html2canvas (PDF生成)
    - @react-oauth/google (Google認証)
  backend:
    - Python 3.11+ + FastAPI
    - SQLAlchemy (ORM)
    - Pydantic (バリデーション)
    - google-auth-oauthlib (Google OAuth認証)
    - google-api-python-client (Google Sheets API)
  database:
    - PostgreSQL (Neon)
```

## 開発環境
```yaml
ポート設定:
  # 複数プロジェクト並行開発のため、一般的でないポートを使用
  frontend: 3247
  backend: 8432
  database: 5433

環境変数:
  設定ファイル: .env.local（ルートディレクトリ）
  必須項目:
    - DATABASE_URL
    - GOOGLE_CLIENT_ID
    - GOOGLE_CLIENT_SECRET
    - GOOGLE_REDIRECT_URI
    - FRONTEND_URL
    - BACKEND_URL
```

## テスト認証情報
```yaml
開発用アカウント:
  email: test@business-trip-tool.local
  password: DevTest2025!Trip

外部サービス:
  Google Cloud Platform: 各自のGoogleアカウントで設定
  Neon Database: 各自のアカウントで設定
```

## コーディング規約

### 命名規則
```yaml
ファイル名:
  - コンポーネント: PascalCase.tsx (例: TripList.tsx, TripForm.tsx)
  - ユーティリティ: camelCase.ts (例: calculateAllowance.ts)
  - 定数: UPPER_SNAKE_CASE.ts (例: API_ENDPOINTS.ts)

変数・関数:
  - 変数: camelCase
  - 関数: camelCase
  - 定数: UPPER_SNAKE_CASE
  - 型/インターフェース: PascalCase
```

### コード品質
```yaml
必須ルール:
  - TypeScript: strictモード有効
  - 未使用の変数/import禁止
  - console.log本番環境禁止
  - エラーハンドリング必須
  - 旅費計算ロジックは必ずテスト実装

フォーマット:
  - インデント: スペース2つ
  - セミコロン: あり
  - クォート: シングル
```

### コミットメッセージ
```yaml
形式: [type]: [description]

type:
  - feat: 新機能
  - fix: バグ修正
  - docs: ドキュメント
  - style: フォーマット
  - refactor: リファクタリング
  - test: テスト
  - chore: その他

例:
  - "feat: 出張登録機能を追加"
  - "feat: 旅費自動計算ロジックを実装"
  - "fix: PDF出力時の日本語フォント問題を修正"
```

## プロジェクト固有ルール

### APIエンドポイント
```yaml
命名規則:
  - RESTful形式を厳守
  - 複数形を使用 (/business-trips)
  - ケバブケース使用 (/travel-expense-policy)

主要エンドポイント:
  - POST /api/auth/google: Google OAuth認証
  - GET /api/business-trips: 出張一覧取得
  - POST /api/business-trips: 出張登録
  - PUT /api/business-trips/:id: 出張更新
  - DELETE /api/business-trips/:id: 出張削除
  - POST /api/business-trips/bulk-import: CSV一括インポート
  - GET /api/travel-expense-policy: 旅費規定取得
  - PUT /api/travel-expense-policy: 旅費規定更新
  - POST /api/settings/connect-sheets: Google Sheets連携
  - GET /api/settings: システム設定取得
  - PUT /api/settings: システム設定更新
```

### 型定義
```yaml
配置:
  frontend: src/types/index.ts
  backend: src/types/index.ts

同期ルール:
  - 両ファイルは常に同一内容を保つ
  - 片方を更新したら即座にもう片方も更新
  - 特に BusinessTrip、TravelExpensePolicy の型定義は厳密に同期

主要型:
  - User: ユーザー情報
  - BusinessTrip: 出張データ
  - TravelExpensePolicy: 旅費規定
  - SystemSettings: システム設定
  - CalculatedAllowance: 計算結果
```

### 旅費計算ロジック
```yaml
実装場所:
  - backend/src/services/allowance_calculator.py

重要ルール:
  - 金額は常に整数で扱う（小数点以下切り捨て）
  - 旅費規定のJSONスキーマをバリデーション
  - 計算ロジックは必ずユニットテストを実装
  - 計算結果はログに記録（デバッグ用）

計算フロー:
  1. 旅費規定を取得
  2. 出張期間（日数）を計算
  3. 日帰り/宿泊を判定
     - 開始日 = 終了日 → 日帰り出張（日帰り日当を適用）
     - 開始日 < 終了日 → 宿泊出張（宿泊日当×日数を適用）
  4. 宿泊費を計算（定額 or 実費上限）
  5. 交通費を計算（定額 or 実費）
  6. 合計手当額を返す
```

### Google Sheets連携
```yaml
実装場所:
  - backend/src/services/sheets_service.py

重要ルール:
  - アクセストークンの有効期限チェック（1時間）
  - リフレッシュトークンで自動更新
  - API制限（1日500リクエスト）を考慮
  - エラー時は詳細なログを記録

連携フロー:
  1. OAuth認証でトークン取得
  2. 新規スプレッドシート作成
  3. ヘッダー行を挿入
  4. 出張データを追記
  5. スプレッドシートURLを保存
```

### PDF生成
```yaml
実装場所:
  - frontend/src/utils/pdfGenerator.ts

重要ルール:
  - jsPDF + html2canvasを使用
  - IPAexフォントを埋め込み（日本語対応）
  - PDF生成は非同期処理
  - ローディング状態を表示

フォーマット:
  - A4サイズ
  - マージン: 上下左右20mm
  - フォント: IPAexゴシック
  - 出力項目: 会社名、出張先、期間、目的、手当内訳、合計額
```

## 🆕 最新技術情報（知識カットオフ対応）

### React v18対応
```yaml
注意点:
  - react-pdfはReact v18で動作しないため使用禁止
  - 代わりにjsPDF + html2canvasを使用
  - StrictModeで2回レンダリングされる挙動に注意
```

### Google Sheets API v4
```yaml
注意点:
  - v3は2021年6月に終了済み
  - v4の公式ドキュメントを参照
  - google-api-python-clientの最新版を使用
```

### FastAPI
```yaml
注意点:
  - 非同期処理（async/await）を活用
  - Pydanticモデルで厳密なバリデーション
  - CORS設定を適切に行う
```

## ⚠️ プロジェクト固有の注意事項

### Google OAuth認証
```yaml
重要:
  - クライアントIDとシークレットは環境変数で管理
  - リダイレクトURIをGCP Consoleに正しく登録
  - スコープ: openid, email, profile, https://www.googleapis.com/auth/spreadsheets
  - アクセストークンは1時間で失効するため、リフレッシュトークンで更新
```

### CSV一括インポート
```yaml
制約事項:
  - ブラウザのメモリ制限により、1回あたり500件以内を推奨
  - CSVファイルサイズは10MB以内を推奨
  - エンコーディング: UTF-8またはShift-JIS対応
  - 必須カラム: 出張先、開始日、終了日、目的
```

### 旅費規定のJSON形式
```yaml
柔軟性:
  - JSON形式で柔軟な条件設定が可能
  - 地域別、役職別など、カスタマイズ可能な設計
  - バリデーションスキーマを定義し、不正なデータを防ぐ

例:
  daily_allowance_rules:
    - condition: "destination == '海外'"
      amount: 5000
    - condition: "destination == '国内'"
      amount: 2000
```

### データベース設計
```yaml
重要:
  - PostgreSQLのJSON型を活用（旅費規定、PDF設定）
  - インデックス: user_id, start_date, end_date
  - トランザクション処理を適切に使用
  - マイグレーション管理（Alembic）
```

## 📝 作業ログ（最新5件）
```yaml
- 2025-11-02: 要件定義書作成完了
- 2025-11-02: 技術スタック決定（React 18 + FastAPI + PostgreSQL）
- 2025-11-02: 外部API確定（Google Sheets API, Google OAuth 2.0）
- 2025-11-02: ページ構成確定（4ページ）
- 2025-11-02: CLAUDE.md生成完了
```

---

**プロジェクト目標**: 出張報告書作成を30分から3分に短縮し、計算ミスをゼロにする。
