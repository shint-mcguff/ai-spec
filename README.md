# ai-spec

Claude Code 用スキル。リポジトリと同じディレクトリに gitignore した `.ai/` を置き、AIがそこに spec を書き、そこを開発の **SSOT（single source of truth）** にして進める。

チャット履歴は消える。使うAIも変わる（Claude Code / Codex / Cursor / ChatGPT）。だから真実はチャットでなくファイルに置く。議論するたびに `.ai/SPEC.md` を更新し、次のセッションはそれを読んでから始める。

## `.ai/` の構成

| ファイル | 役割 | 書き方 |
|---|---|---|
| `.ai/README.md` | AI向けの運用プロトコル（どのAIでも同じ） | テンプレのまま |
| `.ai/SPEC.md` | **現在の真実**。目的・スコープ・設計・現状・未決・次アクション | 状態なので書き換える |
| `.ai/DECISIONS.md` | **決定ログ**。いつ・何を・なぜ決め、何を捨てたか | 追記のみ |
| `.ai/notes/` | 任意。調査メモ・下書き | 自由 |

root の `CLAUDE.md` と `AGENTS.md` は「まず `.ai/SPEC.md` を読め」の数行のポインタ。こちらはコミットするので、Claude Code 以外のAIも同じ入口から入れる。

## インストール

ユーザーグローバル（全リポジトリで使う）:

```bash
git clone https://github.com/shint-mcguff/ai-spec.git
mkdir -p ~/.claude/skills
cp -r ai-spec/skills/ai-spec ~/.claude/skills/ai-spec
```

プロジェクト単位（そのリポジトリだけで使う）:

```bash
cp -r ai-spec/skills/ai-spec <your-repo>/.claude/skills/ai-spec
```

## 使い方

```
/ai-spec init <path> [name]   # 新しいリポジトリを立て、.ai/ と CLAUDE.md/AGENTS.md ポインタを置いて初回コミット
/ai-spec adopt [path]         # 既存リポジトリに後付け。「現状」はコードを読んで埋める
/ai-spec sync                 # 直近の議論を SPEC.md / DECISIONS.md に書き戻す
```

`.ai/SPEC.md` があるリポジトリでは、明示しなくても sync の運用（開始時に読む・決定は即追記・終了前に次アクションを書く）に従う。

## 鉄則

- **SPEC は状態、DECISIONS はログ**。SPEC を追記で伸ばさない。DECISIONS を書き換えない
- **チャットにしか無い決定を作らない**。合意した瞬間に書く
- **憶測を書かない**。未確認は `未確認` と明記する
- **`.ai/` はローカル**。別マシンやチームで共有したくなったら `.gitignore` から外す

## 他のAIで使う

スキル機構は Claude Code のものだが、`.ai/README.md` に運用プロトコルが書いてあるので、`AGENTS.md` を読むツール（Codex・Cursor 等）はそのまま同じ運用に乗れる。

## License

MIT
