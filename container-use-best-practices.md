# Container-use ベストプラクティス

## 概要

このドキュメントでは、Container-useの高度な活用法とベストプラクティスをまとめています。基本的なワークフローについては [Environment Workflow ガイド](./environment-workflow-guide.md) を、実践例については [Environment Workflow 実践例](./environment-workflow-examples.md) を参照してください。

## 🚀 高度なワークフロー

### 直接リモートプッシュ手法

Container-useで作成した環境をローカルブランチにマージすることなく、任意のブランチ名で直接リモートリポジトリにプッシュする方法です。この手法は、CI/CDパイプラインとの統合やローカル環境を汚さない開発に特に有効です。

#### 背景と課題

従来のContainer-useワークフローでは：
- `container-use merge <env-id>` でローカルのメインブランチに統合
- `container-use apply <env-id>` でカスタムコミットメッセージで統合

しかし、以下のケースでは直接リモートプッシュが有効です：
- CI/CDパイプラインのトリガーとなるブランチを作成したい
- ローカルのワーキングブランチに影響を与えたくない
- 複数の実験的ブランチを並行してリモートに保存したい
- 外部レビューツールと連携したい

#### 方法1: checkout + ブランチリネーム（推奨）

```bash
# 1. 環境の作業内容を事前確認
container-use diff <environment-id>
container-use log <environment-id>

# 2. 環境をローカルワークスペースにチェックアウト
container-use checkout <environment-id>
# この時点で 'cu-<environment-id>' ブランチに切り替わります

# 3. 現在のブランチを任意の名前にリネーム（作業履歴を完全保持）
git branch -M docs/container-use-best-practices

# 4. リモートリポジトリに直接プッシュ
git push origin docs/container-use-best-practices

# 5. Pull Request の作成（GitHub CLI 使用例）
gh pr create --title "Add Container-use Best Practices Documentation" \
             --body "$(cat <<'EOF'
## Summary
Container-use 環境 <environment-id> での作業結果

## Changes Made
- Comprehensive best practices documentation
- Direct remote push techniques validation
- Practical workflow examples

## Test Plan
- [x] Created in Container-use environment
- [x] Validated direct remote push
- [x] Documented real execution examples
EOF
)"
```

**方法1の利点**:
- ✅ Container-useでの全ての作業履歴を保持
- ✅ コミットハッシュが変わらない
- ✅ 最もシンプルで効率的
- ✅ 実際に検証済み（本ドキュメント作成で使用）

#### 方法1-B: checkout + 新規ブランチ作成（代替手法）

```bash
# 既存のブランチから新しいブランチを作成する場合
container-use checkout <environment-id>
git checkout -b docs/alternative-branch-name
git push origin docs/alternative-branch-name
```

**方法1-Bの特徴**:
- 元のブランチ（cu-<environment-id>）を保持したい場合に有効
- 複数のブランチバリエーションを作成する場合に適している

#### 実践例: 本ドキュメントでの検証結果

```bash
# 実際に実行した手順（2024年12月14日検証）
$ container-use checkout informed-bedbug
Switched to branch 'cu-informed-bedbug'

$ git branch -M docs/container-use-best-practices
$ git push origin docs/container-use-best-practices
remote: Create a pull request for 'docs/container-use-best-practices' on GitHub by visiting:
remote:      https://github.com/70-10/container-use-practice/pull/new/docs/container-use-best-practices
To ssh://github.com/70-10/container-use-practice.git
 * [new branch]      docs/container-use-best-practices -> docs/container-use-best-practices
```

**検証結果**: ✅ 完全に成功。Container-use環境（informed-bedbug）で作成した700行超のドキュメントが、ローカルマージなしで直接リモートブランチとしてプッシュされました。

#### 使用ケース

##### CI/CDパイプラインとの統合
```bash
# 例: 自動テスト用ブランチの作成
container-use checkout test-implementation-env
git branch -M ci/automated-tests-$(date +%Y%m%d-%H%M%S)
git push origin ci/automated-tests-$(date +%Y%m%d-%H%M%S)

# GitHub Actions などでこのブランチをトリガーに自動テストを実行
```

