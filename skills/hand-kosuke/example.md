# 実例: PR #<番号> に QA を投げる

## 1. 投げる（報告だけ）

```bash
kosuke projects list | python3 -c "import json,sys;[print(p['id'],p['githubRepoUrl']) for p in json.load(sys.stdin)['data']]"
# → <projectId>  https://github.com/<owner>/<repo>

kosuke sessions create <projectId> \
  --start-branch <head ブランチ> \
  --title "PR #<番号> の QA とレビュー" \
  --content "<template.md の依頼文（レビュー）>"
# → data.session.id = <sessionId>（以降 SESSION）

kosuke sessions send <projectId> SESSION --content "<template.md の依頼文（QA）>"   # 走行中は Up next に積まれる
```

QA だけを回すこともできる（約 5 分で終わった例がある）。レビューを足すときは上のように 2 通目を積む。`--fix` を付ける回は、依頼文の `/kosuke:…` の後ろに `--fix` を足す。

## 2. 監視を予約する（CronCreate）

**30 分後に 1 回**（分は 0 / 30 を避ける。例: 今 16:20 なら 16:51）:

```
cron: "51 16 28 10 *"   recurring: false
prompt: |
  hand-kosuke の監視（1 回目）。project=<projectId> session=<sessionId> repo=<repo のパス> pr=<番号> mode=report。
  1. kosuke sessions get <project> <session> の latestChatJob.status を見る。
  2. running / queued なら、CronCreate で 15 分おきの監視（cron "7,22,37,52 * * * *"、recurring: true、同じ内容）を作り、ID を控える。
  3. completed / failed / cancelled なら、messages と artifacts を読み、動画を保存し、template.md で HTML を作る。
     そのあと CronDelete（15 分おきの ID があれば）と、3 行の報告。
```

**15 分おき**は上の 2 の中で作る。終わったら必ず `CronDelete`（7 日で自動失効、セッション内のみ。Claude を閉じると消える）。

## 3. 終わったら読む

```bash
P=<projectId>; S=<sessionId>
kosuke sessions get $P $S | python3 -c "import json,sys;d=json.load(sys.stdin)['data'];print(d['latestChatJob'],d['prUrl'])"
kosuke sessions messages $P $S --limit 1        # 最終メッセージ（text ブロックが結論）
kosuke sessions artifacts list $P $S
for s in qa-report-custom-1 qa-checklist-1; do kosuke sessions artifacts get $P $S $s > $s.json; done
kosuke sessions artifacts get $P $S qa-video-1-s1-a1   # data.url（署名付き）を curl -o media/s1.mp4
```

## 4. 出力の置き場

```
<repo>/.kosuke/kosuke-output/<yyyymmddThhmmss-qa<PR番号>>/
  index.html   state.json   media/*.mp4   sources/*
```

`state.json` の形は下のとおり。

## 5. 今回分かったこと

- `sessions create` は `--start-branch` で PR のブランチを指す。CLI に QA を直接起動するコマンドは無く、依頼文に `/kosuke:automated-qa` と書くと走った
- QA 環境の worker は `dev-echo`（偽の応答）で依頼データが 0 件。実際の失敗・報告・ローテーションは BLOCKED になる。種データか結合テストで補う
- カスタムのチェックリストだと PR にラベルもコメントも付かない
- 動画の署名付き URL は期限切れになるので、その場で保存する
- PR のブランチには、頼んでいない `Fix adversarial review findings …` が Kosuke から push されていた（Review & QA の `--fix` と思われる）。始める前に `git fetch` で他の push を確かめ、`--fix` の回は競合に備える
