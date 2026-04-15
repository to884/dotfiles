# Azure Key Vault Integration

このプロジェクトでは、機密情報を安全に管理するためにAzure Key Vaultと連携しています。

## 必要な準備

### 1. Azure CLIのインストールと設定

```bash
# Windows (winget)
winget install Microsoft.AzureCLI

# macOS (Homebrew)  
brew install azure-cli

# Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

### 2. Azure CLIでログイン

```bash
az login
az account set --subscription "your-subscription-id"
```

### 3. Key Vaultの作成（必要に応じて）

```bash
# リソースグループ作成
az group create --name "rg-keyvault" --location "japaneast"

# Key Vault作成
az keyvault create \
    --name "your-keyvault-name" \
    --resource-group "rg-keyvault" \
    --location "japaneast"
```

## Key Vaultにシークレットを保存

### SSH関連の鍵

```bash
# SSH秘密鍵
az keyvault secret set \
    --vault-name "your-keyvault-name" \
    --name "ssh-github-private-key" \
    --file ~/.ssh/id_ed25519

# SSH公開鍵  
az keyvault secret set \
    --vault-name "your-keyvault-name" \
    --name "ssh-github-public-key" \
    --file ~/.ssh/id_ed25519.pub
```

### Git認証情報

```bash
# GitHub Personal Access Token
az keyvault secret set \
    --vault-name "your-keyvault-name" \
    --name "github-personal-access-token" \
    --value "ghp_your_token_here"

# GitHubユーザー名
az keyvault secret set \
    --vault-name "your-keyvault-name" \
    --name "github-username" \
    --value "your-github-username"
```

## chezmoi設定の更新

`~/.config/chezmoi/chezmoi.toml`を作成：

```toml
[data]
    keyVaultName = "your-keyvault-name"
    subscriptionId = "your-subscription-id"
    
    [data.git]
        name = "Your Name"
        email = "your-email@example.com"
```

## 使用方法

### 1. SSH鍵の自動取得

```bash
# SSH鍵セットアップスクリプトの実行
chezmoi execute-template ~/.ssh/setup-keys.sh.tmpl | bash

# SSH接続テスト
ssh -T git@github.com
```

### 2. 設定ファイルの更新

```bash
# chezmoi設定を適用
chezmoi apply

# Git設定の確認
git config --global --list
```

### 3. テンプレート内でのシークレット取得

chezmoi テンプレート内でAzure Key Vaultからシークレットを取得：

```go-template
{{/* API キーを取得 */}}
api_key = {{ output "azure-keyvault-secret" .keyVaultName "api-key" }}

{{/* 条件付きでシークレットを取得 */}}
{{- if .keyVaultName }}
database_password = {{ output "azure-keyvault-secret" .keyVaultName "db-password" }}
{{- end }}
```

## セキュリティのベストプラクティス

1. **アクセス制御**: Key Vaultへのアクセスは必要最小限に制限
2. **シークレット更新**: 定期的にシークレットをローテーション
3. **監査**: Key Vaultのアクセスログを定期的に確認
4. **バックアップ**: 重要なシークレットは適切にバックアップ

## トラブルシューティング

### よくある問題

1. **Azure CLIにログインしていない**
   ```bash
   az login
   ```

2. **Key Vaultへのアクセス権限がない**
   ```bash
   az keyvault set-policy --name "your-keyvault-name" --upn your-email@example.com --secret-permissions get list
   ```

3. **シークレットが見つからない**
   ```bash
   az keyvault secret list --vault-name "your-keyvault-name"
   ```

4. **chezmoi externalスクリプトが実行できない**
   ```bash
   chmod +x ~/.local/share/chezmoi/.chezmoiexternal/azure-keyvault-secret
   ```
