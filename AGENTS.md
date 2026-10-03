# dotfiles の作業規約

公開できる macOS 共通設定を chezmoi で管理する。個人・業務の AI 指示、アカウント、個人用シェル設定は隣接する非公開 machine-config に置く。秘密鍵、トークン、認証ファイル、履歴はどちらにも取り込まない。

## 正本と配置

- 作業リポジトリは `git rev-parse --show-toplevel`、chezmoi の設定元は `chezmoi source-path` で確認する。特定のユーザー名を前提にしない。
- 標準配置は `~/Developer/github/dotfiles` と `~/Developer/github/machine-config`。
- `.chezmoiroot` が `home/` を設定元にする。README、Brewfile、scripts、AI 規約をホームへ展開しない。
- ツールが対応する場合は XDG の設定パスを使う。zsh はホーム、Cursor は macOS の Application Support を使う。
- 公開設定の反映とパッケージの導入は分ける。自動インストール・アップグレード・cleanup を追加しない。

## 変更と確認

1. 関係する source ファイルを編集し、render 後の構文を検査する。
2. `chezmoi diff` で対象と既存のローカル変更を確認する。
3. 依頼された対象だけ `chezmoi apply <対象パス>` で反映する。
4. アプリの実効設定・動作と対象の `chezmoi status` を確認する。調査のみの場合は反映しない。

`./scripts/check` は公開 source の検査、`./scripts/install-packages --check` はパッケージの充足確認。パッケージを導入する依頼では `./scripts/install-packages`、Cursor 拡張には `./scripts/install-cursor-extensions` を使う。

未コミットの変更を消さない。commit・push・外部送信は明示された依頼の範囲だけ行う。詳細は `.agents/skills/dotfiles-workflow/SKILL.md`。
