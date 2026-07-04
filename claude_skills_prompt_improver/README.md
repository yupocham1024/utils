# claude_skills_prompt_improver

Claude 用の Skill。ユーザーが送った曖昧・主観的・情報不足なプロンプトを、より具体的で客観的なプロンプトに書き換えてから作業を進めるようにする。[Claude Code](https://claude.com/claude-code)・[claude.ai](https://claude.ai) の両方で導入できる(挙動に一部差分あり。後述)。

## これは何をするか

新しいユーザーメッセージを受け取るたびに、Claude が以下を自動でチェックする。

1. プロンプトが曖昧・主観的・情報不足でないかを点検する
   - 例:「このバグ直して」「グラフをいい感じにして」など
2. 改善が必要なら、チャット本文に書き換え後のプロンプトを表示する
3. 続けて選択肢を提示し、ユーザーに選ばせる
   - 改良後のプロンプトを使用する
   - 元のプロンプトを使用する
   - その他(自由記述で修正案を入力)
4. 選ばれた内容で作業を開始する

以下のようなメッセージは、書き換え対象から自動的に除外される(スキップされる)。

- はい/いいえだけの回答
- 事前に提示された選択肢からの選択
- 「続けて」「それでお願いします」「OK」など、追加情報を伴わない相槌・承認
- 直前の出力への修正・フィードバック
- すでに具体的な指示(対象・数値・出力形式が明記されているもの)
- スラッシュコマンドの実行
- ログ・コード・エラーなど、扱いが文脈上自明な貼り付けデータ

詳細な判定ロジックは [`prompt-improver/SKILL.md`](prompt-improver/SKILL.md) を参照。

## インストール方法(Claude Code)

Claude Code の Skill は `~/.claude/skills/` 配下に置くと、ユーザーが使う **すべてのプロジェクトで** 有効になる(プロジェクト単位ではなくユーザー単位で有効化される)。

### 1. `prompt-improver` フォルダをコピーする

```bash
cp -r prompt-improver ~/.claude/skills/prompt-improver
```

このリポジトリを直接 clone している場合は、リポジトリを更新するたびに反映させたいことが多いので symlink でもよい。

```bash
ln -s "$(pwd)/prompt-improver" ~/.claude/skills/prompt-improver
```

### 2. Claude Code を再起動する(または新しいセッションを開始する)

Skill 一覧はセッション開始時に読み込まれるため、既存の Claude Code セッションを開いたままでは反映されない。新しいセッションを開始することで `prompt-improver` が Skill 一覧に表示されるようになる。

### 3. 動作確認

新しいセッションで、わざと曖昧なプロンプトを投げてみる。

```
このコードいい感じにして
```

書き換え案と選択肢のダイアログが表示されれば導入成功。

### アンインストール

```bash
rm -rf ~/.claude/skills/prompt-improver
```

(symlink で導入した場合は `rm ~/.claude/skills/prompt-improver` で symlink のみ削除される)

## インストール方法(claude.ai)

claude.ai では Skill を **ZIP ファイル** としてアップロードする。Claude Code とはアップロード先が別なので、Claude Code に導入済みでも別途アップロードが必要。

### 前提条件

- **コード実行(Code execution)機能が有効になっていること**。無効になっている場合は Settings 内でオンにする。
- Free / Pro / Max / Team / Enterprise のいずれのプランでも利用可能。

### 1. `prompt-improver` フォルダを ZIP 化する

`SKILL.md` がフォルダ直下(ルート)に来るように zip 化する。

```bash
cd claude_skills_prompt_improver
zip -r prompt-improver.zip prompt-improver
```

### 2. claude.ai にアップロードする

**Settings > Features > Skills** から、作成した `prompt-improver.zip` をアップロードする。
Team / Enterprise プランの管理者は組織全体に配布することもできる。

### 3. 動作確認

新しいチャットで、わざと曖昧なプロンプトを投げてみる(例:「このコードいい感じにして」)。書き換え案が提示されれば導入成功。

### claude.ai 特有の注意点(重要)

このスキルは書き換え確認の UI に Claude Code 専用の `AskUserQuestion`(選択式ダイアログ)ツールを使っている。**claude.ai のチャットにはこのツールに相当する機能がなく**、そのままでは選択肢をダイアログとして出すことができない。claude.ai 上では Claude が代わりに次のいずれかで代替する。

- 「1. 改良後のプロンプトを使用する / 2. 元のプロンプトを使用する / 3. その他」のようなテキストの番号選択として提示し、返信の番号やテキストをパースする
- あるいは通常の自由回答形式の確認に留める

Claude Code版と比べて対話の見た目・操作感は劣化する。claude.ai 専用に `AskUserQuestion` 部分をテキストベースの番号選択の指示に書き換えたバージョンが必要な場合は別途対応する。

## 注意事項

- Claude Code / claude.ai いずれの Skill も「Claude 自身がそのターンで発火させるべきか判断する」仕組みであり、ハーネスが強制的に全メッセージをフックするものではない。`SKILL.md` の `description` に強めのトリガー条件を書き込むことで発火率を高めているが、100% 毎回発火する保証はない。
- 発火が弱い、または過剰に発火してしまう場合は `prompt-improver/SKILL.md` の `description` やスキップ条件を調整する。
- Claude Code と claude.ai でカスタム Skill は自動的に同期されない。両方で使いたい場合はそれぞれに導入する必要がある。
