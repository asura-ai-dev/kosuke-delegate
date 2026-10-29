---
name: hand-kosuke
description: "PR を作ったあと、Kosuke（kosuke CLI）に QA とレビューを投げ、30 分後から 15 分おきに監視し、終わったら結果を 1 枚の HTML にして .kosuke/kosuke-output/ に出す。「kosukeで」「kosukeに投げて」で報告だけ（--fix なし）、「kosukeで修正も」「kosukefix」で --fix 付き。"
---

# hand-kosuke

PR のブランチを Kosuke に渡して QA とレビューを走らせ、監視して、結果を HTML にまとめる。マージはしない。

## いつ使う

- 「kosukeで」「kosukeに投げて」→ **報告だけ**（`--fix` なし。コードは変えさせない）
- 「kosukeで修正も」「kosukefix」→ **`--fix` 付き**（Kosuke が直して commit する。[注意](#fix-の注意)）
- 使わない: Kosuke が未ログイン（`kosuke auth status` が失敗）。ユーザーに `kosuke auth login` を頼む。

## 流れ

1. **PR を確かめる**。無ければ `gh pr create`（ユーザーが頼んだときだけ）。head ブランチと PR 番号を控える
2. **projectId を引く**: `kosuke projects list` の `githubRepoUrl` が今のリポと一致する行。無ければ止まって聞く
3. **投げる**: `kosuke sessions create <projectId> --start-branch <head> --title … --content …` で 1 セッション。文面は [template.md](template.md) の「依頼文」。QA は [example.md](example.md) のとおり `sessions send` で続けて積む（レビュー → QA の順）
4. **監視を予約する**（CronCreate）: **30 分後に 1 回** → まだ動いていれば **15 分おき**。終わったら CronDelete。文面は [example.md](example.md)
5. **終わったら読む**: `sessions get` / `messages` / `artifacts get`。動画は署名付き URL なので `curl` で保存する
6. **HTML を作る**: [template.md](template.md) を埋めて `.kosuke/kosuke-output/<id>/index.html` に出す（`state.json`・`media/`・`sources/` も）
7. **3 行で報告**: 合否、原因、次の一手。マージはユーザーに任せる

## 守ること

- `--fix` を付けるのは「修正も」「kosukefix」と言われたときだけ。付けた回は、始める前に「Kosuke が commit する」と一言伝える
- API キーを出力に出さない。CLI の全コマンドは [cli-list.md](cli-list.md)
- 動画は私には見えない。画面の説明はテスターの文字の記録から書き、そう明記する
- BLOCKED は「壊れていない」の意味ではない。原因（偽 worker、データ不足、版の未検証）を分けて書く
- PR の head に他の人や codex が push していないか、始める前に `git fetch` で見る

## fix の注意

Kosuke の設定が Automatic だと、修正は commit まで自動で進む。どのブランチに積まれるかは設定と実行で変わる（PR の head に直接入ることがあった）。終わったら `git fetch` と `git log origin/<head>`、`git ls-remote origin 'kosuke/*'` で増えた commit を見てから、ユーザーに渡す。ほかの push と競合しうるので、始める前にも `git fetch` で確かめる。
