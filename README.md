# dotfiles

macOSの開発環境設定を[chezmoi](https://www.chezmoi.io)で管理するリポジトリです。
XDG Base Directory仕様に基づき、各種設定を`~/.config`配下に集約しています。私用Macおよび会社用Macの双方で、同一のシェル、Git、ターミナル、エディタ環境を迅速かつ確実に再現します。

対象環境: Apple Silicon Mac（macOS、Homebrew導入環境）

---

## 設計方針と管理対象

本リポジトリは、環境の再現性とホームディレクトリの清潔性を保つため、以下の設計方針を採用しています。

1. **XDG Base Directory仕様への準拠**: ツール固有の設定ファイルを`~/.config`配下に集約し、ホームディレクトリ直下の散乱を防止します。
2. **宣言的なパッケージ管理**: Homebrewの`Brewfile`により、開発CLI、GUIアプリケーション、必要なフォントを一元管理します。
3. **安全なマシン別設定**: メールアドレスなどの個体差をchezmoiのテンプレート機能で動的に注入し、秘密情報の混入を防ぎます。

### 管理対象ファイル一覧

| リポジトリ内のパス | 展開先パス | 説明 |
|---|---|---|
| `dot_config/git/config.tmpl` | `~/.config/git/config` | Gitの共通設定、認証ヘルパー、ユーザー情報テンプレート |
| `dot_config/git/ignore` | `~/.config/git/ignore` | 全リポジトリ共通の大域的除外設定（.envrc、.DS_Store等） |
| `dot_config/ghostty/config` | `~/.config/ghostty/config` | Ghosttyターミナルの外観、フォント、キーバインド、通知設定 |
| `dot_config/starship.toml` | `~/.config/starship.toml` | Starshipプロンプトの設定（Google Cloud等の表示制御） |
| `dot_config/zed/settings.json` | `~/.config/zed/settings.json` | Zedエディタの設定（テーマ、フォント、パネル配置） |
| `dot_config/zed/keymap.json` | `~/.config/zed/keymap.json` | Zedエディタのキーバインド |
| `dot_local/bin/executable_with-direnv` | `~/.local/bin/with-direnv` | Gitワークスペースのdirenv環境でコマンドを実行するラッパー |
| `private_dot_ssh/private_config.tmpl` | `~/.ssh/config` | SSHホスト設定（GitHub個人・会社アカウントの切り替え） |
| `dot_zprofile` | `~/.zprofile` | ログインシェル設定（Homebrewの環境変数、重複のないPATH管理） |
| `dot_zshrc` | `~/.zshrc` | 対話型シェル設定（Starship起動、direnvフック、エイリアス） |
| `Brewfile` | （リポジトリ直下） | Homebrewで管理するパッケージ、Cask、フォント定義 |
| `run_onchange_install-packages.sh.tmpl` | （chezmoiフック） | Brewfile更新時に自動でbrew bundleを実行するスクリプト |
| `.chezmoi.toml.tmpl` | （chezmoi設定） | 初回セットアップ時に必要なパラメータ（Gitメールアドレス等）を対話入力する定義 |

---

## セットアップ手順

新しいMacで環境を構築する際の手順です。

### 1. 前提ツールの導入

Xcode Command Line ToolsとHomebrewをインストールし、続けてchezmoiを導入します。

```bash
# Homebrew のインストール
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# chezmoi のインストール
brew install chezmoi
```

### 2. 環境の初期化と反映

リポジトリを指定して初期化を実行します。初期化の過程で、Gitコミットに使用するメールアドレスの入力を求められます。

```bash
# リポジトリの初期化と設定反映
chezmoi init --apply yutoaoki412
```

適用前に設定の差分を確認したい場合は、以下のように分割して実行できます。

```bash
chezmoi init yutoaoki412
chezmoi diff
chezmoi apply
```

`chezmoi apply`の実行時、`Brewfile`に定義されたCLIツール、GUIアプリケーション（Ghostty、Zed、Cursor、Raycast）、およびJetBrainsMono Nerd Fontが自動的にインストールされます。

### 3. セットアップ完了後の確認

ターミナルを再起動するか、新しいシェルセッションを開始して設定を読み込みます。

```bash
exec zsh -l
```

---

## 日常の運用フロー

### 設定ファイルの変更と取り込み

各ツールの設定ファイル（`~/.config/ghostty/config`や`~/.zshrc`など）は、普段通り直接編集して動作を確認できます。変更内容をリポジトリへ反映する際は、chezmoiのコマンドを使用します。

```bash
# 変更した設定ファイルをリポジトリに取り込む
chezmoi re-add

# または特定のファイルのみを指定して取り込む
chezmoi re-add ~/.config/ghostty/config
chezmoi re-add ~/.zshrc
```

取り込んだ変更を確認し、Gitでコミットとプッシュを行います。

```bash
chezmoi cd
git diff
git commit -am "feat: update configuration"
git push
exit
```

### 別の端末での変更適用

リモートリポジトリの最新設定をローカルマシンに取得して適用します。

```bash
chezmoi update
```

### Homebrewパッケージの更新

新しいパッケージを追加、あるいは不要なパッケージを削除した後は、`Brewfile`を更新して変更をコミットします。

```bash
# 現在のインストール状況をもとに Brewfile をダンプ
brew bundle dump --force --file="$(chezmoi source-path)/Brewfile"

chezmoi cd
git add Brewfile
git commit -m "chore: update Brewfile"
git push
exit
```

`Brewfile`の内容に変更があると、次回の`chezmoi apply`または`chezmoi update`の実行時に自動でパッケージの同期スクリプトが動作します。

---

## リポジトリ構成

```
dotfiles/
├── .chezmoi.toml.tmpl                     # 初期化用テンプレート
├── .envrc                                 # ローカル環境変数定義
├── Brewfile                               # パッケージ・Cask・フォントマニフェスト
├── run_onchange_install-packages.sh.tmpl   # パッケージ自動インストールフック
├── dot_config/
│   ├── ghostty/
│   │   └── config                         # Ghostty設定
│   ├── git/
│   │   ├── config.tmpl                    # Git設定テンプレート
│   │   └── ignore                         # 大域的gitignore
│   ├── starship.toml                      # Starshipプロンプト設定
│   └── zed/
│       ├── keymap.json                    # Zedキーバインド
│       └── settings.json                  # Zed設定
├── dot_local/
│   └── bin/
│       └── executable_with-direnv         # direnvラッパースクリプト
├── private_dot_ssh/
│   └── private_config.tmpl                # SSHホスト設定テンプレート
├── dot_zprofile                           # ログインシェル設定
├── dot_zshrc                              # 対話型シェル設定
└── README.md                              # 本ドキュメント
```
