# kosuke CLI の早見（hand-kosuke で使うもの）

`kosuke --help` は npm の `@kosuke-ai/cli`（Node 22.18 以上）。出力は JSON。ID は位置引数、値はフラグ。

## 認証

| コマンド | 用途 |
|---|---|
| `kosuke auth status` | ログイン状態（キーは出ない） |
| `kosuke auth login` | キーを貼って保存（本人が行う） |
| `KOSUKE_API_KEY` / `--api-key` | CI やスクリプト用。保存キーより優先 |

## プロジェクト

| コマンド | 用途 |
|---|---|
| `kosuke projects list` | `id` / `name` / `githubRepoUrl` / `deploymentStatus` / `qaGuide` |
| `kosuke projects get <projectId>` | 1 件 |
| `kosuke projects branches <projectId>` | ブランチ一覧 |
| `kosuke projects qa-guide set <projectId> --content "…"` | QA ガイドを保存 |

## セッション

| コマンド | 用途 |
|---|---|
| `kosuke sessions create <projectId> --start-branch <b> --title … --content …` | 開始（すぐ走る）。`--status backlog` で後回し |
| `kosuke sessions send <projectId> <sessionId> --content …` | 次の依頼を積む。走行中は Up next で待つ |
| `kosuke sessions steer …` | 走行中の turn に割り込む |
| `kosuke sessions get <projectId> <sessionId>` | `session`・`latestChatJob`（status / errorCode）・`prUrl` |
| `kosuke sessions jobs list <projectId> <sessionId>` | turn ごとの job と状態 |
| `kosuke sessions jobs cancel <projectId> <sessionId> <jobId>` | 走行中の turn を止める |
| `kosuke sessions watch <projectId> <sessionId> <jobId> --timeout <秒>` | NDJSON で追う。時間切れは exit 124 |
| `kosuke sessions messages <projectId> <sessionId> [--limit N]` | 会話。`data.messages[].blocks[]`（type: text / tool / artifact） |
| `kosuke sessions message <projectId> <sessionId> <messageId>` | 1 件を全文で |
| `kosuke sessions artifacts list <projectId> <sessionId>` | 成果物の行（本文なし） |
| `kosuke sessions artifacts get <projectId> <sessionId> <slug>` | markdown / html は `body`、動画・画像は署名付き `url`（期限あり） |
| `kosuke sessions files list|get <projectId> <sessionId>` | サンドボックス内のファイル（止まっていると空） |
| `kosuke sessions needs-you` | 自分の待ち・走行中・最近 |
| `kosuke sessions update …` / `delete …` | 変更・削除（削除は聞いてから） |

## スキル（依頼文に書いて呼ぶ。CLI に直接の起動コマンドは無い）

| スキル | 内容 |
|---|---|
| `/kosuke:automated-qa` | Testing Checklist を動くアプリに対して実行（約 45 分）。`--fix` / `--restart` / `--sync` |
| `/kosuke:adversarial-review` | 複数モデルでレビュー。`--fix` / `--quick` |
| `/kosuke:open-pr` | PR を作成・更新 |

## 結果の読み方

- `latestChatJob.status`: `running` の間は待つ。`completed` / `failed` / `cancelled` で終わり
- QA の合否は PR のラベル（`qa: passed|failed|blocked|skipped`）。カスタムのチェックリストだとラベルは付かず、最終メッセージとレポート Artifact に出る
- レビューの判定は `review: passed|open items|failed`
