# Environment Workflow 実践例とトラブルシューティング

## 実際のワークフロー例

### 例1: 新機能の実装

#### シナリオ
Webアプリケーションに新しいユーザー認証機能を追加する

#### 実際のコマンド実行例

```bash
# 1. 現在の環境状況を確認
$ container-use list
No active environments found.

# 2. エージェントに指示（Claude Codeでの例）
$ claude "ユーザー認証機能を実装してください。JWT トークンを使った認証システムを作成し、ログイン・ログアウト・ユーザー登録の機能を含めてください。"

# エージェントが環境 "auth-implementation-xyz" を作成

# 3. 作業進捗を確認
$ container-use list
Environment ID: auth-implementation-xyz
Title: ユーザー認証機能の実装
Status: Running
Branch: container-use/auth-implementation-xyz

# 4. 変更内容を確認
$ container-use diff auth-implementation-xyz
Added files:
+ src/auth/jwt.js
+ src/auth/middleware.js
+ src/routes/auth.js
+ tests/auth.test.js

Modified files:
~ src/app.js (added auth routes)
~ package.json (added dependencies)

# 5. 詳細なコミット履歴を確認
$ container-use log auth-implementation-xyz
commit abc123 - Add JWT authentication middleware
commit def456 - Implement user registration endpoint
commit ghi789 - Add login/logout functionality
commit jkl012 - Add comprehensive auth tests

# 6. テスト結果を確認
$ container-use terminal auth-implementation-xyz
> npm test
All tests passing ✅

# 7. 作業を承認してマージ
$ container-use merge auth-implementation-xyz
Successfully merged auth-implementation-xyz into main branch.
Environment auth-implementation-xyz deleted.
```

### 例2: バグ修正

#### シナリオ
本番環境でメモリリークが発生している問題を修正

```bash
# 1. 緊急修正用の環境を作成
$ claude "本番でメモリリークが発生しています。Node.jsアプリのメモリ使用量を調査し、リークの原因を特定して修正してください。"

# エージェントが環境 "memory-leak-fix-abc" を作成

# 2. 修正作業の監視
$ container-use log memory-leak-fix-abc --follow
commit 123abc - Add memory profiling tools
commit 456def - Identify memory leak in user session management
commit 789ghi - Fix memory leak by properly clearing timers

# 3. 修正内容の確認
$ container-use diff memory-leak-fix-abc
Modified files:
~ src/session/manager.js (fixed timer cleanup)
~ src/middleware/session.js (improved garbage collection)
+ tests/memory-leak.test.js (added memory leak tests)

# 4. メモリテストの実行
$ container-use terminal memory-leak-fix-abc
> npm run test:memory
Memory leak test: PASSED ✅
Memory usage stable after 1000 requests.

# 5. カスタムコミットメッセージで適用
$ container-use apply memory-leak-fix-abc
Enter commit message: Fix critical memory leak in session management

Deployed to main branch with commit: "Fix critical memory leak in session management"
```

### 例3: 複数アプローチの比較

#### シナリオ
パフォーマンス改善のために複数のアプローチを比較検討

```bash
# 1. 複数の最適化アプローチを並行実験
$ claude "データベースクエリのパフォーマンスを改善してください。インデックス最適化アプローチで実装してください。"
# 環境 "db-optimization-index" が作成される

$ claude "同じデータベースパフォーマンス問題を、クエリキャッシュアプローチで解決してください。"
# 環境 "db-optimization-cache" が作成される

$ claude "同じ問題を、データベース接続プール最適化で解決してください。"
# 環境 "db-optimization-pool" が作成される

# 2. 全環境の状況確認
$ container-use list
Environment ID: db-optimization-index
Title: データベースインデックス最適化
Status: Completed

Environment ID: db-optimization-cache
Title: クエリキャッシュ実装
Status: Completed

Environment ID: db-optimization-pool
Title: 接続プール最適化
Status: Completed

# 3. 各アプローチの性能比較
$ container-use terminal db-optimization-index
> npm run benchmark
Average query time: 45ms

$ container-use terminal db-optimization-cache
> npm run benchmark
Average query time: 12ms

$ container-use terminal db-optimization-pool
> npm run benchmark
Average query time: 38ms

# 4. 最適なアプローチ（キャッシュ）を選択
$ container-use merge db-optimization-cache

# 5. 他のアプローチを削除
$ container-use delete db-optimization-index
$ container-use delete db-optimization-pool
```