##### 外部レビューツールとの連携
```bash
# 例: コードレビュー専用ブランチ
container-use checkout feature-development-env
git branch -M review/feature-xyz-$(date +%m%d)
git push origin review/feature-xyz-$(date +%m%d)

# 外部レビューツール（例: CodeGuru, SonarQube）で解析
```

##### 段階的デプロイメント
```bash
# ステージング環境用ブランチ
git branch -M staging/feature-release-v1.2
git push origin staging/feature-release-v1.2

# プロダクション環境用ブランチ（承認後）
git branch -M production/feature-release-v1.2
git push origin production/feature-release-v1.2
```

#### 注意事項とトラブルシューティング

##### 前提条件の確認
```bash
# Git認証の設定確認
ssh -T git@github.com  # SSH認証の場合
git config --global credential.helper  # HTTPS認証の場合

# ユーザー情報の確認
git config --global user.name
git config --global user.email

# リモートリポジトリの確認
git remote -v
```

##### よくある問題と解決方法

**問題1: 認証エラー**
```bash
# 症状: Permission denied (publickey) または 401 Unauthorized
# 解決方法1: SSH認証の設定確認
ls -la ~/.ssh/
cat ~/.ssh/id_rsa.pub
ssh-add ~/.ssh/id_rsa

# 解決方法2: Personal Access Token の設定
git config --global credential.helper store
# 初回プッシュ時にユーザー名とPATを入力
```

**問題2: ブランチが既に存在する**
```bash
# 症状: fatal: A branch named 'docs/xyz' already exists
# 解決方法: 一意なブランチ名の生成
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BRANCH_NAME="docs/best-practices-$TIMESTAMP"
git branch -M $BRANCH_NAME
git push origin $BRANCH_NAME
```

**問題3: リポジトリへの権限不足**
```bash
# 症状: remote: Permission to user/repo denied
# 解決方法: リポジトリの権限確認
gh repo view --json permissions
# または GitHub上でリポジトリの設定を確認
```

**問題4: リモート追跡ブランチの不整合**
```bash
# 症状: Your branch is up to date with 'container-use/old-env-name'
# 解決方法: ブランチリネーム後のリモート追跡設定
git branch --unset-upstream
git push -u origin docs/container-use-best-practices
```

### 複数環境の並行管理

Container-useでは複数の環境を並行して実行できるため、異なるアプローチを同時に試すことができます。

```bash
# 複数の実験的アプローチを並行実行
claude "アプローチA: Redis を使用してキャッシュ機能を実装"
claude "アプローチB: Memcached を使用してキャッシュ機能を実装"
claude "アプローチC: インメモリキャッシュでキャッシュ機能を実装"

# 各環境の状況確認
container-use list

# 各環境を異なるブランチにプッシュして比較
container-use checkout redis-cache-env
git branch -M experiment/redis-cache
git push origin experiment/redis-cache

container-use checkout memcached-cache-env
git branch -M experiment/memcached-cache
git push origin experiment/memcached-cache

container-use checkout inmemory-cache-env
git branch -M experiment/inmemory-cache
git push origin experiment/inmemory-cache

# 性能テスト結果に基づいて最適解を選択
# （例：Redisが最も高性能だった場合）
container-use merge redis-cache-env
container-use delete memcached-cache-env
container-use delete inmemory-cache-env
```

## 💡 実践的なテクニック

### ブランチ命名戦略

#### 標準的な命名規則
```bash
# 機能開発
feature/user-authentication
feature/payment-integration
feature/search-optimization

# バグ修正
fix/memory-leak-session
fix/validation-error-signup
fix/performance-bottleneck

# 実験的変更
experiment/ml-recommendations
experiment/new-ui-framework
experiment/database-migration

# ホットフィックス
hotfix/security-vulnerability
hotfix/critical-bug-production

# リファクタリング
refactor/auth-module-cleanup
refactor/database-abstraction
refactor/api-standardization

# ドキュメント
docs/api-documentation
docs/deployment-guide
docs/best-practices
```

