# dotfiles

[chezmoi](https://www.chezmoi.io) で管理する個人の dotfiles リポジトリです。
私用 Mac・会社用 Mac の両方で、**1コマンド**で同じ開発環境を再現できます。

## 管理しているファイル

| 実際のパス | リポジトリ内のファイル | 説明 |
|-----------|----------------------|------|
| `~/Library/Application Support/com.mitchellh.ghostty/config.ghostty` | `private_Library/...` | Ghostty ターミナル設定 |
| `~/.config/starship.toml` | `dot_config/starship.toml` | Starship プロンプト設定 |
| `~/.zshrc` | `dot_zshrc` | zsh 設定 |
| `~/.zprofile` | `dot_zprofile` | zsh ログインシェル設定 |
| `~/.gitconfig` | `dot_gitconfig.tmpl` | git 設定（セットアップ時にメールアドレスを設定） |
| `Brewfile` | `Brewfile` | Homebrew パッケージ・アプリ一覧 |

## 🚀 新しい Mac をセットアップする（1コマンド）

### 前提条件

macOS に Homebrew と chezmoi がインストールされていること。
まだの場合は以下を実行してください：

```bash
# 1. Homebrew をインストール
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. chezmoi をインストール
brew install chezmoi
```

### セットアップ（1コマンド）

```bash
chezmoi init --apply yutoaoki412
```

実行すると以下が順番に行われます：

1. このリポジトリを `~/.local/share/chezmoi` にクローン
2. **「Git email address:」** と聞かれるので git で使うメールアドレスを入力
3. 全ての設定ファイルをホームディレクトリに展開
4. `Brewfile` を元に Homebrew パッケージを自動インストール

---

## 📋 日常の使い方

### 設定ファイルを変更したとき

設定ファイルは **直接 `~/.config/...` や `~/Library/...` を編集**して OK です。
編集後に以下で chezmoi に取り込みます：

```bash
# 変更を chezmoi に反映（全ファイル一括）
chezmoi re-add

# 特定のファイルだけ反映したいとき
chezmoi re-add ~/.zshrc
chezmoi re-add ~/Library/Application\ Support/com.mitchellh.ghostty/config.ghostty
```

変更を確認してからプッシュ：

```bash
chezmoi cd            # dotfiles リポジトリ（~/.local/share/chezmoi）に移動
git diff              # 何が変わったか確認
git add .
git commit -m "feat: update ghostty config"
git push
exit                  # 元のディレクトリに戻る
```

### chezmoi 経由で直接編集する方法（別のやり方）

```bash
# chezmoi のエディタで開いて編集 → apply までやってくれる
chezmoi edit ~/.zshrc
chezmoi apply
```

### 別のマシンで最新の設定を受け取る

```bash
chezmoi update
```

リモートの変更を pull して、ローカルの設定ファイルに自動反映されます。

---

## 🍺 Brewfile（パッケージ）を更新したとき

新しいツールをインストールして Brewfile にも追加したいとき：

```bash
# 現在インストールされているパッケージから Brewfile を再生成
brew bundle dump --force --file=~/.local/share/chezmoi/Brewfile

# コミット
chezmoi cd
git add Brewfile
git commit -m "chore: update Brewfile"
git push
exit
```

> **Tips**: Brewfile が変更されると、次回 `chezmoi apply` / `chezmoi update` 実行時に
> `run_onchange_install-packages.sh.tmpl` が自動的に実行されて `brew bundle install` が走ります。

---

## ⚙️ 仕組みの説明

### メールアドレスの設定の仕組み

chezmoi の **テンプレート機能** を使っています。

初回 `chezmoi init` 時に `~/.config/chezmoi/chezmoi.toml` が自動生成されます
（このファイルはリポジトリに含まれず、そのマシンだけに存在します）：

```toml
[data]
  email = "your@email.com"  # 初回セットアップ時に入力した値
```

`dot_gitconfig.tmpl` はこの値を参照してメールアドレスを設定します：

```toml
[user]
  name = Yuto Aoki
  email = {{ .email }}   ← マシンによって変わる
```

### ファイル名の命名規則（chezmoi のルール）

chezmoi はファイル名のプレフィックスで管理方法を判断します：

| プレフィックス | 意味 |
|--------------|------|
| `dot_` | `~/.` に展開される（例: `dot_zshrc` → `~/.zshrc`） |
| `private_` | パーミッション `600` で展開される（秘密情報を含むファイルに） |
| `.tmpl` サフィックス | テンプレートとして処理される |
| `run_onchange_` | ファイルが変更されたときだけ実行されるスクリプト |

---

## 🔄 chezmoi コマンドまとめ

| コマンド | 何をするか |
|---------|-----------|
| `chezmoi init --apply yutoaoki412` | 新マシンで全設定を1コマンド展開 |
| `chezmoi update` | リモートの最新変更をローカルに反映 |
| `chezmoi re-add` | 実ファイルへの変更を chezmoi に取り込む |
| `chezmoi apply` | chezmoi の状態をホームディレクトリに反映 |
| `chezmoi diff` | chezmoi の状態と実ファイルの差分を表示 |
| `chezmoi edit ~/.zshrc` | chezmoi 経由でファイルを編集 |
| `chezmoi cd` | dotfiles リポジトリに移動 |
| `chezmoi status` | 管理ファイルの変更状況を一覧表示 |

---

## 📁 リポジトリ構成

```
dotfiles/
├── .chezmoi.toml.tmpl              # 初回セットアップ設定（git メールアドレスを入力）
├── Brewfile                        # Homebrew パッケージ一覧
├── run_onchange_install-packages.sh.tmpl  # Brewfile 変更時に自動実行
├── dot_config/
│   └── starship.toml               # → ~/.config/starship.toml
├── dot_gitconfig.tmpl              # → ~/.gitconfig（テンプレート）
├── dot_zprofile                    # → ~/.zprofile
├── dot_zshrc                       # → ~/.zshrc
└── private_Library/
    └── private_Application Support/
        └── com.mitchellh.ghostty/
            └── config.ghostty      # → ~/Library/Application Support/.../config.ghostty
```
