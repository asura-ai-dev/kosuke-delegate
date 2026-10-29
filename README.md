# kosuke-delegate

PR を作ったあと、[Kosuke](https://kosuke.ai) の CLI に QA と二重レビューを投げ、監視して、結果を 1 枚の HTML にまとめる Claude Code のスキル `hand-kosuke` です。マージはしません。

## 使い方

1. Node 22.18 以上で CLI を入れ、ログインします（ログインはご自身で）。

   ```bash
   npm install -g @kosuke-ai/cli
   kosuke auth login
   ```

2. このスキルを入れます（Claude Code）。

   ```
   /plugin marketplace add asura-ai-dev/kosuke-delegate
   /plugin install kosuke-delegate@kosuke-delegate
   ```

   Codex の場合は `skills/hand-kosuke` を `~/.codex/skills/` に置いてください。

3. PR を作ったあと、エージェントに言います。
   - 「kosukeで」 … 報告だけ（`--fix` なし。コードは変えません）
   - 「kosukeで修正も」「kosukefix」 … `--fix` 付き（Kosuke が commit します）

出力は `<repo>/.kosuke/kosuke-output/<id>/index.html` に出ます。

## 注意

- `--fix` の回は、Kosuke が commit を積みます。始める前に `git fetch` で他の push を確かめてください。
- 動画の中身はエージェントには見えません。画面の説明はテスターの文字の記録から書きます。
- Kosuke の使用には、Kosuke の登録と、Claude Code / Codex などの契約が別に要ります。

## ライセンス

MIT
