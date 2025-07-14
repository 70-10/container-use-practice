# Container-use 環境設定ガイド

## 概要

Container-useの環境設定機能を使用すると、すべてのエージェントセッションで使用するデフォルト環境を定義できます。これにより、特定のプロジェクトのニーズに合わせてベースイメージ、依存関係、セットアップをカスタマイズできます。

## 設定オプション

### 1. ベースイメージの設定

実行環境（Python、Node.js、Go、Ubuntuなど）を選択します。

```bash
# Python環境の設定
container-use config base-image set python:3.11

# Node.js環境の設定
container-use config base-image set node:18

# Ubuntu環境の設定
container-use config base-image set ubuntu:22.04

# Go環境の設定
container-use config base-image set golang:1.21
```

### 2. セットアップコマンドの設定

インストールおよび設定スクリプトを実行します。

```bash
# Pythonプロジェクトの例
container-use config setup-command add "pip install -r requirements.txt"
container-use config setup-command add "pip install pytest black"

# Node.jsプロジェクトの例
container-use config setup-command add "npm install"
container-use config setup-command add "npm run build"

# システムパッケージのインストール例
container-use config setup-command add "apt-get update && apt-get install -y git curl"
```

#### セットアップコマンドのベストプラクティス

- **冪等性**: コマンドを複数回実行しても同じ結果になるようにする
- **パッケージマネージャーの使用**: pipやnpmなどの標準ツールを活用
- **論理的な順序**: 依存関係を考慮してコマンドを配置
- **エラーハンドリング**: 必要に応じて`&&`や`||`を使用

### 3. 環境変数の設定

開発設定やパスを設定します。

```bash
# Python環境の例
container-use config env set DEBUG true
container-use config env set PYTHONPATH /workdir

# Node.js環境の例
container-use config env set NODE_ENV development
container-use config env set PORT 3000

# 一般的な設定例
container-use config env set LANG ja_JP.UTF-8
container-use config env set TZ Asia/Tokyo
```

#### 環境変数のベストプラクティス

- **標準的な命名規則**: 大文字とアンダースコアを使用
- **機密情報の回避**: パスワードやAPIキーは環境変数に保存しない
- **開発専用設定**: 開発環境特有の設定に使用

## プロジェクト別設定パターン

### Python プロジェクト

```bash
# ベースイメージの設定
container-use config base-image set python:3.11

# 依存関係のインストール
container-use config setup-command add "pip install -r requirements.txt"
container-use config setup-command add "pip install pytest black flake8"

# 環境変数の設定
container-use config env set PYTHONPATH /workdir
container-use config env set DEBUG true
```

### Node.js プロジェクト

```bash
# ベースイメージの設定
container-use config base-image set node:18

# 依存関係のインストール
container-use config setup-command add "npm install"
container-use config setup-command add "npm run build"

# 環境変数の設定
container-use config env set NODE_ENV development
container-use config env set PORT 3000
```

### Go プロジェクト

```bash
# ベースイメージの設定
container-use config base-image set golang:1.21

# 依存関係のダウンロード
container-use config setup-command add "go mod download"
container-use config setup-command add "go mod tidy"

# 環境変数の設定
container-use config env set GOOS linux
container-use config env set GOARCH amd64
```

### フルスタック Web アプリケーション

```bash
# ベースイメージの設定
container-use config base-image set ubuntu:22.04

# システムレベルの依存関係
container-use config setup-command add "apt-get update && apt-get install -y curl git"
container-use config setup-command add "curl -fsSL https://deb.nodesource.com/setup_18.x | bash -"
container-use config setup-command add "apt-get install -y nodejs python3 python3-pip"

# プロジェクト依存関係
container-use config setup-command add "npm install"
container-use config setup-command add "pip3 install -r requirements.txt"

# 環境変数の設定
container-use config env set NODE_ENV development
container-use config env set DEBUG true
```

## 管理コマンド

### 設定の確認

```bash
# 現在の設定を表示
container-use config show

# 特定の設定項目を確認
container-use config base-image show
container-use config setup-command list
container-use config env list
```

### 設定の削除

```bash
# 特定のセットアップコマンドを削除
container-use config setup-command remove "pip install pytest"

# 特定の環境変数を削除
container-use config env unset DEBUG

# すべての設定をリセット
container-use config reset
```

## 重要な注意事項

### 新環境への適用

設定は**新しく作成される環境にのみ適用**されます。既存の環境には影響しません。

```bash
# 設定変更後、新環境で反映される
container-use config base-image set python:3.11
container-use create  # 新しい設定が適用される
```

### 設定の優先順位

1. **プロジェクト固有の設定**: プロジェクトディレクトリ内の設定
2. **グローバル設定**: ユーザーレベルの設定
3. **デフォルト設定**: Container-useの標準設定

## トラブルシューティング

### 一般的な問題と解決方法

#### セットアップコマンドの失敗

```bash
# パッケージマネージャーのアップデート
container-use config setup-command add "apt-get update"
container-use config setup-command add "pip install --upgrade pip"

# タイムアウトの対策
container-use config setup-command add "pip install --timeout 300 -r requirements.txt"
```

#### 権限エラー

```bash
# sudoが必要な場合
container-use config setup-command add "sudo apt-get install -y package-name"

# ユーザー権限でのインストール
container-use config setup-command add "pip install --user package-name"
```

#### ネットワークの問題

```bash
# プロキシ設定
container-use config env set HTTP_PROXY http://proxy.example.com:8080
container-use config env set HTTPS_PROXY http://proxy.example.com:8080

# DNSの設定
container-use config setup-command add "echo 'nameserver 8.8.8.8' >> /etc/resolv.conf"
```

### デバッグ方法

```bash
# 詳細ログの有効化
container-use config env set CONTAINER_USE_DEBUG true

# 環境作成時のログ確認
container-use create --verbose

# 設定の段階的テスト
container-use config setup-command add "echo 'Step 1 completed'"
```

## ベストプラクティス

### 設定管理

1. **設定の文書化**: チーム内で設定内容を共有
2. **バージョン管理**: 設定変更の履歴を保持
3. **段階的適用**: 大きな変更は段階的に実施

### パフォーマンス最適化

1. **イメージサイズの最小化**: 必要最小限のベースイメージを選択
2. **キャッシュの活用**: レイヤーキャッシュを意識したコマンド順序
3. **並列処理**: 可能な場合は並列でインストール

### セキュリティ考慮事項

1. **最新バージョンの使用**: ベースイメージとパッケージの定期更新
2. **機密情報の保護**: 環境変数に機密情報を保存しない
3. **最小権限の原則**: 必要最小限の権限でコマンドを実行

## 応用例

### CI/CD統合

```bash
# テスト環境の設定
container-use config base-image set python:3.11-slim
container-use config setup-command add "pip install -r requirements.txt"
container-use config setup-command add "pip install pytest coverage"
container-use config env set CI true
```

### マイクロサービス開発

```bash
# 各サービス用の設定プロファイル
container-use config base-image set node:18-alpine
container-use config setup-command add "npm install"
container-use config env set SERVICE_NAME user-service
container-use config env set PORT 3001
```

このガイドを参考に、プロジェクトに最適な環境設定を構築してください。