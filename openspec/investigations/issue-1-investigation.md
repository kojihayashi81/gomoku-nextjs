# Issue #1 調査ノート

## 調査対象

**タイトル:** 五目並べ（連珠）基本ルールと禁じ手の調査・整理

**概要:** 15×15 盤面・黒先手・5 連勝利を基本とする五目並べ（連珠）のルールと、黒番のみに適用される禁じ手（三三・四四・長連）を箇条書きで整理する。完了条件は基本ルールおよび禁じ手の定義を文書化することである。

**調査の焦点:**

- 活三（open three）の厳密定義と判定ロジック（両端が開いている三の条件、飛び三の扱い）
- 三三禁じ手の同時成立判定（1 手で活三が 2 方向以上に成立する条件）
- 四四禁じ手の同時成立判定（1 手で四が 2 方向以上に成立する条件）
- 長連（6 個以上連続）の判定と白番への非適用
- 禁じ手チェックの実装パターン（仮置き→判定→戻し方式）

## 調査プロセス

Code Investigator によるリポジトリ静的解析と Web Investigator による外部情報収集の組み合わせで実施した。Web 調査は今回も失敗しており、公式連珠ルールなどの一次情報は取得できていない。

- **検索・参照したファイル・関数:**
  - `openspec/investigations/issue-1-investigation.md:1-116` — Issue #1 の既存調査ノート（禁じ手定義の暫定まとめ・外部 PR 参照・Web 調査未完了の旨を含む）
  - `.devcontainer/devcontainer.json:1-16` — Node.js 22 + `@anthropic-ai/claude-code` の開発環境定義
  - `.devcontainer/Dockerfile:1-19` — Ubuntu ベース + Node.js 22 + gh CLI のコンテナイメージ定義
  - `.github/ISSUE_TEMPLATE/ai-task.yml:1-44` — AI タスク Issue テンプレート（本 Issue #1 の元フォーマット）
  - `.claude/commands/investigate.md:1-80` — investigate スキルの実行手順定義（調査→保存→PR 作成ワークフロー）
  - `.claude/scripts/save-investigation.sh:1-12` — 調査ノートを `openspec/investigations/` に書き出すシェルスクリプト

- **参照した既存ノート・仕様書:**
  - `openspec/investigations/issue-1-investigation.md` — 本調査の更新対象。前回の調査で作成された暫定定義まとめが記載されており、活三の厳密な定義と公式ルール一次情報が未取得のまま残っている

- **検討して除外した方向性:**
  - リポジトリ内のゲームロジックコードの参照 — 本リポジトリはグリーンフィールド状態であり、package.json・Next.js・TypeScript・ゲームロジック関連ファイルが存在しないため対象外
  - 外部リポジトリ（kojihayashi81/gomoku-nextjs）の PR コードの直接解析 — 今回の調査スコープでは実施せず。参考リンクとして記録するにとどめた

## 調査結果

### サマリー

本リポジトリは五目並べのゲームロジックを一切含まないグリーンフィールド状態であり、Issue #1 は「実装タスク」ではなく「ルール文書化タスク」である。基本ルール（15×15 盤面・黒先手・5 連勝利）と禁じ手 3 種（三三・四四・長連）の暫定定義は整理済みだが、三三の判定核心となる「活三」の厳密な定義については、Web 調査が複数回失敗しており、公式連珠ルールによる一次情報確認が取れていない。

### 詳細

**基本ルール:**
- 盤面: 15×15
- 先手: 黒（黒が必ず先に打つ）
- 勝利条件: 縦・横・斜めのいずれかで自石を 5 個連続して並べる
- 禁じ手は**黒番のみ**に適用（白番には適用されない）

**禁じ手の種類（黒のみ）:**

1. **長連（ちょうれん / overline）**
   - 定義: 黒石が 6 個以上連続して並ぶ着手
   - 効果: 打った時点で黒の負け
   - 備考: 白番は長連を打っても反則とならず、6 連以上でも勝利となる

2. **四四（しし / double-four）**
   - 定義: 黒が 1 手で同時に 2 つ以上の「四」を成立させる着手
   - 四の定義（暫定）: 次の 1 手で 5 連（勝利）になれる状態（両端の開閉は問わない）
   - 効果: 打った時点で黒の負け

