# AGENTS.md

本ドキュメントは、このdotfilesリポジトリ（[yutoaoki412/dotfiles](https://github.com/yutoaoki412/dotfiles)）で作業するすべてのAIエージェント（OpenAI Codex、Claude Code、Cursor、Gemini/Antigravity等）に対する技術的コンテキスト、制約事項、および運用規定です。

---

## 1. リポジトリの前提とアーキテクチャ

- **管理方式**: [chezmoi](https://www.chezmoi.io) を用いた宣言的な設定管理。
- **配置規約**: XDG Base Directory仕様に完全準拠し、設定ファイルはホームディレクトリ直下ではなく `~/.config`（リポジトリ内では `dot_config/`）に配置します。
- **対象環境**: Apple Silicon macOS（Homebrew導入環境）。
- **SSOT（信頼できる単一の情報源）**:
  - 作業ディレクトリは `/Users/aokiyuto/Developer/dotfiles` です。
  - chezmoiの `sourceDir` はこのディレクトリに固定されています（`~/.local/share/chezmoi` は本リポジトリへのシンボリックリンクです）。
  - すべてのファイル作成・編集・Git操作は必ず `/Users/aokiyuto/Developer/dotfiles` 内で実行してください。

---

## 2. 必須の安全規範と制約事項

- **秘密情報の混入禁止**: APIキー、個人トークン、SSH秘密鍵、クラウド認証情報をリポジトリ内に配置してはいけません。
- **ホームディレクトリへの汚染防止**:
  - リポジトリ管理用ファイル（`README.md`、`Brewfile`、`AGENTS.md`、`.envrc` 等）がホームディレクトリ直下に展開されないよう、`.chezmoiignore` を必ず維持してください。
- **パスのポータビリティ**:
  - 設定ファイル内に `/Users/aokiyuto` 等の環境依存パスをハードコードしてはいけません。`$HOME`、`~`、またはchezmoiテンプレート変数（`{{ .chezmoi.homeDir }}`）を使用してください。
- **タスク完了基準（定義済みのDone）**:
  - 変更作業の完了時は、必ず `chezmoi status` および `chezmoi diff` を実行し、リポジトリとローカル実環境の間に予期しない差分が残っていないことを検証してください。

---

## 3. 主要ファイルと配置対応

| リポジトリ内のパス | 展開先パス | 役割 |
|---|---|---|
| `dot_config/git/config.tmpl` | `~/.config/git/config` | Git設定（メールアドレスはchezmoiデータから注入） |
| `dot_config/git/ignore` | `~/.config/git/ignore` | 大域的gitignore（Git標準により自動認識） |
| `dot_config/ghostty/config` | `~/.config/ghostty/config` | Ghosttyターミナル設定（XDG標準パス） |
| `dot_config/starship.toml` | `~/.config/starship.toml` | Starshipプロンプト設定 |
| `dot_config/zed/settings.json` | `~/.config/zed/settings.json` | Zedエディタ設定 |
| `dot_config/zed/keymap.json` | `~/.config/zed/keymap.json` | Zedキーバインド |
| `dot_local/bin/executable_with-direnv` | `~/.local/bin/with-direnv` | ワークスペースdirenvラッパー |
| `private_dot_ssh/private_config.tmpl` | `~/.ssh/config` | SSH設定（GitHubアカウント分岐） |
| `dot_zprofile` | `~/.zprofile` | ログインシェル（Homebrew環境、重複排除PATH） |
| `dot_zshrc` | `~/.zshrc` | 対話シェル（Starship/direnvフック、エイリアス） |
| `Brewfile` | （リポジトリ直下） | CLIツール、GUI Cask、フォント定義 |
| `.chezmoiignore` | （リポジトリ直下） | ホーム直下への誤展開を防止する除外リスト |

---

## 4. 標準コマンド

### 状態確認と差分検証
```bash
# chezmoi の管理状態と差分を確認
chezmoi status
chezmoi diff

# Homebrew パッケージの過不足を確認
brew bundle check --verbose --file=Brewfile
```

### 反映と同期
```bash
# リポジトリの変更をホームディレクトリに反映
chezmoi apply

# 実環境で直接編集した内容をリポジトリへ取り込む
chezmoi re-add <対象の実ファイルパス>
```

### 各種ツールの構文検証
```bash
# Ghostty 設定のバリデーション
ghostty +validate-config

# Git 大域的除外の動作確認
git check-ignore -v <確認対象ファイル>
```

---

## 5. 多段手順の参照先
設定の追加・更新・Brewfileの再同期などの詳細な多段ワークフローは、`.agents/skills/dotfiles-workflow/SKILL.md` を参照してください。
