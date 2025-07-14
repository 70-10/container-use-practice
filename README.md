# Container-use Practice

Container-useプラットフォームの理解と実践的な使用法を学ぶためのドキュメント集です。

## 📚 ドキュメント構成

### 1. [Container-use 概要](./container-use-overview.md)
- Container-useプラットフォームの基本概念
- 主要機能と技術的基盤
- 利用メリットとコミュニティ情報

### 2. [Environment Workflow ガイド](./environment-workflow-guide.md)
- Environment Workflowの詳細解説
- 環境の構成要素（Git Branch、Container、履歴）
- ワークフローの4つのフェーズ
- ベストプラクティスと重要コマンド

### 3. [Environment Workflow 実践例](./environment-workflow-examples.md)
- 実際のワークフロー例（機能実装、バグ修正、複数アプローチ比較）
- トラブルシューティングガイド
- パフォーマンス最適化のコツ
- 監視とレポート機能

### 4. [Environment Configuration Guide](./docs/environment-configuration.md)
- Container-use環境設定の詳細ガイド
- ベースイメージ、セットアップコマンド、環境変数の設定方法
- プロジェクト別設定パターン（Python、Node.js、Go、フルスタック）
- トラブルシューティングとベストプラクティス

### 5. [Container-use Best Practices](./container-use-best-practices.md)
- 高度なワークフローとベストプラクティス
- 直接リモートプッシュ手法と複数環境の並行管理
- CI/CD統合とチーム開発での活用方法
- セキュリティ考慮事項とトラブルシューティング

## 🚀 クイックスタート

### 基本的な使用の流れ

1. **環境の確認**
   ```bash
   container-use list
   ```

2. **エージェントに作業指示**
   ```bash
   claude "新しい機能を実装してください"
   ```

3. **作業内容の確認**
   ```bash
   container-use diff <environment-id>
   container-use log <environment-id>
   ```

4. **作業の統合**
   ```bash
   container-use merge <environment-id>  # または apply
   ```

### よく使うコマンド

| コマンド | 用途 |
|---------|------|
| `container-use list` | 環境一覧表示 |
| `container-use diff <env-id>` | 変更差分確認 |
| `container-use log <env-id>` | コミット履歴確認 |
| `container-use terminal <env-id>` | 環境への直接アクセス |
| `container-use merge <env-id>` | 履歴保持統合 |
| `container-use apply <env-id>` | カスタム統合 |
| `container-use delete <env-id>` | 環境削除 |

## 🎯 学習パス

### 初心者向け
1. [概要](./container-use-overview.md)でContainer-useの基本を理解
2. [ワークフローガイド](./environment-workflow-guide.md)で作業の流れを学習
3. [実践例](./environment-workflow-examples.md)の「例1: 新機能の実装」を実践

### 中級者向け
1. 複数アプローチの比較実験を実践
2. トラブルシューティング手法を習得
3. パフォーマンス最適化テクニックを活用

### 上級者向け
1. 複雑なワークフローの設計
2. チーム開発での環境管理
3. カスタム統合戦略の開発

## 🛠 実践的なユースケース

### 開発シナリオ別ガイド

- **新機能開発**: 分離された環境での安全な実験
- **バグ修正**: 迅速な問題解決とテスト
- **リファクタリング**: 段階的な改善と品質保証
- **パフォーマンス最適化**: 複数手法の比較検討
- **セキュリティ更新**: 慎重な修正と検証

### チーム開発での活用

- 複数メンバーでの並行開発
- コードレビューの効率化
- 実験的機能の評価プロセス

## 📖 参考リソース

### 公式ドキュメント
- [Container-use 公式サイト](https://container-use.com/)
- [Environment Workflow](https://container-use.com/environment-workflow)

### コミュニティ
- Discord: #container-useチャンネル
- GitHub: リポジトリとDiscussions
- YouTube: チュートリアルプレイリスト

## 🔧 設定とカスタマイズ

### 環境設定
- `.mcp.json`: MCP（Model Context Protocol）設定
- `CLAUDE.md`: プロジェクト固有の指示

### カスタマイズ例
- 特定のワークフローに最適化したコマンドエイリアス
- プロジェクト固有のテンプレート環境
- 自動化されたテストと統合パイプライン

---

**注意**: このプロジェクトは学習と実践を目的としています。本番環境での使用前に、十分なテストと検証を行ってください。

## 作業確認方法

この環境での作業内容を確認するには：

```bash
# ログの確認
container-use log loved-satyr

# 差分の確認  
container-use diff loved-satyr

# 環境への直接アクセス
container-use checkout loved-satyr
```