## よくある問題とトラブルシューティング

### 問題1: 環境が応答しなくなった

```bash
# 症状確認
$ container-use list
Environment ID: stuck-environment
Status: Running (no response)

# 解決方法1: 環境の再起動
$ container-use restart stuck-environment

# 解決方法2: 強制終了と再作成
$ container-use delete stuck-environment --force
$ claude "同じタスクをもう一度実行してください"
```

### 問題2: メモリ不足エラー

```bash
# 環境の詳細情報確認
$ container-use inspect memory-heavy-task
Container Memory: 8GB (98% used)
Container CPU: 4 cores (45% used)

# 解決方法: より多くのリソースで再作成
$ container-use delete memory-heavy-task
$ claude --memory 16GB "同じタスクを実行してください"
```

### 問題3: 期待した結果が得られない

```bash
# 詳細なログで問題を調査
$ container-use log problematic-environment --verbose

# インタラクティブセッションで直接調査
$ container-use terminal problematic-environment
> # 手動でデバッグ作業

# 問題を特定したら、エージェントに追加指示
$ claude "先ほどの実装で、Xの部分にYの問題があります。Zの方法で修正してください。"
```

### 問題4: 依存関係の競合

```bash
# 競合の詳細確認
$ container-use diff dependency-conflict
Modified files:
~ package.json (conflicting version requirements)

# クリーンな環境で再実行
$ container-use delete dependency-conflict
$ claude "依存関係を慎重に管理しながら、同じ機能を実装してください。競合するパッケージがあれば代替案を提案してください。"
```

## パフォーマンス最適化のコツ

### 1. 環境の並行利用

```bash
# 複数の独立したタスクを並行実行
$ claude "フロントエンドのUIコンポーネントを更新してください" &
$ claude "バックエンドAPIのレスポンス時間を改善してください" &
$ claude "データベーススキーマを最適化してください" &

# すべての作業を監視
$ watch container-use list
```

### 2. 段階的な統合

```bash
# 大きな変更を段階的に統合
$ container-use apply frontend-update
$ container-use apply backend-optimization  
$ container-use apply database-schema
```

### 3. 定期的なクリーンアップ

```bash
# 不要な環境を定期的に削除
$ container-use list --status completed | grep "7 days ago" | xargs container-use delete
```

## ベストプラクティスの実例

### 1. 実験的な機能開発

```bash
# 新しいアイデアの検証
$ claude "実験的に、機械学習を使った商品推薦機能を実装してみてください。まずは簡単なプロトタイプから始めて、効果を測定してください。"

# 結果を評価してから本格実装を決定
$ container-use diff ml-recommendation-prototype
$ container-use terminal ml-recommendation-prototype
> python evaluate_recommendations.py
Recommendation accuracy: 78%
Performance: 150ms average response time

# 良い結果なら本格実装、そうでなければ削除
$ container-use merge ml-recommendation-prototype  # または delete
```

### 2. セキュリティアップデート

```bash
# セキュリティ修正の慎重な適用
$ claude "CVE-2024-XXXX の脆弱性に対応してください。依存関係を更新し、影響範囲を最小限に抑えながら修正してください。"

# セキュリティテストの実行
$ container-use terminal security-update
> npm audit
> npm run test:security

# 問題がなければ適用
$ container-use apply security-update
```

### 3. レガシーコードのリファクタリング

```bash
# 段階的なリファクタリング
$ claude "legacy/auth.js を現代的なコードに段階的にリファクタリングしてください。機能を壊すことなく、テストカバレッジを維持しながら行ってください。"

# 各段階での確認
$ container-use log refactor-auth
commit 1: Extract utility functions
commit 2: Add comprehensive tests
commit 3: Modernize authentication logic
commit 4: Update error handling

# 段階的な統合
$ container-use merge refactor-auth
```

## 監視とレポート

### 環境使用状況の追跡

```bash
# 環境の利用統計
$ container-use stats --last-week
Total environments created: 45
Successful merges: 38
Deleted environments: 7
Average completion time: 2.3 hours

# 最も生産的な環境タイプ
$ container-use report --by-type
Feature development: 85% success rate
Bug fixes: 95% success rate
Refactoring: 78% success rate
```

このガイドを参考に、Environment Workflowを効果的に活用してください。