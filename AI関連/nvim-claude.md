## CLI上でNvimとClaude Codeを組み合わせて使うためのガイド

### 情報源

本ドキュメントは以下の公式・準公式ソースに基づいて作成されています。

- GitHub `anthropics/claude-code` 公式リポジトリ（README, CHANGELOG, plugins ディレクトリ）
- GitHub `coder/claudecode.nvim` リポジトリ（Neovim IDE統合プラグイン）
- GitHub Issue #1234「Support for other IDEs (Neovim/Emacs)」
- Google AI概要から取得したCLI Reference情報
- Anthropic Engineering Blog の各記事

---

## 1. インストール方法

公式READMEにて、npmによるインストールは非推奨（deprecated）となっている。現在の推奨方法は以下の通り。

### macOS/Linux（推奨）

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### Homebrew（macOS/Linux）

```bash
brew install --cask claude-code
```

### npm（非推奨）

```bash
npm install -g @anthropic-ai/claude-code
```

インストール後、プロジェクトディレクトリで `claude` と入力してセッションを開始する。インストール状態は `claude doctor` で確認できる。

---

## 2. Core Slash Commands（セッション内コマンド）

セッション内で `/` を入力すると全コマンド一覧が表示される。

| コマンド           | 説明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `/help`            | Built-in Commands Referenceの全表示。カスタムSkillを含む     |
| `/compact`         | 会話履歴を要約してコンテキストウィンドウの空きを作る。使用率70%超で推奨 |
| `/clear`           | 会話履歴の完全クリア。エイリアスとして `/reset`, `/new` もあり。使用率85%超で推奨 |
| `/model`           | AIモデルの切り替え（セッション再起動不要）                   |
| `/config`          | ツール権限、モデル選択などの設定メニュー                     |
| `/rewind`          | ファイルチェックポイントにロールバック。大きな編集の前に使える安全装置 |
| `/undo`            | `/rewind` のエイリアス                                       |
| `/cost`            | 現在のセッションのトークンコスト確認                         |
| `/bug`             | バグ報告の送信                                               |
| `/doctor`          | 設定の診断。MCPサーバーの設定不整合なども検出                |
| `/resume`          | 直前のセッションディレクトリのセッション一覧表示。`Ctrl+A`で全プロジェクト表示 |
| `/plugin`          | プラグインのインストール・管理。Installedタブで`f`キーでお気に入り登録 |
| `/skills`          | 利用可能なスキルの一覧表示                                   |
| `/tui`             | フリッカーフリーなTUI（テキストUI）レンダリングモードへ切替  |
| `/focus`           | ノーマルとverboseトランスクリプトの切替                      |
| `/context`         | コンテキスト管理                                             |
| `/init`            | 初期化（Built-inスラッシュコマンドとしてSkillツールから呼出可能） |
| `/review`          | コードレビュー                                               |
| `/security-review` | セキュリティレビュー                                         |
| `/team-onboarding` | ローカルのClaude Code使用状況からチーム向けランプアップガイドを生成 |
| `/proactive`       | `/loop` のエイリアス                                         |
| `/recap`           | セッション復帰時にコンテキストの要約を提供。`/config` で設定可能 |

---

## 3. CLIフラグ（起動時オプション）

| フラグ                            | 説明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| `claude`                          | 対話セッションを開始                                         |
| `claude "質問"`                   | ワンショット質問                                             |
| `claude -p "プロンプト"`          | ヘッドレスモード（パイプ対応、非対話的に実行して結果を出力） |
| `claude -c` / `claude --continue` | 直前のセッションを再開                                       |
| `claude --resume`                 | セッションを選択して再開                                     |
| `claude --resume <session-id>`    | 指定セッションを再開。`--name` でカスタム名も保持            |
| `claude --model <model>`          | モデル指定                                                   |
| `claude --verbose`                | 詳細出力                                                     |
| `claude --ide`                    | IDE統合モードで起動                                          |
| `claude --disable-slash-commands` | 全スキル・コマンドを無効化                                   |
| `claude doctor`                   | インストール状態の診断                                       |
| `claude migrate-installer`        | グローバルnpmインストールからローカルインストール（`~/.claude/local/`）への移行 |
| `claude install`                  | ネイティブバイナリのインストール（Alpha）                    |
| `claude auto-mode defaults`       | Auto Modeのデフォルト分類ルール表示                          |

---

## 4. キーボードショートカット（セッション内）

| ショートカット  | 説明                                                         |
| --------------- | ------------------------------------------------------------ |
| `Shift+Tab`     | 入力モードの切替（複数行入力等）                             |
| `Ctrl+R`        | プロンプト履歴の検索                                         |
| `Ctrl+C`        | 現在の応答を中断                                             |
| `Ctrl+G`        | Claudeの最後の応答を外部エディタでコメント付きコンテキストとして表示 |
| `Ctrl+O`        | verbose表示のトグル                                          |
| `Ctrl+A`        | `/resume` 内で全プロジェクトのセッション表示                 |
| `Escape`        | 入力のキャンセル                                             |
| `@ファイルパス` | ファイル内容をプロンプトのコンテキストに含める               |

