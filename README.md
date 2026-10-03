# macOS dotfiles

Apple Silicon macOS の共通設定を chezmoi で再現する。個人・業務の AI 指示やアカウント設定は非公開の machine-config に分け、認証情報と履歴は実機で管理する。

```text
📁 ~/Developer/github/
├── 📁 dotfiles/                 # 公開: 共通設定とパッケージ宣言
│   ├── 📄 .chezmoiroot          # home/ を設定元にする
│   ├── 📁 home/                # chezmoi の展開対象
│   ├── 📄 Brewfile
│   └── 📁 scripts/             # 検査と明示インストール
├── 📁 machine-config/           # 非公開: 個人設定と共通 AI 指示
├── 📁 i3-data-infra/
├── 📁 i3-data-model/
└── 📁 その他のリポジトリ/
```

## 配置先

| 設定 | 展開先 |
|---|---|
| Git | `~/.config/git/config` と `ignore` |
| Ghostty | `~/.config/ghostty/config.ghostty` |
| Zed・Starship | `~/.config/zed/`、`~/.config/starship.toml` |
| Cursor | `~/Library/Application Support/Cursor/User/settings.json` |
| zsh | `~/.zshenv`、`~/.zprofile`、`~/.zshrc` |
| direnv ラッパー | `~/.local/bin/with-direnv` |

Git identity は machine-config の `~/.config/git/config.local`。個人シェル設定は `~/.config/zsh/local.zshenv` と `local.zsh`。SSH 設定・共通 AI 指示も machine-config が管理する。秘密鍵・Keychain・AWS/GCP/GitHub の認証はコピーしない。GitHub の Git 操作は SSH、API 操作は gh を使う。

## 初回導入と更新

Homebrew を公式手順で導入し、`chezmoi` とこのリポジトリを取得する。隣接する machine-config の README に従って個人設定を配置した後、設定とパッケージを明示反映する。

```sh
cd ~/Developer/github/dotfiles
./scripts/install-packages --check
./scripts/install-packages
./scripts/install-cursor-extensions --check
./scripts/install-cursor-extensions
chezmoi init --source "$PWD"
./scripts/check
chezmoi diff
chezmoi apply
```

通常は変更した対象パスだけ diff・apply する。chezmoi apply でパッケージは導入されない。installer は既存 native GUI を保持し、明示 upgrade と自動 cleanup を抑える。不足依存の導入に伴う依存更新は Homebrew が行う。

プロジェクトのツール版は各 repo の mise.toml が優先。`.zshenv` で shims を選択し、対話シェルは mise activate と direnv hook を使う。`.envrc` の実行許可は内容確認後に各 repo で `direnv allow`。移動すると許可を取り直す場合がある。

GCP の既定プロジェクトは、各 repo のローカル `.envrc` に `export CLOUDSDK_CORE_PROJECT=<project-id>` を指定する。dotfiles へのプロジェクト登録は不要。`.envrc` は Git 管理対象外なので、別の端末では作成し直す。作成・変更後は内容を確認して `direnv allow` を実行し、`direnv exec . printenv CLOUDSDK_CORE_PROJECT` で読み込みを確認する。シェルの現在の環境には、次のプロンプト表示時に反映される。

`CLOUDSDK_CORE_PROJECT` はプロジェクト選択だけを行い、GCP の認証は別途必要。Starship の GCP 表示は `CLOUDSDK_ACTIVE_CONFIG_NAME` を条件にしているため、プロジェクトIDだけの指定では表示されない。

Ghostty の設定再読込は Ctrl+Shift+R。Quick Terminal の位置変更は完全再起動が必要で、グローバル Ctrl+M は macOS のアクセシビリティ許可に依存する。開始先は `~/Developer/github`、既存ウィンドウ・タブ・split では作業ディレクトリを引き継ぐ。

Zed 拡張は auto_install_extensions、Cursor 拡張は scripts/cursor-extensions.txt を正本にする。Starship の GCP 表示はプロジェクトで CLOUDSDK_ACTIVE_CONFIG_NAME が選ばれたときだけ表示する。

## 検査

`./scripts/check` は source の構文とシェル起動・導入処理の境界を検査する。反映後は `chezmoi status` と各アプリの実効設定を確認する。新規ファイルを含めた公開範囲を精査し、秘密情報を含めない。認証・メモリ・セッション・キャッシュは管理しない。
