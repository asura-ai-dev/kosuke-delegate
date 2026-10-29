# template: 依頼文と HTML

## 依頼文（レビュー）

```
/kosuke:adversarial-review を、ブランチ <head>（PR #<番号>、base は <base>）に対して実行してください。
[報告だけの回] --fix は付けない。修正の commit も push もしない。報告だけ。
[fix の回] --fix を付けて実行する。
この PR の変更: <箇条書き 3〜5 行。ファイルと意味>
設計の正本: <docs のパスと章>
```

## 依頼文（QA）

```
/kosuke:automated-qa を、同じブランチに対して実行してください。[報告だけの回は --fix なし・commit なし]
確かめてほしいこと: (1) … (2) … (3) …
実サービスで確かめられない項目は BLOCKED にして理由を書いてください。iOS など対象外は先に書く。
```

## HTML（1 枚）

出力先: `<repo>/.kosuke/kosuke-output/<id>/index.html`。`<id>` は `yyyymmddThhmmss-qa<PR番号>`。
外部読み込みなし・1 ファイル・ライト/ダーク対応。動画は `media/` の mp4 を `<video controls>` で。

節の並び（この順を変えない）:

1. **結論** — 1〜2 文。確かめられたこと / 確かめられていないこと。PASS / BLOCKED / FAILED の数
2. **結果と録画** — 項目ごとのカード（バッジ、説明、足りないもの、動画）
3. **原因** — BLOCKED / FAILED の理由を 3 つ前後のカードに
4. **対策** — 案の表（案 / 効くところ / 手間）と、おすすめ 1 行
5. **今後の流れ** — 番号つき 5 段。各段に担当（あなた / 私 / cloud）
6. **やるべきこと** — 優先・内容・担当の表
7. **フッター** — 元データのパス、Kosuke のセッション ID、動画を見られていない旨

骨組み（スタイルは `.kosuke/kosuke-output` の既存ページからコピーする。無ければ下の最小版）:

```html
<!doctype html><html lang="ja"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{PR}} QA 結果と次の一手</title>
<style>
:root{--bg:#f9f9f8;--surface:#fff;--line:#e5e4e0;--ink:#1c1c1a;--muted:#6a6962;--pass:#2f6b3f;--pass-bg:#e4f0e7;--block:#8a5a12;--block-bg:#f6ecd6;--fail:#9a2f2f;--fail-bg:#f6e3e3}
@media (prefers-color-scheme:dark){:root{--bg:#161614;--surface:#1e1e1b;--line:#33332f;--ink:#ecebe6;--muted:#a3a29a;--pass:#7cc18f;--pass-bg:#1c2e21;--block:#e3b867;--block-bg:#382e18;--fail:#e38d8d;--fail-bg:#3a1f1f}}
body{margin:0;background:var(--bg);color:var(--ink);font:15px/1.75 system-ui,"Hiragino Sans",sans-serif;padding:32px 16px 64px}
main{max-width:960px;margin:0 auto;display:flex;flex-direction:column;gap:32px}
.card{background:var(--surface);border:1px solid var(--line);border-radius:6px;padding:16px 18px}
video{width:100%;border-radius:4px}
</style></head><body><main>
<header><h1>{{PR}} の QA 結果と、次の一手</h1><p>{{対象ブランチ・sha・日付}}</p></header>
<section class="card"><h2>結論</h2>{{結論}}</section>
<section><h2>結果と録画</h2>{{カード × 項目数: バッジ + 説明 + <video controls src="media/sN.mp4">}}</section>
<section><h2>原因</h2>{{カード × 3}}</section>
<section><h2>対策</h2>{{表}}</section>
<section class="card"><h2>今後の流れ</h2>{{5 段}}</section>
<section><h2>やるべきこと</h2>{{表}}</section>
<footer>{{元データ・セッション ID・動画は未視聴の旨}}</footer>
</main></body></html>
```

`state.json`:

```json
{ "id": "<id>", "spec": "<PR と目的>", "format": "html", "status": "ready", "entry": "index.html",
  "source": { "tool": "kosuke", "project_id": "…", "session_id": "…", "start_branch": "…", "tested_sha": "…" },
  "result": { "passed": 0, "failed": 0, "blocked": 0 },
  "media": ["media/s1.mp4"], "sources": ["sources/qa-report-….html"], "createdAt": "<ISO>" }
```

書き方の約束:

- 録画の中身は見られない。説明は「テスターの記録によると」の立場で書く
- 原因は PR のコードと環境を分ける（BLOCKED の多くは環境）
- 対策は「決定的に確かめる > 実環境で通す > 録画つきで残す」の順で勧める