#### プロジェクト固有の命名規則
```bash
# チケット番号を含む場合
feature/PROJ-123-user-dashboard
fix/PROJ-456-login-timeout

# 開発者名を含む場合（小規模チーム）
feature/alice-new-search-api
experiment/bob-performance-test

# 日付を含む場合（時系列管理）
hotfix/20241214-security-patch
experiment/20241214-cache-strategy

# 環境IDを含む場合（トレーサビリティ）
feature/env-abc123-user-auth
experiment/env-xyz789-performance
```

### 環境の効率的な管理

#### 環境リソースの監視
```bash
# 現在のアクティブ環境の確認
container-use list

# 環境のリソース使用状況確認（架空のコマンド例）
container-use stats <environment-id>

# 重いタスクの場合の設定調整例（架空のオプション例）
claude --memory 8GB --cpu 4 "大量データの処理タスクを実行"
```

#### 環境のクリーンアップ戦略
```bash
# 完了した環境の定期クリーンアップ
container-use list --status completed

# 古い環境の削除（7日以上経過したもの）
# 注意: このコマンドは架空です。実際のコマンドは確認が必要
# container-use list --older-than 7d | xargs -I {} container-use delete {}

# 手動での環境削除
container-use delete <environment-id>
```

## 🔧 CI/CD統合

### GitHub Actions との連携

#### ワークフロー例: Container-use ベースの自動テスト
```yaml
# .github/workflows/container-use-integration.yml
name: Container-use Integration Test

on:
  push:
    branches: 
      - 'feature/**'
      - 'experiment/**'
      - 'docs/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Container-use
        run: |
          # Container-useのインストール（実際のインストール方法に応じて調整）
          curl -L https://github.com/dagger/container-use/releases/latest/download/container-use-linux-amd64 -o container-use
          chmod +x container-use
          sudo mv container-use /usr/local/bin/
      
      - name: Run Tests in Container-use Environment
        run: |
          # 環境作成とテスト実行（実際のコマンドに応じて調整）
          container-use create "Automated Testing Environment" --from-branch ${{ github.ref_name }}
          container-use run-command test-env "npm test"
          container-use run-command test-env "npm run lint"
          
      - name: Collect Test Results
        run: |
          container-use checkout test-env
          # テスト結果の収集とアップロード
```

### 自動デプロイメントパイプライン

```bash
# 段階的デプロイメントのワークフロー

# 1. 開発環境での作業完了
container-use checkout development-env

# 2. ステージング環境用ブランチの作成
git branch -M deploy/staging-$(date +%Y%m%d-%H%M%S)
git push origin deploy/staging-$(date +%Y%m%d-%H%M%S)

# 3. ステージング環境でのテスト完了後、プロダクション用ブランチ作成
# （マニュアル承認後に実行）
git branch -M deploy/production-$(date +%Y%m%d-%H%M%S)
git push origin deploy/production-$(date +%Y%m%d-%H%M%S)
```

## 🎯 チーム開発ベストプラクティス

### 環境の共有と引き継ぎ

#### 環境の文書化
```bash
# 環境の詳細情報をドキュメント化
container-use log feature-env > environment-history.md
container-use diff feature-env > environment-changes.diff

# チームメンバーへの引き継ぎ情報の作成
cat > handover-notes.md << EOF
# 環境引き継ぎ情報

## 環境詳細
- Environment ID: feature-env
- ブランチ: docs/new-authentication-guide
- ステータス: レビュー準備完了
- テスト結果: 全てパス

## 注意事項
- データベースマイグレーションが必要
- 新しい環境変数 AUTH_SECRET_KEY の設定が必要

## 次のステップ
1. コードレビューの実施
2. ステージング環境でのテスト
3. プロダクションデプロイ
EOF
```

#### 環境のバックアップと復元
```bash
# 重要な環境のバックアップ（架空のコマンド例）
# container-use export feature-env feature-env-backup.tar.gz

# 必要時の復元（架空のコマンド例）
# container-use import feature-env-backup.tar.gz

# 実際は環境の履歴とdiffを保存
container-use log important-env > backup/important-env-history.log
container-use diff important-env > backup/important-env-changes.diff
```

### レビュープロセスの効率化

