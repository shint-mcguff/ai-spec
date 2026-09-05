---
name: ai-spec
description: 新しいリポジトリを立てるとき、gitignoreした `.ai/` フォルダにAIが書くspec（SPEC.md＝現在の真実、DECISIONS.md＝決定ログ）を置き、そこを開発のSSOTにして進める。「新しいrepo立てて／プロジェクト始めたい／specをSSOTにして／.aiを整備して」と頼まれたら使う。既存リポジトリへの後付け（adopt）と、議論のたびにspecへ書き戻す運用（sync）も担う。チャット履歴や使うAIが変わっても、`.ai/SPEC.md` を読めば同じ地点から再開できるようにするのが狙い。Usage: /ai-spec init <path> [name] | /ai-spec adopt [path] | /ai-spec sync
---

# ai-spec — `.ai/` をリポジトリ開発のSSOTにする

コードと同じディレクトリに `.ai/` を置き、AIがそこへspecを書く。`.ai/` は `.gitignore` に入れるので、コミット履歴を汚さずに議論の結果を溜められる。チャット履歴は消えるし、明日は別のAI（Codex・Cursor・ChatGPT）を使うかもしれない。だから**真実はチャットでなくファイルに置く**。議論するたびに `.ai/SPEC.md` を更新し、次のセッションはそれを読んでから始める。

## `.ai/` の構成

| ファイル | 役割 | 書き方 |
|---|---|---|
| `.ai/README.md` | AI向けの運用プロトコル（どのAIでも同じ） | テンプレのまま。触らない |
| `.ai/SPEC.md` | **現在の真実**。目的・スコープ・設計・現状・未決・次アクション | 状態なので**書き換える**。追記で伸ばさない |
| `.ai/DECISIONS.md` | **決定ログ**。いつ・何を・なぜ決め、何を捨てたか | ログなので**追記のみ**。過去の行は直さない |
| `.ai/notes/` | 任意。調査メモ・議事・下書きの置き場 | 自由。SPECに昇格したら消してよい |

root の `CLAUDE.md` と `AGENTS.md` は「まず `.ai/SPEC.md` を読め」と書いた数行のポインタ。これはコミットする（`.ai/` 本体はコミットしない）。

## init — 新しいリポジトリを立てる

`/ai-spec init <path> [name]`。path が無ければ1問だけ聞く。name の既定はフォルダ名。

1. **場所を確認する**。既に存在してgit管理下なら init でなく adopt に切り替える
2. **骨格を作る**（テンプレは本スキルの `templates/`）
   ```bash
   mkdir -p <path>/.ai/notes && cd <path> && git init -b main
   ```
   - `.gitignore` に次を書く（既存があれば `.ai/` の行だけ追加）
     ```
     # AI作業用spec（ローカルSSOT。共有したくなったらこの行を外す）
     .ai/
     .DS_Store
     .claude/settings.local.json
     ```
   - `.ai/README.md` ← `templates/ai-README.md`
   - `.ai/SPEC.md` ← `templates/SPEC.md`（`{{name}}` `{{date}}` を置換）
   - `.ai/DECISIONS.md` ← `templates/DECISIONS.md`
   - `CLAUDE.md` と `AGENTS.md` ← `templates/pointer.md`（同じ内容）
   - `README.md` は name と1行の説明だけ
3. **初回コミット**: `git add -A && git commit -m "chore: bootstrap repo with .ai spec folder"`。`git status` で `.ai/` が untracked に出ないことを確認する
4. **リモートは作らない**。GitHubへの作成・push は外向きの操作なので、ユーザーが指示したときだけ `gh repo create` を提案して実行する
5. **最初の議論をする**。目的・スコープ・技術選定をユーザーとの対話で詰め、結果を `SPEC.md` に書く（下の sync と同じ要領）。SPECが空のまま実装に入らない

## adopt — 既存リポジトリに後付けする

`/ai-spec adopt [path]`（既定はカレント）。init の 2〜3 を既存構造を壊さずに行う。

- `.gitignore` は追記のみ。`CLAUDE.md` / `AGENTS.md` が既にあれば、先頭にポインタ2行を足すだけで既存本文は残す
- `SPEC.md` の「現状」はコードを読んで埋める（README・package.json・ディレクトリ構成・主要エントリ）。憶測で書かない。分からない欄は `未確認` と書く
- 既存の設計ドキュメント（docs/、ADR、Notion 等）があれば SPEC から**リンクする**。転記して二重管理にしない

## sync — 議論のたびに書き戻す

明示の `/ai-spec sync` だけでなく、**`.ai/SPEC.md` があるリポジトリで作業しているときは常にこの運用に従う**。

セッション開始時:
1. `.ai/SPEC.md` を読む。次に `.ai/DECISIONS.md` の末尾10件
2. 「現状」と実際のコードがズレていたら、先に SPEC を直してから本題に入る（ズレたSSOTは無いのと同じ）

議論・作業の途中と終了時:
3. **決めたことは即 DECISIONS.md に追記**する（日付・決定・理由・捨てた案。1件3〜6行）。「後でまとめて」にしない。決定が起きた発言の直後に書く
4. **SPEC.md の該当セクションを書き換える**。スコープ・設計・未決・次アクションを現在形に揃える。古い記述は消す（経緯は DECISIONS にある）
5. 未決のまま終わる論点は「未決事項」に**問いの形**で残す。「検討中」とだけ書かない
6. セッション終了前に「次アクション」を、次のAIが**そのまま着手できる粒度**（ファイル名・コマンド・受け入れ条件）で書く

## 鉄則

- **SPEC は状態、DECISIONS はログ**。SPEC を追記で伸ばすと「どれが今の真実か」が分からなくなる。DECISIONS を書き換えると「なぜ」が消える
- **チャットにしか無い決定を作らない**。ユーザーと合意した瞬間に書く。書けていない決定は無かったことになる前提で動く
- **憶測を書かない**。未確認の数値・依存・仕様は `未確認` と明記する。次のAIが確認済みと誤読するのが一番危ない
- **`.ai/` はローカル**。gitignore なので別マシンには届かない。チームや別端末で共有したくなったら、`.gitignore` から外してコミットするか、`.ai/` を丸ごとコピーする。判断はユーザー
- **文体**: 読み手はAIと人の両方。装飾・比喩・メタ宣言を足さず、事実と決定だけを短く書く
- 不可逆操作（リモート作成・push・削除・公開）はユーザーの承認を取ってから行う

## 完成チェック（init / adopt の出力前）

- [ ] `git status` に `.ai/` が出ない（gitignore が効いている）
- [ ] `CLAUDE.md` と `AGENTS.md` の先頭が `.ai/SPEC.md` へのポインタになっている
- [ ] `SPEC.md` の `{{name}}` `{{date}}` が置換済み。空欄は `未確認` か `未決` で埋め、憶測が無い
- [ ] `DECISIONS.md` に「.ai/ をSSOTにする」の初回エントリがある
- [ ] 最初の議論を経て、SPEC の目的・スコープが ユーザー自身の言葉で書けている（テンプレの説明文が残っていない）
