# claude_skills_level_aware_explainer

Claude 用の Skill。AI が出力する専門用語・難しい概念が理解の壁にならないよう、ユーザーの既知範囲を基準に、その場で平易な注釈を添えるようにする。[Claude Code](https://claude.com/claude-code)・[claude.ai](https://claude.ai) の両方で導入できる。

## これは何をするか

会話の中で技術的・数学的な説明をするたびに、Claude が以下を自動でチェックする。

1. 説明の内容が、あらかじめ登録した「既知範囲」(WordPress・PHP・Linuxの日常操作、SQLの基本操作+JOIN・サブクエリ、高校数学)に収まっているかを判定する
2. 既知範囲を超える場合、専門用語はそのまま残しつつ、出てくるたびに簡潔な注釈を `専門用語(平易な言い換え)` の形で添える
3. 大学教養レベルの数学(線形代数・微分方程式・基礎的な確率統計)は「履修済みだが忘れている」前提で軽い再確認レベルの注釈にとどめ、専門レベルの数学(抽象代数・位相空間論・測度論など)はフルの解説を添える

以下のようなメッセージ・説明は、平易化の対象から自動的に除外される(スキップされる)。

- 説明が既知範囲(基本操作・JOIN/サブクエリ・高校数学・PHP/WordPress/Linuxの日常操作)に完全に収まっている場合
- 技術・数学的な説明を伴わない雑談、要件確認、進捗報告
- 同一回答内で、直前にすでに平易化した用語をそのまま再言及する場合

判定の詳細な基準(既知/未知の境界線、注釈フォーマット、数学の2段階の扱い)は [`level-aware-explainer/SKILL.md`](level-aware-explainer/SKILL.md) を参照。

## 対象ユーザー像を自分向けに調整する

このスキルは特定の1人(エンジニア歴3年、WordPress/PHP/Linux/SQL基本操作を扱える、数学は高校レベルは既知・大学教養レベルはうろ覚え)を前提に基準を作っている。自分の知識レベルに合わせて使いたい場合は `level-aware-explainer/SKILL.md` の「対象ユーザー像(既知範囲の基準)」と「発火判定」のセクションを書き換えてから導入する。

## インストール方法(Claude Code)

Claude Code の Skill は `~/.claude/skills/` 配下に置くと、ユーザーが使う **すべてのプロジェクトで** 有効になる(プロジェクト単位ではなくユーザー単位で有効化される)。

### 1. `level-aware-explainer` フォルダをコピーする

```bash
cp -r level-aware-explainer ~/.claude/skills/level-aware-explainer
```

このリポジトリを直接 clone している場合は、リポジトリを更新するたびに反映させたいことが多いので symlink でもよい。

```bash
ln -s "$(pwd)/level-aware-explainer" ~/.claude/skills/level-aware-explainer
```

### 2. Claude Code を再起動する(または新しいセッションを開始する)

Skill 一覧はセッション開始時に読み込まれるため、既存の Claude Code セッションを開いたままでは反映されない。新しいセッションを開始することで `level-aware-explainer` が Skill 一覧に表示されるようになる。

### 3. 動作確認

新しいセッションで、既知範囲外の話題を含む質問を投げてみる。

```
このSQL、ウィンドウ関数を使って書き換えられますか?
```

回答中の専門用語に `(平易な言い換え)` の注釈がつけば導入成功。逆に、基本操作の範囲内の質問(例:「このテーブルにSELECT文を書いて」)では注釈がつかないことも確認するとよい。

### アンインストール

```bash
rm -rf ~/.claude/skills/level-aware-explainer
```

(symlink で導入した場合は `rm ~/.claude/skills/level-aware-explainer` で symlink のみ削除される)

## インストール方法(claude.ai)

claude.ai では Skill を **ZIP ファイル** としてアップロードする。Claude Code とはアップロード先が別なので、Claude Code に導入済みでも別途アップロードが必要。

### 前提条件

- **コード実行(Code execution)機能が有効になっていること**。無効になっている場合は Settings 内でオンにする。
- Free / Pro / Max / Team / Enterprise のいずれのプランでも利用可能。

### 1. `level-aware-explainer` フォルダを ZIP 化する

`SKILL.md` がフォルダ直下(ルート)に来るように zip 化する。

```bash
cd claude_skills_level_aware_explainer
zip -r level-aware-explainer.zip level-aware-explainer
```

### 2. claude.ai にアップロードする

**Settings > Features > Skills** から、作成した `level-aware-explainer.zip` をアップロードする。
Team / Enterprise プランの管理者は組織全体に配布することもできる。

### 3. 動作確認

新しいチャットで、既知範囲外の話題を含む質問を投げてみる(例:「Dockerコンテナの仕組みを教えて」)。回答中の専門用語に平易な注釈がつけば導入成功。

## 注意事項

- Claude Code / claude.ai いずれの Skill も「Claude 自身がそのターンで発火させるべきか判断する」仕組みであり、ハーネスが強制的に全メッセージをフックするものではない。`SKILL.md` の `description` に既知/未知の境界を具体的に書き込むことで発火率を高めているが、100% 毎回発火する保証はない
- 発火が弱い、または既知範囲内の話題にまで過剰に発火してしまう場合は `level-aware-explainer/SKILL.md` の「対象ユーザー像」「発火判定」を調整する
- Claude Code と claude.ai でカスタム Skill は自動的に同期されない。両方で使いたい場合はそれぞれに導入する必要がある