#### レビュー前チェックリスト
```bash
# レビュー依頼前の確認項目
cat > review-checklist.md << EOF
# レビュー前チェックリスト

## コード品質
- [ ] テストが全て通過している
- [ ] リント/フォーマットエラーがない
- [ ] コードカバレッジが基準を満たしている

## ドキュメント
- [ ] README が更新されている
- [ ] API ドキュメントが更新されている
- [ ] CHANGELOG が記載されている

## セキュリティ
- [ ] セキュリティチェックが完了している
- [ ] 機密情報がハードコードされていない
- [ ] 依存関係の脆弱性がない

## パフォーマンス
- [ ] パフォーマンステストが実行済み
- [ ] メモリリークがない
- [ ] レスポンス時間が基準内
EOF

# 自動チェックスクリプト例
container-use terminal review-env << 'EOF'
# テスト実行
npm test
if [ $? -ne 0 ]; then
  echo "❌ テストが失敗しました"
  exit 1
fi

# リントチェック
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ リントエラーがあります"
  exit 1
fi

# セキュリティチェック
npm audit --audit-level high
if [ $? -ne 0 ]; then
  echo "⚠️ セキュリティ脆弱性が検出されました"
fi

echo "✅ 基本チェックが完了しました"
EOF
```

### 競合回避戦略

#### ブランチの命名空間管理
```bash
# 開発者別の命名空間
feature/alice/user-dashboard
feature/bob/api-optimization
fix/charlie/memory-leak

# 機能別の命名空間
feature/auth/login-improvements
feature/payment/stripe-integration
feature/ui/responsive-design

# 優先度別の命名空間
hotfix/critical/security-patch
feature/high-priority/performance-boost
feature/low-priority/ui-polish

# チーム別の命名空間
feature/frontend/component-library
feature/backend/api-versioning
feature/devops/deployment-automation
```

#### 並行開発の調整
```bash
# 関連する機能の調整
# 例：認証機能とユーザー管理機能が関連する場合

# チーム内でブランチのベースを調整
git checkout feature/auth/base-implementation
git checkout -b feature/user-management/extends-auth
git push origin feature/user-management/extends-auth

# 定期的な統合テスト
container-use checkout auth-env
container-use checkout user-management-env
# 両方の変更を統合したテスト環境を作成
```

## 📊 監視とメトリクス

### 環境使用状況の追跡

```bash
# 環境の利用状況確認（架空のコマンド例）
# container-use report --period week --format json > weekly-usage.json

# 実際は手動でのログ確認
container-use list > current-environments.log
echo "$(date): 現在のアクティブ環境数: $(container-use list | wc -l)" >> usage-log.txt

# 環境の成功率分析（手動集計）
# 成功した環境数 / 総環境数 を定期的に記録
```

### 生産性メトリクス

```bash
# 開発サイクル時間の測定（手動）
# 環境作成から統合まで時間を記録

echo "Environment: docs-best-practices-123
Created: $(date '+%Y-%m-%d %H:%M:%S')
Completed: [統合時に記録]
Cycle Time: [計算して記録]" >> cycle-time-log.txt

# 環境の再利用率（手動計算）
# 削除された環境数 vs 正常に統合された環境数
```

## ⚠️ セキュリティ考慮事項

### 機密情報の取り扱い

```bash
# 環境変数での機密情報管理（架空のコマンド例）
# container-use config secret set DATABASE_PASSWORD "vault://prod/db/password"
# container-use config secret set API_KEY "op://vault/api/key"

# 機密ファイルの除外
cat > .containerignore << EOF
*.env
*.key
*.pem
config/secrets.*
.ssh/
credentials.json
EOF

# gitignore との整合性確認
cat .gitignore | grep -E "\.(env|key|pem)$|secrets|credentials"
```

### アクセス制御