---

## 5. NvimとClaude Codeの統合

GitHub Issue #1234の議論から、以下の3パターンが確認されている。

### パターンA：Nvimターミナルスプリットで直接利用（最もシンプル）

```vim
:term claude
```

Nvimのビルトインターミナルでclaude codeを起動する。MCP接続なしのシンプルな方法で、すぐに始められる。

ターミナルモードの切り替えが最重要の基本操作となる。

- `i` でインサートモード（ターミナルへの入力）
- `<C-\><C-n>` でノーマルモード（Nvimの操作に戻る）

### パターンB：`coder/claudecode.nvim` プラグイン（IDE統合）

VS Code拡張と同等のWebSocket MCP統合を実現するNeovimプラグイン。Anthropic公式ではなくコミュニティ製だが、公式Issue内でも広く参照されている最も成熟したプラグインである。

#### 主な機能

- Claude Codeがリアルタイムで現在のファイルや選択テキストを認識
- Claudeが提案する変更のネイティブdiff表示
- ファイルを開く、診断情報の取得などのエディタ操作をClaudeが実行
- ビジュアル選択のコンテキスト送信

#### インストール（lazy.nvim）

```lua
{
  "coder/claudecode.nvim",
  dependencies = { "folke/snacks.nvim" },
  config = true,
  keys = {
    { "<leader>a",  nil,                               desc = "AI/Claude Code" },
    { "<leader>ac", "<cmd>ClaudeCode<cr>",              desc = "Toggle Claude" },
    { "<leader>af", "<cmd>ClaudeCodeFocus<cr>",         desc = "Focus Claude" },
    { "<leader>ar", "<cmd>ClaudeCode --resume<cr>",     desc = "Resume Claude" },
    { "<leader>aC", "<cmd>ClaudeCode --continue<cr>",   desc = "Continue Claude" },
    { "<leader>am", "<cmd>ClaudeCodeSelectModel<cr>",   desc = "Select Claude model" },
    { "<leader>ab", "<cmd>ClaudeCodeAdd %<cr>",         desc = "Add current buffer" },
    { "<leader>as", "<cmd>ClaudeCodeSend<cr>", mode = "v", desc = "Send to Claude" },
    { "<leader>as", "<cmd>ClaudeCodeTreeAdd<cr>",       desc = "Add file",
      ft = { "NvimTree", "neo-tree", "oil", "minifiles", "netrw" } },
    { "<leader>aa", "<cmd>ClaudeCodeDiffAccept<cr>",    desc = "Accept diff" },
    { "<leader>ad", "<cmd>ClaudeCodeDiffDeny<cr>",      desc = "Deny diff" },
  },
}
```

#### 必要要件

- Neovim >= 0.8.0
- Claude Code CLIインストール済み
- `folke/snacks.nvim`

#### 主なコマンド

| コマンド                              | 説明                                         |
| ------------------------------------- | -------------------------------------------- |
| `:ClaudeCode`                         | Claudeターミナルのトグル                     |
| `:ClaudeCodeFocus`                    | スマートフォーカス/トグル                    |
| `:ClaudeCodeSend`                     | ビジュアル選択をClaudeに送信                 |
| `:ClaudeCodeAdd <file> [start] [end]` | ファイル（行範囲指定可）をコンテキストに追加 |
| `:ClaudeCodeSelectModel`              | モデル選択                                   |
| `:ClaudeCodeDiffAccept`               | diff変更の承認                               |
| `:ClaudeCodeDiffDeny`                 | diff変更の拒否                               |

#### 仕組み

プラグインがWebSocketサーバーを起動し、`~/.claude/ide/<port>.lock` にロックファイルを書き込む。Claude Code CLIがこのロックファイルを検出して自動接続し、MCPプロトコル経由でエディタの操作権限を取得する。VS Code拡張と同じプロトコルを使用している。

#### ローカルインストールの場合の設定

`claude migrate-installer` でローカルインストールに移行した場合、パスを明示する必要がある。

```lua
opts = {
  terminal_cmd = "~/.claude/local/claude",
}
```

#### Diffの操作

Claudeが変更を提案すると、ネイティブのNvim diffビューが開く。

- 承認: `:w`（保存）または `<leader>aa`
- 拒否: `:q` または `<leader>ad`
- Claudeの提案内容を承認前に編集することも可能

### パターンC：MCP統合（上級）

MCP対応のNeovimプラグインを使って、Claude CodeのMCPサーバーに接続する方法。

```bash
CLAUDE_CODE_SSE_PORT=3000 claude
```

環境変数でポートを指定してClaude Codeを起動し、MCP対応プラグインからそのポートに接続する。

