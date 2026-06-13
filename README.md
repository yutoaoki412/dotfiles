# dotfiles

macOS の開発環境設定を [chezmoi](https://www.chezmoi.io) で管理するリポジトリです。
私用 Mac・会社用 Mac のどちらでも、同じシェル・Git・ターミナル環境を再現することを目的としています。

> 対象環境: Apple Silicon Mac（Homebrew が `/opt/homebrew` にインストールされている前提）

## 目次

- [このリポジトリが管理しているもの](#このリポジトリが管理しているもの)
- [セットアップ（新しい Mac で初めて使う）](#セットアップ新しい-mac-で初めて使う)
- [日常の使い方](#日常の使い方)
- [Brewfile を更新したとき](#brewfile-を更新したとき)
- [仕組みの解説](#仕組みの解説)
- [コマンドリファレンス](#コマンドリファレンス)
- [リポジトリ構成](#リポジトリ構成)

---

## このリポジトリが管理しているもの

| 実際のパス | リポジトリ内のファイル | 内容 |
|-----------|----------------------|------|
| `~/.zshrc` | `dot_zshrc` | zsh の対話シェル設定（PATH への追加、starship の起動） |
| `~/.zprofile` | `dot_zprofile` | ログインシェル設定（Homebrew の環境変数、PATH への追加） |
| `~/.gitconfig` | `dot_gitconfig.tmpl` | Git のユーザー名・メールアドレス、`gh` CLI を使った GitHub 認証設定 |
| `~/.config/starship.toml` | `dot_config/starship.toml` | [starship](https://starship.rs) プロンプトの表示設定 |
| `~/Library/Application Support/com.mitchellh.ghostty/config.ghostty` | `private_Library/.../config.ghostty` | [Ghostty](https://ghostty.org) ターミナルの設定（外観・キーバインドなど） |
| （リポジトリ直下） | `Brewfile` | Homebrew でインストールするパッケージ・Cask・エディタ拡張・Go/npm ツールの一覧 |
| （初回セットアップ時のみ使用） | `.chezmoi.toml.tmpl` | 初回 `chezmoi init` 時に Git のメールアドレスを質問するテンプレート |
| （Brewfile 変更時に自動実行） | `run_onchange_install-packages.sh.tmpl` | `Brewfile` が変更されたときだけ `brew bundle install` を実行するスクリプト |

### Brewfile に含まれているもの

`Brewfile` には以下のようなものがまとめて記載されています（詳細は [Brewfile](Brewfile) を参照）。

- **CLI ツール**: `awscli`, `chezmoi`, `dive`, `gh`, `go`, `node`, `ripgrep`, `starship` など
- **Cask（GUI アプリ）**: `gcloud-cli`
- **エディタ拡張**（VS Code 系）: ESLint, Prettier, Go, Terraform, Python 関連など多数
- **Go ツール**: `dlv`, `goimports`, `gopls`, `impl`
- **npm パッケージ**: `@github/copilot`

---

## セットアップ（新しい Mac で初めて使う）

### 前提条件

macOS に Homebrew と chezmoi がインストールされていること。まだの場合は以下を実行します。

```bash
# 1. Homebrew をインストール
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. chezmoi をインストール
brew install chezmoi
```

### 手順（推奨: 内容を確認しながら進める）

`chezmoi apply` は **既存の `~/.zshrc` や `~/.gitconfig` などをこのリポジトリの内容で上書きします。**
初めて実行する場合や、すでに何か設定が入っている Mac では、以下のように差分を確認しながら進めるのがおすすめです。

```bash
# 1. リポジトリを取得するだけ（ホームディレクトリはまだ変更されない）
chezmoi init yutoaoki412

# 2. 何が変更されるか確認する
chezmoi diff

# 3. 問題なければ反映する
chezmoi apply
```

`chezmoi init` の実行中に **「Git email address:」** と聞かれるので、Git のコミットに使うメールアドレスを入力してください（[仕組みの解説](#仕組みの解説)を参照）。

### 1 コマンドで一気にセットアップする場合

内容を把握済みで、確認なしで一気に反映してよい場合は次の 1 コマンドでも実行できます。

```bash
chezmoi init --apply yutoaoki412
```

### セットアップ中に何が起こるか

1. このリポジトリが `~/.local/share/chezmoi` にクローンされる
2. 「Git email address:」の入力を求められる
3. `dot_*` で始まるファイルがホームディレクトリ配下に展開される（既存ファイルは上書き）
4. `Brewfile` に基づいて `brew bundle install` が自動実行され、CLI ツール・Cask・エディタ拡張・Go/npm パッケージがインストールされる

> **補足**: 手順 4 は新規にインストールするパッケージの数によっては数分〜十数分かかることがあります。

---

## 日常の使い方

### 設定ファイルを変更したとき

設定ファイルは **直接 `~/.config/...` や `~/Library/...` を編集**して OK です。
編集後に以下で chezmoi に取り込みます。

```bash
# 変更を chezmoi に反映（全ファイル一括）
chezmoi re-add

# 特定のファイルだけ反映したいとき
chezmoi re-add ~/.zshrc
chezmoi re-add ~/Library/Application\ Support/com.mitchellh.ghostty/config.ghostty
```

変更を確認してからプッシュします。

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

## Brewfile を更新したとき

新しいツールをインストールして `Brewfile` にも追加したいときは、現在インストールされているパッケージから再生成できます。

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

> **Tips**: `Brewfile` が変更されると、次回 `chezmoi apply` / `chezmoi update` 実行時に
> `run_onchange_install-packages.sh.tmpl` が自動的に実行されて `brew bundle install` が走ります。

---

## 仕組みの解説

### メールアドレスの設定の仕組み

chezmoi の **テンプレート機能** を使っています。

初回 `chezmoi init` 時に `~/.config/chezmoi/chezmoi.toml` が自動生成されます
（このファイルはリポジトリに含まれず、そのマシンだけに存在します）。

```toml
[data]
  email = "your@email.com"  # 初回セットアップ時に入力した値
```

`dot_gitconfig.tmpl` はこの値を参照してメールアドレスを設定します。

```toml
[user]
  name = Yuto Aoki
  email = {{ .email }}   # マシンによって変わる
```

### ファイル名の命名規則（chezmoi のルール）

chezmoi はファイル名のプレフィックス・サフィックスで管理方法を判断します。

| プレフィックス / サフィックス | 意味 |
|--------------------------|------|
| `dot_` | `~/.` に展開される（例: `dot_zshrc` → `~/.zshrc`） |
| `private_` | パーミッション `600` で展開される（秘密情報を含むファイルに） |
| `.tmpl` サフィックス | テンプレートとして処理される（`{{ }}` の中身が値に置き換わる） |
| `run_onchange_` | ファイルの内容が変更されたときだけ実行されるスクリプト |

---

## コマンドリファレンス

| コマンド | 何をするか |
|---------|-----------|
| `chezmoi init yutoaoki412` | リポジトリを取得するだけ（ホームディレクトリはまだ変更しない） |
| `chezmoi init --apply yutoaoki412` | 新しい Mac で全設定を 1 コマンドで展開する |
| `chezmoi diff` | chezmoi の状態と実ファイルの差分を表示する |
| `chezmoi apply` | chezmoi の状態をホームディレクトリに反映する |
| `chezmoi re-add` | 実ファイルへの変更を chezmoi に取り込む |
| `chezmoi edit ~/.zshrc` | chezmoi 経由でファイルを編集する |
| `chezmoi cd` | dotfiles リポジトリ（`~/.local/share/chezmoi`）に移動する |
| `chezmoi status` | 管理ファイルの変更状況を一覧表示する |
| `chezmoi update` | リモートの最新変更を pull してローカルに反映する |

---

## リポジトリ構成

```
dotfiles/
├── .chezmoi.toml.tmpl                     # 初回セットアップ時に Git メールアドレスを入力させるテンプレート
├── Brewfile                                # Homebrew パッケージ・Cask・拡張機能・ツール一覧
├── run_onchange_install-packages.sh.tmpl   # Brewfile 変更時に自動実行される brew bundle install
├── dot_config/
│   └── starship.toml                       # → ~/.config/starship.toml
├── dot_gitconfig.tmpl                      # → ~/.gitconfig（テンプレート）
├── dot_zprofile                            # → ~/.zprofile
├── dot_zshrc                               # → ~/.zshrc
└── private_Library/
    └── private_Application Support/
        └── com.mitchellh.ghostty/
            └── config.ghostty               # → ~/Library/Application Support/com.mitchellh.ghostty/config.ghostty
```
