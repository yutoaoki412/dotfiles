---
name: dotfiles-workflow
description: 公開dotfilesの設定変更、反映、パッケージ管理と実効性確認を行う。
---

# dotfiles の変更手順

公開 source は `home/`、非公開の個人・AI設定は隣接する machine-config が正本。対象リポジトリと chezmoi source を確認してから作業する。

## 設定の更新

依頼が調査だけなら書き換えない。変更では source を編集し、テンプレートは `chezmoi execute-template` 等で render してから検査する。Ghostty は source の `--config-file` を指定した `+validate-config`、JSON/JSONC/TOML と zsh は `scripts/check` で確認する。まだ適用していない source の検証に live 設定を代用しない。

`chezmoi diff <対象パス>` で、意図した変更と既存のローカル変更を確認する。許可された対象を `chezmoi apply <対象パス>` で反映し、実効設定・必要な動作と対象の status を確認する。他の設定の差分を消すために全体を再適用しない。

## パッケージの更新

Brewfile は意図した依存関係の宣言。dump は一時ファイルへ出し、必要な差分だけ取り込む。`--force` で宣言を全面置換しない。`scripts/install-packages --check` を確認し、導入が依頼された場合だけ明示 installer を実行する。インストール済みのネイティブ GUI とプロジェクトの mise 指定を尊重する。不要と確認できていないパッケージを一括削除しない。

## 完了と commit

関係する構文、反映、実効動作が確認できれば完了。調査に全体diffの解消を要求しない。`git diff --check`、新規ファイルを含む公開範囲の検査を行い、確認結果と残る制約を報告する。commit と push は明示依頼された場合だけ行う。
