# 設定の反映とパッケージ導入は分ける。導入は scripts/install-packages。
# プロジェクト指定のバージョンは各リポジトリの mise.toml で管理する。
# Node.js・Terraform は mise、Python は uv で導入する（`uv python install --default`）。

# 基盤 CLI
brew "chezmoi"
brew "gh"
brew "direnv"
brew "mise"
brew "starship"
brew "ripgrep"
brew "jq"
brew "coreutils"
brew "dive"
brew "awscli"
cask "gcloud-cli"
cask "codex"

# ランタイム（Node.js・Terraform は mise、Python は uv が管理する）
brew "go"
brew "bun"

# 既存のアプリを上書きしない導入は scripts/install-packages を使う。
cask "ghostty"
cask "zed"
cask "cursor"
cask "raycast"
cask "font-jetbrains-mono-nerd-font"

# 実機で使用中の Go ツール。既存ツールを更新せず、不足分だけを導入する。
go "github.com/go-delve/delve/cmd/dlv"
go "golang.org/x/tools/cmd/goimports"
go "golang.org/x/tools/gopls"
go "github.com/josharian/impl"

# Cursor 拡張は scripts/cursor-extensions.txt と install-cursor-extensions で管理。
# brew の vscode DSL は code を優先するため使用しない。