```bash
# Git リポジトリレベルでのアクセス制御
# GitHub/GitLab の設定で制御

# 環境ごとのアクセス制御（将来的な機能として）
# container-use config access-control --environment production-env --users alice,bob
# container-use config access-control --environment staging-env --teams frontend,backend

# 現状では、リポジトリのアクセス権限に依存
gh repo view --json permissions
```

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### 問題: 環境が重くなりすぎた
```bash
# 症状: 環境の応答が遅い、メモリ使用量が高い

# 診断手順
container-use terminal slow-env
# 環境内でリソース使用状況を確認
top
df -h
du -sh * | sort -hr

# 解決方法1: 環境の最適化
# 不要なファイル、キャッシュの削除
npm cache clean --force
docker system prune -f  # Docker 使用時

# 解決方法2: 新しい環境での再実装
container-use delete slow-env
# 必要な変更のみを新しい環境で実装
```

#### 問題: リモートプッシュが失敗する
```bash
# 症状: git push でエラーが発生

# デバッグ情報の収集
git config --list
git remote -v
git status
git branch -vv

# 段階的な解決
# 1. リモート追跡ブランチの同期
git fetch origin

# 2. ローカルブランチの確認
git log --oneline -5

# 3. 強制プッシュが必要な場合（注意深く使用）
git push origin docs/feature-branch --force-with-lease

# 4. 新しいブランチでの再作成
BACKUP_BRANCH="backup-$(date +%s)"
git branch $BACKUP_BRANCH
git checkout -b docs/new-attempt
git push origin docs/new-attempt
```

#### 問題: 環境間での不整合
```bash
# 症状: 同じタスクで作成した環境で異なる結果

# 環境の状態確認
container-use log env1 > env1-log.txt
container-use log env2 > env2-log.txt
diff env1-log.txt env2-log.txt

container-use diff env1 > env1-diff.txt
container-use diff env2 > env2-diff.txt
diff env1-diff.txt env2-diff.txt

# 整合性の確保
# 1. 両環境の実行履歴を比較
# 2. 差異の原因を特定
# 3. 必要に応じて一方の環境を削除して再作成
```

#### 問題: Git認証エラー
```bash
# 症状: fatal: could not read Username/Password

# 解決手順1: SSH認証の確認
ssh -T git@github.com
# 結果を確認: "Hi username! You've successfully authenticated"

# 解決手順2: HTTPS認証の確認
git config --global credential.helper
# credential.helper が設定されているか確認

# 解決手順3: Personal Access Token の再設定
git config --global --unset credential.helper
git config --global credential.helper store
# 次回プッシュ時にユーザー名とPATを入力

# 解決手順4: リモートURLの確認
git remote -v
# HTTPS/SSH の適切な URL が設定されているか確認
```

## 📚 関連リソース

- [Environment Workflow ガイド](./environment-workflow-guide.md) - 基本的なワークフロー
- [Environment Workflow 実践例](./environment-workflow-examples.md) - 具体的な使用例  
- [Container-use 概要](./container-use-overview.md) - システム全体の概要
- [CLAUDE.md](./CLAUDE.md) - 開発ガイドライン

## 🎯 まとめ

このベストプラクティス集を活用することで：

### 開発効率の向上
- **安全な実験**: ローカル環境に影響を与えない隔離された開発
- **並行開発**: 複数のアプローチを同時に検証
- **迅速な反復**: 失敗した場合の高速なやり直し

### チーム連携の強化
- **標準化されたワークフロー**: チーム全体で一貫した開発プロセス
- **透明性の確保**: 全ての作業履歴が追跡可能
- **競合の回避**: 適切な命名規則とブランチ戦略

### 品質とセキュリティの向上
- **自動化されたチェック**: CI/CDパイプラインとの統合
- **段階的なレビュー**: 外部ツールとの連携によるコード品質向上
- **セキュリティの確保**: 機密情報の適切な管理

### スケーラブルな開発
- **大規模プロジェクト対応**: 複数チームでの並行開発支援
- **リソース最適化**: 効率的な環境管理とクリーンアップ
- **継続的改善**: メトリクスベースの開発プロセス改善

Container-useの真の力を発揮するために、これらのベストプラクティスを段階的に導入し、チームの開発文化に合わせてカスタマイズしていくことをお勧めします。

---

**このドキュメントは実際のContainer-use環境（informed-bedbug）で作成・検証されています。**  
**検証日**: 2024年12月14日  
**検証環境**: informed-bedbug  
**検証手法**: `git branch -M` + `git push origin` による直接リモートプッシュ