# Environment Workflow ガイド

## 概要

Environment Workflowは、Container-useにおける作業の流れを体系化したものです。各環境は**使い捨てのサンドボックス**として機能し、安全で効率的な開発を可能にします。

## 環境の構成要素

各環境は以下の3つの要素から構成されます：

### 🌳 Git Branch（ブランチ）
- 変更履歴とコードの追跡
- 標準的なGitワークフローとの統合
- メインブランチからの分離と安全な実験

### 📦 Container（コンテナ）
- 分離された実行環境
- コードと依存関係を含む完全な開発環境
- 他の作業への影響を排除

### 📝 Complete History（完全な履歴）
- 実行されたすべてのコマンドの記録
- コンテナの状態変更の追跡
- 作業プロセスの完全な可視性

## ワークフローの流れ

### 1. エージェントが新しい環境を作成

```mermaid
graph LR
    A[メインブランチ] --> B[新しい環境]
    B --> C[分離されたコンテナ]
```

- 現在のブランチ状態から開始
- メインのコードベースから完全に分離
- クリーンな状態での作業開始

### 2. 分離された環境での作業

エージェントは以下の作業を安全に実行できます：
- ✅ コードの変更
- ✅ 依存関係の追加・変更
- ✅ 実験的な実装
- ✅ テストの実行

既存の作業への干渉は一切ありません。

### 3. 作業内容の観察

以下のコマンドで作業内容を確認できます：

#### 環境一覧の確認
```bash
container-use list
```

#### 変更履歴の確認
```bash
container-use log <environment-id>
```

#### コード変更の確認
```bash
container-use diff <environment-id>
```

#### インタラクティブな探索
```bash
container-use terminal <environment-id>
```

### 4. 意思決定フェーズ

作業内容を確認した後、以下の3つの選択肢があります：

#### ✅ 作業を受け入れる

**Merge（マージ）**: エージェントのコミット履歴を保持
```bash
container-use merge <environment-id>
```

**Apply（適用）**: カスタムコミットメッセージで統合
```bash
container-use apply <environment-id>
```

#### 🔄 反復と改善

同じ環境で作業を続行：
- 不足している機能の追加
- バグの修正
- 要件の調整

#### 🗑️ 新たな開始

問題のある環境を削除して再開：
```bash
container-use delete <environment-id>
```

## ベストプラクティス

### 効率的な評価方法

1. **クイック評価コマンドから開始**
   ```bash
   container-use list
   container-use diff <env-id>
   ```

2. **詳細確認が必要な場合**
   ```bash
   container-use log <env-id>
   container-use terminal <env-id>
   ```

### コミット履歴の管理

- **Merge**: 詳細な作業履歴を保持したい場合
- **Apply**: 統一されたコミットメッセージを作成したい場合

### 環境の削除タイミング

以下の場合は環境を削除して再開を検討：
- 期待した結果が得られない
- 方向性が間違っている
- より良いアプローチを思いついた

### メインブランチの保護

- テストされた作業のみをメインブランチに統合
- 実験的な変更は環境内で完結
- 安定性を保つための慎重な統合

## よくある使用パターン

### パターン1: 機能開発

```bash
# 1. 環境作成
container-use create "新機能の実装"

# 2. 作業確認
container-use diff <env-id>

# 3. 受け入れ
container-use merge <env-id>
```

### パターン2: 実験的な変更

```bash
# 1. 複数の環境で異なるアプローチを試行
container-use create "アプローチA"
container-use create "アプローチB"

# 2. 最適な結果を選択
container-use apply <best-env-id>

# 3. 他の環境を削除
container-use delete <other-env-id>
```

### パターン3: 段階的な改善

```bash
# 1. 初期実装
container-use create "基本実装"

# 2. 段階的改善（同じ環境で継続）
# エージェントに追加要件を指示

# 3. 最終確認と統合
container-use log <env-id>
container-use merge <env-id>
```

## 重要なコマンド一覧

| コマンド | 説明 | 使用タイミング |
|---------|------|-------------|
| `container-use list` | 環境一覧表示 | 作業開始時 |
| `container-use log <env-id>` | コミット履歴表示 | 詳細確認時 |
| `container-use diff <env-id>` | 変更差分表示 | クイック確認時 |
| `container-use terminal <env-id>` | インタラクティブアクセス | 深い調査時 |
| `container-use merge <env-id>` | 履歴保持統合 | 詳細履歴重視時 |
| `container-use apply <env-id>` | カスタム統合 | 統一メッセージ重視時 |
| `container-use delete <env-id>` | 環境削除 | やり直し時 |

## まとめ

Environment Workflowは、安全で効率的な開発を可能にする強力なパラダイムです：

- **安全性**: 分離された環境での実験
- **可視性**: 完全な作業履歴の追跡
- **柔軟性**: 複数の統合オプション
- **効率性**: 並行開発と迅速な反復

このワークフローを活用することで、AIエージェントとの協力的な開発を最大限に活用できます。