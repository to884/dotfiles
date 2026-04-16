# Dotfiles - chezmoi で管理する個人設定

このディレクトリには、chezmoiで管理される個人設定ファイル（dotfiles）が含まれています。

## 📋 概要

このdotfilesリポジトリは、開発環境の個人設定を複数のマシン間で同期・管理するためのものです。  
[chezmoi](https://www.chezmoi.io/)を使用して、設定ファイルのバージョン管理とデプロイを行います。

## ✨ 管理される設定ファイル

### シェル設定
- `~/.bashrc` - Bash設定（Linuxbrew、starshipの初期化）
- `~/.zshrc` - Zsh設定（Linuxbrew、starshipの初期化）

### Git設定
- `~/.gitconfig` - Git全体設定（delta、エイリアス、ユーザー情報）

### curl設定
- `~/.curlrc` - curl全体設定（Zscaler証明書パス）

### SSH設定
- `~/.ssh/config` - SSH クライアント設定（接続設定、ホスト別設定、セキュリティ設定）
- `~/.ssh/setup-keys.sh` - SSHキーセットアップスクリプト（Azure Key Vault連携）

### ターミナルツール設定
- `~/.tmux.conf` - tmux設定（キーバインド、プラグイン）
- `~/.config/powerline/` - PowerLine設定（ステータスバー）
- `~/.config/starship.toml` - starshipプロンプト設定
- `~/.config/topgrade.toml` - topgrade設定（パッケージ一括更新）

## 🚀 セットアップ

### 前提条件

1. **chezmoiが利用可能であること**
   ```bash
   which chezmoi
   # 例）linuxbrew を使用してインストールした場合：
   # /home/linuxbrew/.linuxbrew/bin/chezmoi
   ```

3. **Nerd Fontのインストールと設定**
   - Starship、Powerline、Tmuxなどのツールが、プロンプトやステータスバーでアイコンを正しく表示するために必要です
   - 推奨フォント:
     - [Hack Nerd Font](https://www.nerdfonts.com/)
     - [JetBrains Mono Nerd Font](https://www.nerdfonts.com/)
     - [Fira Code Nerd Font](https://www.nerdfonts.com/)
   - インストール方法: [Nerd Fonts公式サイト](https://www.nerdfonts.com/) を参照
   - インストール後、ターミナルエミュレータの設定でNerd Fontを選択してください
   - ⚠️ アイコンが正しく表示されない場合は、フォント設定を確認してください

### 初回セットアップ

#### 1. このリポジトリからdotfilesを初期化

```bash
# 任意のdotfilesディレクトリを使用する場合
chezmoi init /path/to/dotfiles

# GitHubリポジトリから初期化する場合
chezmoi init https://github.com/your-username/dotfiles.git
```

#### 2. 変更内容を確認

```bash
# dry-runモードで確認
chezmoi diff
```

#### 3. 設定を適用

```bash
# 設定ファイルを適用
chezmoi apply -v
```

#### 4. シークレット（機密情報）の配置

chezmoi適用後、SSH鍵などのシークレットを手動で配置します：

```bash
# SSH鍵のセットアップ状況を確認
~/.ssh/setup-keys.sh

# 新しい鍵を生成する場合
ssh-keygen -t ed25519 -C "your.email@example.com"

# または、既存の鍵をコピー（Windows から WSL の場合）
cp /mnt/c/Users/YourName/.ssh/id_ed25519 ~/.ssh/
chmod 600 ~/.ssh/id_ed25519
```

詳細は「[🔒 セキュリティとシークレット管理](#-セキュリティとシークレット管理)」セクションを参照してください。

### 日常的な使い方

#### 設定ファイルを編集

```bash
# chezmoiエディタで編集
chezmoi edit ~/.bashrc

# または直接編集
chezmoi edit --apply ~/.gitconfig
```

#### 変更を確認して適用

```bash
# 変更内容を確認
chezmoi diff

# 変更を適用
chezmoi apply -v
```

#### 新しいマシンで設定を同期

```bash
# GitHubリポジトリから初期化
chezmoi init https://github.com/your-username/dotfiles.git

# 適用
chezmoi apply -v

# シークレットの配置（手動）
~/.ssh/setup-keys.sh
# 上記のガイドに従ってSSH鍵などを配置
```

## 📁 ディレクトリ構造

```
dotfiles/
├── README.md                    # このファイル
├── AZURE_KEY_VAULT.md          # Azure Key Vault連携ドキュメント
├── .chezmoi.toml.tmpl          # chezmoi設定（テンプレート）
├── .chezmoiignore              # chezmoi除外ファイル
├── chezmoi.toml.example        # chezmoi設定例（Azure Key Vault用）
├── dot_bashrc.tmpl             # ~/.bashrc
├── dot_zshrc.tmpl              # ~/.zshrc
├── dot_gitconfig.tmpl          # ~/.gitconfig
├── dot_curlrc.tmpl             # ~/.curlrc（Zscaler証明書設定）
├── dot_tmux.conf               # ~/.tmux.conf
├── dot_ssh/
│   ├── config.tmpl             # ~/.ssh/config
│   └── setup-keys.sh.tmpl      # ~/.ssh/setup-keys.sh（Azure Key Vault連携）
├── dot_config/
│   ├── starship.toml           # ~/.config/starship.toml
│   ├── topgrade.toml           # ~/.config/topgrade.toml
│   └── powerline/              # ~/.config/powerline/
│       ├── config.json
│       ├── colors.json
│       ├── colorschemes/
│       │   └── tmux/
│       │       └── tmux-colorscheme.json
│       └── themes/
│           └── tmux/
│               └── tmux-theme.json
├── .chezmoiexternal/
│   └── azure-keyvault-secret   # Azure Key Vault連携用
├── .agents/
│   └── skills/                 # AI エージェントスキル定義
└── .github/
    └── instructions/           # GitHub Copilot インストラクション
```

## 🔄 Ansibleとの関係

### 役割分担

| 管理対象 | ツール | 理由 |
|---------|--------|------|
| システムパッケージ | Ansible | べき等性、root権限が必要 |
| Linuxbrewパッケージ | Ansible | 環境構築の一部 |
| 個人設定ファイル | chezmoi | ユーザー設定、頻繁な変更 |
| システム設定 | Ansible | sudoers、SSH、Docker等 |

### ワークフロー

```bash
# 1. システム基盤をAnsibleで構築
ansible-playbook -i inventories/wsl/hosts site.yml

# 2. 個人設定をchezmoiで適用
chezmoi init https://github.com/your-username/dotfiles.git
chezmoi apply
```

## 📖 参考リンク

- [chezmoi公式ドキュメント](https://www.chezmoi.io/)
- [chezmoi Quick Start](https://www.chezmoi.io/quick-start/)
- [chezmoi User Guide](https://www.chezmoi.io/user-guide/setup/)

## 🔧 カスタマイズ

### ユーザー情報の設定

`.chezmoi.toml.tmpl` でユーザー固有の情報を管理できます。

初回実行時に以下の情報を対話的に入力します：
- Windows ユーザー名（WSL環境の場合）
- Git 氏名
- Git メールアドレス
- GitHub ユーザー名（オプション）
- SSH秘密鍵のファイル名（デフォルト: `id_ed25519`）
- SSH公開鍵のファイル名（デフォルト: `id_ed25519.pub`）

生成される設定ファイルの構造：

```toml
[data]
    [data.git]
        name = "Your Name"
        email = "your.email@example.com"
    [data.github]
        user = "your-github-username"

    [data.ssh]
        # SSH鍵のファイル名（~/.ssh/ 配下）
        privateKeyFile = "id_ed25519"
        publicKeyFile = "id_ed25519.pub"

[data.os]
    homePrefix = "/home"  # Windowsの場合は "C:/Users"
```

これらの変数は `.tmpl` ファイル内で使用できます：

```bash
# dot_gitconfig.tmpl
[user]
    name = {{ .git.name }}
    email = {{ .git.email }}

# dot_ssh/setup-keys.sh.tmpl
SSH_PRIVATE_KEY="{{ .ssh.privateKeyFile }}"
SSH_PUBLIC_KEY="{{ .ssh.publicKeyFile }}"
```

### 主要な設定ファイルの説明

#### topgrade 設定 (`~/.config/topgrade.toml`)

topgrade は様々なパッケージマネージャーを一括で更新するツールです。

```bash
# すべてのパッケージを更新
topgrade

# dry-run モード（実際には更新しない）
topgrade --dry-run

# 特定のステップのみ実行
topgrade --only brew

# 特定のステップをスキップ
topgrade --disable system
```

設定ファイルで無効化・カスタマイズ可能：
- Homebrew の自動クリーンアップ
- システムパッケージの更新（WSL環境では無効化推奨）
- Git リポジトリの一括 pull
- カスタムコマンドの実行

参考: [topgrade GitHub](https://github.com/topgrade-rs/topgrade)

#### SSH 設定 (`~/.ssh/config`)

SSH 接続設定を一元管理：
- グローバル設定（接続維持、圧縮、タイムアウト）
- ホスト別の設定（GitHub、開発サーバー、ジャンプホスト）
- WSL 環境での特殊設定（Windows の SSH エージェント利用）

設定例を参考にカスタマイズしてください。

## 🔒 セキュリティとシークレット管理

### 基本方針

このdotfilesリポジトリでは、**シークレット（機密情報）はchezmoiで管理しません**。  
すべてのシークレットは、ユーザーが手動で適切な場所に配置する必要があります。

### 除外されるファイル

`.chezmoiignore` で以下のファイルが自動的に除外されます：

- **SSH関連**
  - `~/.ssh/id_*` （秘密鍵）
  - `~/.ssh/known_hosts`
  - `~/.ssh/authorized_keys`
  
- **認証情報**
  - `~/.aws/credentials` （AWS認証情報）
  - `~/.config/gh/hosts.yml` （GitHub CLI認証）
  
- **暗号化関連**
  - `~/.gnupg/*` （GPG鍵）
  - `~/.password-store/*` （pass パスワードストア）

### シークレットのセットアップ方法

#### 1. SSH鍵のセットアップ

セットアップガイドスクリプトを実行して、現在の状態を確認できます：

```bash
# chezmoi適用後に実行
~/.ssh/setup-keys.sh
```

**新しい鍵を生成する場合：**

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

**既存の鍵をコピーする場合：**

```bash
# Windowsから WSL へ
cp /mnt/c/Users/YourName/.ssh/id_ed25519 ~/.ssh/
chmod 600 ~/.ssh/id_ed25519

# 別のマシンから scp で
scp user@remote:~/.ssh/id_ed25519 ~/.ssh/
chmod 600 ~/.ssh/id_ed25519
```

#### 2. その他のシークレット

必要に応じて手動で配置してください：

```bash
# AWS認証情報
mkdir -p ~/.aws
# credentials ファイルを配置
chmod 600 ~/.aws/credentials

# GitHub CLI認証
# gh auth login を実行するか、手動で配置
mkdir -p ~/.config/gh

# GPG鍵
# gpg --import を使用するか、既存の ~/.gnupg をコピー
```

### セキュリティのベストプラクティス

- ✅ 機密情報は**絶対に**Gitリポジトリにコミットしない
- ✅ `.chezmoiignore` で確実に除外されていることを確認
- ✅ SSH秘密鍵のパーミッションは `600` に設定
- ✅ `.gitconfig` のユーザー情報はテンプレート変数を使用
- ⚠️ どうしても暗号化して管理したい場合は、chezmoi の [age暗号化機能](https://www.chezmoi.io/user-guide/encryption/) を検討

### Azure Key Vault連携（オプション）

自動的なシークレット管理が必要な場合は、[AZURE_KEY_VAULT.md](AZURE_KEY_VAULT.md) を参照してください。  
ただし、基本的には手動配置を推奨します。

## 🔐 Zscaler環境での使用

Zscaler等のSSLインスペクションを使用する企業環境では、curl等のHTTPSツールが証明書エラーで失敗することがあります。  
このdotfilesリポジトリには、Zscaler証明書を自動的に設定する `.curlrc` が含まれています。

### セットアップ手順

1. **Zscaler証明書ファイルを配置**
   ```bash
   mkdir -p ~/.certs
   # Windows環境から証明書をコピー（例）
   cp /mnt/c/Users/YourName/Documents/ZscalerRootCA.cer ~/.certs/
   ```

2. **bootstrap.sh を実行して証明書を変換・登録**
   ```bash
   cd /path/to/devenv-bootstrap
   ./bootstrap.sh
   ```
   
   このスクリプトが以下を実行します：
   - `.cer` 形式を `.pem` 形式に変換
   - システム証明書ストアに登録
   - `~/.certs/ZscalerRootCA.pem` を作成

3. **dotfilesを適用（chezmoiで自動）**
   ```bash
   chezmoi apply
   ```
   
   これにより `~/.curlrc` が作成され、以下の設定が適用されます：
   ```
   cacert = ~/.certs/ZscalerRootCA.pem
   ```

### 動作確認

```bash
# curl でHTTPSサイトにアクセス
curl -I https://github.com

# 証明書検証が正常に動作すれば成功
# HTTP/2 200 のレスポンスが返ります
```

### トラブルシューティング

証明書エラーが発生する場合：

```bash
# 証明書の確認
ls -la ~/.certs/ZscalerRootCA.pem

# .curlrc の確認
cat ~/.curlrc

# システム証明書ストアの確認
./scripts/verify-zscaler-cert.sh --verbose
```

参考: [Zscaler証明書検証スクリプト](../scripts/verify-zscaler-cert.sh)