---

## 6. ターミナルプロバイダーの選択肢（claudecode.nvim）

`claudecode.nvim` は複数のターミナルプロバイダーに対応している。

### `"auto"`（デフォルト）

snacks.nvimが利用可能ならそちらを使用、なければネイティブ。

### `"snacks"`

snacks.nvimのターミナル機能を使用。フローティングウィンドウの設定が可能。

### `"native"`

Neovimのビルトインターミナル。

### `"external"`

Neovim外の別ターミナルアプリで起動。tmuxやAlacrittyなどと併用する場合に有効。

```lua
opts = {
  terminal = {
    provider = "external",
    provider_opts = {
      external_terminal_cmd = "alacritty -e %s",
    },
  },
}
```

### `"none"`

ターミナルUI管理なし。WebSocketサーバーとツールだけ起動し、Claude CLIは外部（tmux等）で自分で管理する。`claude --ide` または `/ide` コマンドで接続。

---

## 7. フローティングウィンドウの設定例

snacks.nvimを使ったフローティングウィンドウ構成。`Ctrl+,` でトグルできる設定例。

```lua
local toggle_key = "<C-,>"
{
  "coder/claudecode.nvim",
  dependencies = { "folke/snacks.nvim" },
  keys = {
    { toggle_key, "<cmd>ClaudeCodeFocus<cr>", desc = "Claude Code", mode = { "n", "x" } },
  },
  opts = {
    terminal = {
      snacks_win_opts = {
        position = "float",
        width = 0.9,
        height = 0.9,
        keys = {
          claude_hide = {
            toggle_key,
            function(self) self:hide() end,
            mode = "t",
            desc = "Hide",
          },
        },
      },
    },
  },
}
```

### ウィンドウ位置のバリエーション

```lua
-- 下部ドロワー
snacks_win_opts = {
  position = "bottom",
  height = 0.4,
  width = 1.0,
  border = "single",
}

-- 右側パネル
snacks_win_opts = {
  position = "right",
  width = 0.4,
  height = 1.0,
  border = "rounded",
}

-- 小さな中央ポップアップ
snacks_win_opts = {
  position = "float",
  width = 120,   -- 固定幅（カラム数）
  height = 30,   -- 固定高（行数）
  border = "double",
  backdrop = 90,
}
```

---

## 8. ワーキングディレクトリの制御（claudecode.nvim）

`autochdir` やバッファローカルのcwd変更に関わらず、Claudeターミナルのワーキングディレクトリを固定できる。

```lua
-- gitリポジトリのルートを使用
require("claudecode").setup({
  git_repo_cwd = true,
})

-- 静的パスを指定
require("claudecode").setup({
  terminal = {
    cwd = vim.fn.expand("~/projects/my-app"),
  },
})

-- カスタムプロバイダー関数
require("claudecode").setup({
  terminal = {
    cwd_provider = function(ctx)
      local cwd = require("claudecode.cwd").git_root(ctx.file_dir or ctx.cwd)
        or ctx.file_dir
        or ctx.cwd
      return cwd
    end,
  },
})
```

---

## 9. 公式プラグインシステム

GitHub `anthropics/claude-code/plugins` ディレクトリには、Anthropic公式のプラグインが含まれている。初学者に特に有用なものを以下に挙げる。

| プラグイン名                | 説明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `commit-commands`           | `/commit`, `/commit-push-pr`, `/clean_gone` のgitワークフロー自動化 |
| `code-review`               | `/code-review` で5つの並列Sonnetエージェントによる自動PRレビュー |
| `frontend-design`           | フロントエンド作業時に自動発動し、本格的なデザインガイダンスを提供 |
| `ralph-wiggum`              | `/ralph-loop` で反復的な自律開発ループを実行                 |
| `hookify`                   | `/hookify` で会話パターンや指示からカスタムHookを簡単に作成  |
| `security-guidance`         | ファイル編集時にセキュリティパターン（コマンドインジェクション、XSS、eval等）を監視するHook |
| `feature-dev`               | `/feature-dev` で7フェーズの体系的な機能開発ワークフロー     |
| `plugin-dev`                | `/plugin-dev:create-plugin` でプラグイン開発の8フェーズガイド |
| `agent-sdk-dev`             | `/new-sdk-app` でAgent SDKアプリの対話的セットアップ         |
| `claude-opus-4-5-migration` | Sonnet 4.x / Opus 4.1 から Opus 4.5 への自動移行             |
| `explanatory-output-style`  | 実装選択やコードベースパターンについて教育的な知見を追加するHook |
| `learning-output-style`     | 判断ポイントでユーザーに有意義なコード記述を促すインタラクティブ学習モード |
| `pr-review-toolkit`         | `/pr-review-toolkit:review-pr` でコメント、テスト、エラー処理、型設計、コード品質、簡素化の観点別レビュー |

### プラグインの構造