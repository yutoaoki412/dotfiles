# dotfiles

macOS の開発環境設定を [chezmoi](https://www.chezmoi.io) で管理するリポジトリです。
私用 Mac・会社用 Mac のどちらでも、同じシェル・Git・ターミナル環境を再現することを目的としています。

> 対象環境: Apple Silicon Mac（Homebrew が `/opt/homebrew` にインストールされている前提）

---

## 管理しているもの

| 実際のパス | リポジトリ内のファイル | 内容 |
|-----------|----------------------|------|
| `~/.zshrc` | `dot_zshrc` | PATH 設定・starship 起動・direnv フック |
| `~/.zprofile` | `dot_zprofile` | ログインシェル設定（Homebrew の PATH）|
| `~/.gitconfig` | `dot_gitconfig.tmpl` | Git ユーザー情報・global gitignore 設定 |
| `~/.gitignore_global` | `dot_gitignore_global` | 全リポジトリ共通で無視するファイル（`.envrc`, `.direnv/` 等） |
| `~/.config/starship.toml` | `dot_config/starship.toml` | プロンプト表示設定（gcloud アカウント表示含む） |
| `~/Library/Application Support/com.mitchellh.ghostty/config.ghostty` | `private_Library/.../config.ghostty` | Ghostty ターミナル設定 |
| （リポジトリ直下） | `Brewfile` | Homebrew パッケージ・Cask・拡張機能一覧 |

### Brewfile に含まれる主なツール

| ツール | 用途 |
|--------|------|
| `chezmoi` | この dotfiles 自体の管理 |
| `starship` | ターミナルプロンプト |
| `direnv` | ディレクトリごとの環境変数管理（GCP アカウント切替等） |
| `gh` | GitHub CLI |
| `awscli` | AWS CLI |
| `gcloud-cli` | Google Cloud CLI |
| `go`, `node`, `ripgrep` | 開発ツール |

### `.envrc` を global gitignore で無視する理由

各プロジェクトの `.envrc` は [gcp-account-manager](https://github.com/yutoaoki412/gcp-account-manager) が管理する個人のワークフロー設定です。チームリポジトリを含む全リポジトリで Git の追跡対象外にすることで、誤ってコミットすることを防いでいます。

---

## セットアップ（新しい Mac で初めて使う）

### 前提条件

Homebrew と chezmoi がインストールされていること。

```bash
# Homebrew をインストール
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# chezmoi をインストール
brew install chezmoi
```

### 手順

`chezmoi apply` は既存の `~/.zshrc` や `~/.gitconfig` などを**上書きします**。初回は差分を確認しながら進めることを推奨します。

```bash
# 1. リポジトリを取得（ホームディレクトリはまだ変更されない）
chezmoi init yutoaoki412

# 2. 変更内容を確認
chezmoi diff

# 3. 問題なければ反映
chezmoi apply
```

実行中に **「Git email address:」** と聞かれます。Git コミットに使うメールアドレスを入力してください。

> 確認不要の場合: `chezmoi init --apply yutoaoki412` で一発完了

### セットアップ後にやること

1. シェルを再起動して `direnv` フックを有効化する
2. GCP アカウント切替が必要な場合は [gcp-account-manager](https://github.com/yutoaoki412/gcp-account-manager) を参照

### ツールから自動追加される行について

`dbt Fusion` 等の一部 IDE 拡張は `~/.zshrc` に自動で行を追記します。これらは `chezmoi apply` で一度消えますが、拡張を使用するときに再度追記されます。意図した動作です。

---

## 日常の使い方

### 設定ファイルを変更したとき

設定ファイルは直接 `~/.config/...` 等を編集して OK です。編集後に chezmoi へ取り込みます。

```bash
# 全ファイル一括で取り込む
chezmoi re-add

# 特定ファイルだけ取り込む
chezmoi re-add ~/.zshrc
chezmoi re-add ~/.config/starship.toml
```

変更を確認してからコミット・プッシュします。

```bash
chezmoi cd          # dotfiles リポジトリに移動
git diff
git add .
git commit -m "feat: ..."
git push
exit                # 元のディレクトリに戻る
```

### 別の Mac で最新設定を受け取る

```bash
chezmoi update
```

### Brewfile を更新したとき

```bash
# 現在インストール済みのパッケージから Brewfile を再生成
brew bundle dump --force --file=~/.local/share/chezmoi/Brewfile

chezmoi cd
git add Brewfile
git commit -m "chore: update Brewfile"
git push
exit
```

> `Brewfile` が変更されると、次回 `chezmoi apply` / `chezmoi update` 時に `brew bundle install` が自動実行されます。

---

## 仕組みの解説

### chezmoi のファイル命名規則

chezmoi はファイル名のプレフィックス・サフィックスで管理方法を判断します。

| プレフィックス / サフィックス | 意味 |
|--------------------------|------|
| `dot_` | `~/.` に展開（例: `dot_zshrc` → `~/.zshrc`） |
| `private_` | パーミッション `600` で展開 |
| `.tmpl` | テンプレートとして処理（`{{ }}` が値に置換） |
| `run_onchange_` | ファイルの内容が変わったときだけ実行されるスクリプト |

### メールアドレスの仕組み

`chezmoi init` 時に入力したメールアドレスが `~/.config/chezmoi/chezmoi.toml` に保存されます。`dot_gitconfig.tmpl` がそれを参照し、マシンごとに異なるメールアドレスを設定できます。

### global gitignore の仕組み

`dot_gitignore_global` → `~/.gitignore_global` に展開されます。  
`dot_gitconfig.tmpl` の `[core] excludesfile = ~/.gitignore_global` によって、全リポジトリで自動的に適用されます。  
`.envrc` や `.direnv/` はプロジェクトごとの `.gitignore` を触らなくても無視されます。

---

## コマンドリファレンス

| コマンド | 何をするか |
|---------|-----------|
| `chezmoi init yutoaoki412` | リポジトリを取得（ホームディレクトリは変更しない） |
| `chezmoi init --apply yutoaoki412` | 新しい Mac で一発セットアップ |
| `chezmoi diff` | chezmoi の状態と実ファイルの差分を確認 |
| `chezmoi apply` | chezmoi の状態をホームディレクトリに反映 |
| `chezmoi re-add` | 実ファイルの変更を chezmoi に取り込む |
| `chezmoi edit ~/.zshrc` | chezmoi 経由でファイルを編集 |
| `chezmoi cd` | dotfiles リポジトリ（`~/.local/share/chezmoi`）に移動 |
| `chezmoi status` | 管理ファイルの変更状況を一覧表示 |
| `chezmoi update` | リモートの変更を pull してローカルに反映 |

---

## リポジトリ構成

```
dotfiles/
├── .chezmoi.toml.tmpl                     # 初回セットアップ時に Git メールアドレスを入力させるテンプレート
├── Brewfile                                # Homebrew パッケージ・Cask・拡張機能・ツール一覧
├── run_onchange_install-packages.sh.tmpl   # Brewfile 変更時に自動実行される brew bundle install
├── dot_config/
│   └── starship.toml                       # → ~/.config/starship.toml
├── dot_gitconfig.tmpl                      # → ~/.gitconfig（メールアドレスはマシンごとに異なる）
├── dot_gitignore_global                    # → ~/.gitignore_global（.envrc 等を全リポジトリで無視）
├── dot_zprofile                            # → ~/.zprofile
├── dot_zshrc                               # → ~/.zshrc（direnv フック含む）
└── private_Library/
    └── private_Application Support/
        └── com.mitchellh.ghostty/
            └── config.ghostty               # → ~/Library/Application Support/.../config.ghostty
```
