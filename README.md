# claude-config

Claude Code のグローバル設定。`~/.claude/` 配下の共有可能なファイルをここで管理し、実体へのシンボリックリンクを `~/.claude/` に置く。

## 構成

| ファイル | 役割 |
|---|---|
| `CLAUDE.md` | 全プロジェクト共通の指示(t_wada principle、サブエージェントルーティングなど) |
| `agents/` | サブエージェント定義(searcher / implementer / reasoner / reviewer) |
| `settings.json` | ハーネス設定・hooks(WSL 向け通知音。Windows 依存あり) |

## セットアップ(新しいマシン)

```bash
git clone git@github.com:omakeno/claude-config.git ~/claude-config
ln -s ~/claude-config/CLAUDE.md ~/.claude/CLAUDE.md
ln -s ~/claude-config/agents ~/.claude/agents
ln -s ~/claude-config/settings.json ~/.claude/settings.json
```

`~/.claude/` 全体は認証情報・履歴を含むため repo 化しないこと。
