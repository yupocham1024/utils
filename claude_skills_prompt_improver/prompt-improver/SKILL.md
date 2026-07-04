---
name: prompt-improver
description: TRIGGER — invoke this before starting work on ANY new user message, in ANY project, to check whether the raw prompt is vague/subjective/underspecified and should be rewritten into a more concrete, objective, and actionable version before acting on it. Always check this on a fresh user request, not just when the user explicitly asks for prompt help. SKIP entirely (do not mention this skill, do not rewrite) when the message is: a plain yes/no answer; a selection from choices already presented (A/B, number picks, etc.); a short acknowledgement/continuation with no new information ("続けて", "それでお願いします", "OK", "そのまま進めて", etc.); feedback or a correction on your immediately preceding output where the target is obvious from context; a slash command; pasted logs/code/errors/data whose intended handling is self-evident from context; or a request that is already concrete and specific (named files, numbers, explicit constraints, explicit output format).
---

# Prompt Improver

## 目的

ユーザーが投げる自然文の指示は、時に曖昧・主観的・情報不足であり、そのまま実行すると意図とズレた結果を返してしまうことがある。作業を始める前に一度プロンプトを点検し、必要な場合だけ「より具体的で」「より客観的で」「適切な出力が得られる」形に書き換え、ユーザーの承認を得てから進める。

## 実行フロー

1. 直近のユーザーメッセージを読む。
2. 下記の「スキップ条件」に該当するなら、何もしない。このスキルに言及せず、書き換え案も出さず、通常通り作業を進める。
3. 該当しない場合、プロンプトに次のような曖昧さがないか点検する。
   - 主語・対象・範囲が不明(「これ直して」→ 何を指すのか特定できるか)
   - 主観的な形容詞・副詞に基準がない(「いい感じに」「きれいに」「なるべく早く」など)
   - 期待する出力形式・成果物が書かれていない
   - 前提条件・制約(対象ファイル、期間、件数、精度など)が欠けている
   - 複数の解釈が成立してしまう表現
4. 曖昧さが見つかった場合、次の方針で書き換える。
   - ユーザーが言っていないことを勝手に創作しない。情報が本当に足りない場合は、妥当な前提を明示した上で提案する(黙って仮定しない)。
   - 直前のやり取り・開いているファイル・プロジェクトの性質など、会話の文脈から拾える情報は積極的に使って具体化する。
   - 対象・数値・範囲・出力形式を可能な限り明文化する。
5. 曖昧さが見当たらず、すでに具体的・客観的なら、書き換えずそのまま進める。無理に書き換えを作らない。
6. 書き換えが必要な場合は、まずチャット本文に書き換え後のプロンプトを表示し、そのあとで `AskUserQuestion` ツールを使って選択肢を提示する。**ユーザーが選ぶまで作業を開始しない**。自由記述の「承認待ち」ではなく、必ずダイアログ形式で選ばせる。

## 提示フォーマット

### ステップ1: チャット本文で書き換え案を表示する

`AskUserQuestion` を呼び出す前に、通常のテキスト出力として以下のように示す。長いプロンプトでもダイアログ内に押し込めず、ここで読める形にする。

```
プロンプトを以下のように解釈しました。

> (書き換え後のプロンプト全文)
```

### ステップ2: `AskUserQuestion` で選ばせる

- `question`: 「この内容で進めますか?」など短い確認文のみ。書き換え後のプロンプト全文は埋め込まない(ステップ1で既に表示済みのため)。
- `header`: 「プロンプト確認」など短いラベル
- `options`(2つ。「その他」はツールが自動で追加するため明示しない):
  1. label: 「改良後のプロンプトを使用する」、description に書き換え後のプロンプトの要点を記載
  2. label: 「元のプロンプトを使用する」、description に「ユーザーの元の文言のまま進める」旨を記載

選択結果に応じて対応する。

- 「改良後のプロンプトを使用する」→ 書き換え後の内容で作業を開始する。
- 「元のプロンプトを使用する」→ ユーザーの元の文言どおりに作業を開始する。
- 「その他」(自由記述)→ ユーザーが入力した内容を新しいプロンプトとして採用し、それに基づいて作業を進める(再度書き換え提案を繰り返さない)。

いずれの選択でも、選択が完了した時点でこのスキルを再度発火させない(スキップ条件の「提示された選択肢からの選択」に該当する)。

## スキップ条件(発火させない/書き換えを行わないケース)

- はい/いいえだけの回答
- 事前に提示された選択肢からの選択(A/B、番号選択など)
- 「続けて」「それでお願いします」「OK」「そのまま進めて」など、追加情報を伴わない同意・継続の合図
- 直前の出力に対する修正指示・フィードバック(対象が直前の文脈から自明な場合)
- すでに対象・条件・数値・出力形式などが明記されている具体的な指示
- スラッシュコマンド(`/xxx`)の実行
- ログ・エラーメッセージ・コード・データを貼り付けて、その扱いが文脈上自明な場合
- 事実確認など、解釈の余地がほとんどない単純な質問

## 書き換え例

**例1**
- 元: 「このバグ直して」
- 書き換え案: 「直前に共有された [具体的なファイル名/エラーメッセージ] で発生している [具体的な症状] を修正してください。再現条件は [文脈から分かる範囲] です。」
  (対象が文脈から特定できない場合は断定せず、「対象のファイル/エラーメッセージを教えてください」と確認する形にする)

**例2**
- 元: 「グラフをいい感じにして」
- 書き換え案: 「[対象のグラフ] について、軸ラベルの追加・配色の統一・凡例の位置調整を行い、見やすさを改善してください。」

**例3(書き換え不要)**
- 「src/main.py の 42 行目の関数名を calculate_total から compute_total にリネームして」
  → すでに具体的なので書き換えずそのまま進める。

## 注意

- このスキルは毎回のユーザーメッセージに対して黙って発火判定を行うためのものであり、判定結果が「スキップ」であることをユーザーに説明する必要はない。
- 書き換え案は簡潔に。長い前置きは不要。
- 承認待ちのループを繰り返さない。一度提示して承認/修正を受け取ったら、その回で作業を進める。