3. **三三（さんさん / double-three）**
   - 定義: 黒が 1 手で同時に 2 つ以上の「活三」を成立させる着手
   - 活三の定義（暫定）: 次の 1 手で四になれる三であり、かつ両端が空いている（開いている）もの
   - 効果: 打った時点で黒の負け

**実装上のアプローチ（Collector の調査焦点より）:**
- **仮置き→判定→戻し** パターンを採用する
  1. 着手候補マスに仮で石を置く
  2. 禁じ手条件（三三・四四・長連）を判定する
  3. 石を戻して盤面状態を元に戻す
- 縦・横・右斜め・左斜めの 4 方向それぞれで連続個数をカウントする関数が必要

**リポジトリ状態（Code Investigator より）:**
- `.devcontainer`・`.github/ISSUE_TEMPLATE`・`.claude/commands` のみを含む CI/ワークフロー基盤のみが存在する
- 技術スタック: Node.js 22（devcontainer 指定）、`@anthropic-ai/claude-code`（postCreateCommand でインストール）、GitHub CLI（apt 経由）
- Next.js・TypeScript などのゲームアプリ向けフレームワークはまだ追加されていない

## 要確認事項

1. **活三の厳密な定義（三三判定の核心）**
   - 「両端が空いている三」という暫定定義は、Web 調査が複数回失敗しているため公式連珠ルール（一次情報）での確認が取れていない
   - Code Investigator の search_hints が示す検索キーワード `renju international federation official rules forbidden moves sansan shishi overline` で連珠世界連盟（RIF）公式ルール文書を別途参照することを推奨する
   - 飛び三（間に空きがある三）を活三として扱うかどうかも未確認

2. **四の定義（四四判定に関連）**
   - 「次の 1 手で 5 連になれる状態」という暫定定義で十分か、または「長連にならない 5 連のみ」に限定するかが不明確
   - 四四と長連が同時に成立する場合の優先順位（どちらの禁じ手を適用するか）も要確認

3. **三三の例外規定**
   - 三三でも同時に 5 連を作った場合は勝利（禁じ手が無効になる例外）の有無が一般五目並べと連珠で異なるが、本 Issue の対象ルール（一般五目並べか連珠か）が明示されていない

4. **外部 PR の実装との整合性**
   - kojihayashi81/gomoku-nextjs の PR #3・#4・#5・#7 が禁じ手を実装している可能性があるが、今回はコード確認を行っていない。本プロジェクトへの流用・参考にする方針があるかを Issue 作成者に確認することを推奨する

5. **カジュアルモード（禁じ手なし）の設計判断**
   - Issue の完了条件（基本ルールの箇条書き + 禁じ手の種類の記載）には明示されていないが、設計上の判断が必要になる可能性がある

## 外部参考ソース

（Web 調査はスキップ — Web Investigator が複数回失敗したため、official_docs および similar_issues の情報は取得できていない）

**Issue にリンクされた外部リポジトリの PR（Collector より）:**
- `kojihayashi81/gomoku-nextjs` PR #3: https://github.com/kojihayashi81/gomoku-nextjs/pull/3
- `kojihayashi81/gomoku-nextjs` PR #4: https://github.com/kojihayashi81/gomoku-nextjs/pull/4
- `kojihayashi81/gomoku-nextjs` PR #5: https://github.com/kojihayashi81/gomoku-nextjs/pull/5
- `kojihayashi81/gomoku-nextjs` PR #7: https://github.com/kojihayashi81/gomoku-nextjs/pull/7

**推奨する次回調査先（Code Investigator の search_hints より）:**
- `renju international federation official rules forbidden moves sansan shishi overline`
- `連珠 活三 厳密な定義 両端空き 判定アルゴリズム`
- `gomoku forbidden moves algorithm implementation three-three four-four overline detection`
- `kojihayashi81 gomoku-nextjs forbidden moves implementation TypeScript`

> 注意: 外部リポジトリの PR は公式ドキュメントではなく、一次情報としての信頼性は未検証です。
