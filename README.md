# dotfiles

Apple Silicon Mac のシェル、Git、SSH、ターミナル、エディタ設定を [chezmoi](https://www.chezmoi.io/) で管理します。仕事と私用で共通に使う設定だけを置きます。

## 管理対象

| ソース | 配置先 |
| --- | --- |
| `dot_zprofile`, `dot_zshrc` | `~/.zprofile`, `~/.zshrc` |
| `dot_gitconfig.tmpl`, `dot_gitignore_global` | `~/.gitconfig`, `~/.gitignore_global` |
| `private_dot_ssh/private_config.tmpl` | `~/.ssh/config` |
| `dot_local/bin/executable_with-direnv` | `~/.local/bin/with-direnv` |
| `dot_config/starship.toml` | `~/.config/starship.toml` |
| `dot_config/zed/keymap.json`, `dot_config/zed/private_settings.json` | `~/.config/zed/keymap.json`, `~/.config/zed/settings.json`（権限 `600`） |
| `private_Library/` | `~/Library/` 内の Ghostty 設定 |

`Brewfile` は明示的にインストールするツールと、設定で使う Ghostty・Zed・フォントの一覧です。chezmoi の `apply` では実行しません。既に手動でインストールしたアプリがある端末では、Homebrew による再インストール前に状態を確認してください。`private_` は配置先の権限を制限する chezmoi の名前で、**Git 上の暗号化を意味しません**。SSH 秘密鍵、認証情報、会社固有の設定、個人用の Codex 指示はこのリポジトリに置きません。

## 新しい Mac で使う

1. [Homebrew](https://brew.sh/) をインストールし、`brew install chezmoi` を実行する。
2. 端末ごとの SSH 鍵を端末内で作成し、GitHub に登録する。clone 前に `~/.ssh/config` で `github-yutoaoki412` を `github.com` に接続するエイリアスとして定義し、`~/.ssh/id_ed25519_yutoaoki412` を指定する。秘密鍵は Git に追加しない。
3. 以下を実行する。`chezmoi init` で Git コミット用メールアドレスを入力する。

```sh
chezmoi init git@github-yutoaoki412:yutoaoki412/dotfiles.git
brew bundle install --no-upgrade --file="$(chezmoi source-path)/Brewfile"
chezmoi diff
chezmoi apply
```

`chezmoi apply` は既存のホームディレクトリの設定を変更します。差分を確認してから実行してください。認証情報と仕事用の指示は各端末の非公開領域で別途設定します。

## 更新する

chezmoi のソースは `chezmoi source-path` で確認できます。通常の設定ファイルは配置先で編集後、対象を指定して `chezmoi re-add ~/.zshrc` のように取り込みます。`.tmpl` ファイルは配置先から `re-add` せず、ソースのテンプレートを直接編集します。

```sh
chezmoi diff
chezmoi status
git -C "$(chezmoi source-path)" status --short
git -C "$(chezmoi source-path)" diff --check
```

差分に秘密情報や端末固有の値がないことと、`git config --show-origin user.email` で公開用のアドレスが使われることを確認し、追加するファイルを個別に指定してください。一括の `chezmoi re-add`、`git add .`、インストール済みパッケージの無条件な `brew bundle dump --force` は使いません。`Brewfile` は必要な項目だけ編集し、`brew bundle check --file="$(chezmoi source-path)/Brewfile"` で現在の端末との差を確認します。

別の端末では、リモートの更新内容を確認してから `chezmoi update` を実行します。

## ローカル環境と認証

`.envrc` と `.direnv/` は全リポジトリ共通の Git 除外設定に入っています。`.envrc` には秘密情報を置かず、必要な認証情報は OS の資格情報ストアなど Git 外で管理します。プロジェクトで `.envrc` の共有が必要な場合は、そのプロジェクトの方針を確認してください。

対話シェルでは `direnv` のフックが有効です。非対話コマンドでは `with-direnv <command>` を使うと、現在の Git 作業ツリーのルートで `direnv exec` を実行できます。

個人用 GitHub リポジトリの Git remote には SSH エイリアス `github-yutoaoki412` を使用します。`gh` の API 認証は各端末で別途設定します。

`.gitignore` はソースへの誤追加を防ぎ、`.chezmoiignore` はホームディレクトリへ配置しないファイルを指定します。これらは既に Git 履歴に入った内容を消す仕組みではありません。秘密情報を誤って追加した場合は、まず認証情報を失効・更新し、履歴の対処を個別に判断します。
