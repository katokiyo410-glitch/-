# このリポジトリについて

静的な英語学習サイト（GitHub Pagesでホスト、`main`ブランチへのpushで自動デプロイ）。

- `index.html` — 英検3級対策クイズアプリ（既存・手動アップロード分）
- `vocab.html` — 英単語・フレーズ スタディアプリ（下記の仕組みで自動生成、単語/フレーズをタブで切り替え）
- `vocabulary.json` — 日々ためていく英単語リスト（データの元）
- `phrases.json` — 日々ためていくフレーズリスト（データの元）

## 単語帳スタディアプリの仕組み

### 1. 単語・フレーズを追加する（毎日）

ユーザーがチャットで伝えたら、対応するJSONファイルに以下の形式で追記してコミットする。
まだ日本語訳・例文は付けない（トリガー発動時にまとめて生成するため、二重管理を避ける）。

- 単語（例:「今日の単語: ambitious, thrive」）→ `vocabulary.json`
  ```json
  { "word": "ambitious", "dateAdded": "YYYY-MM-DD" }
  ```
- フレーズ（例:「今日のフレーズ: kick the bucket」）→ `phrases.json`
  ```json
  { "phrase": "kick the bucket", "dateAdded": "YYYY-MM-DD" }
  ```

同じ単語/フレーズが既にあれば追加しない（重複防止）。

### 2. トリガーが発動したら（＝勉強アプリを作る/更新する）

ユーザーが「単語帳アプリ更新して」「スタディアプリ作って」のように頼んだとき、
または紐づけられたRoutine（CCRのトリガー）が発火したときは、次を行う:

1. `vocabulary.json` と `phrases.json` の中身を全て読む。
2. 各単語・各フレーズについて、以下をClaude自身の知識で生成する:
   - 日本語訳（簡潔に。単語は必要なら品詞も）
   - ネイティブが実際によく使う自然な例文（英語）— フレーズの場合はそのフレーズを使った文
   - その例文の日本語訳
3. `vocab.html` 内の埋め込みデータ配列 `ALL_WORDS`（単語用、`{word, meaning, exampleEn, exampleJa}`）と
   `ALL_PHRASES`（フレーズ用、`{phrase, meaning, exampleEn, exampleJa}`）をこの内容で丸ごと置き換えて再生成する。
   - デザイン・単語/フレーズ切り替えタブ・発音ボタン（🔊、Web Speech APIの`speechSynthesis`、
     `speak`/`speakWord`/`speakExample`/`speakQuiz`関数）など、データ配列以外の仕組みは壊さないこと。
   - 単語・フレーズそれぞれ、0件なら空状態、4件未満なら4択クイズを無効のままにする
     （`CATEGORY_INFO`と`currentItems()`まわりのロジックを前提に、配列の中身だけ差し替えればよい）。
4. 変更をコミットし、現在の作業ブランチにpushする。
   - コミットメッセージ例: `Rebuild vocab study app (N words, M phrases)`
5. `main` にマージ/PR作成しない限りGitHub Pagesには反映されない旨を、作業後に一言添える。

## 注意

- `vocabulary.json` / `phrases.json` は単語・フレーズと追加日だけを持つ「ソースオブトゥルース」。
  訳や例文はここには保存せず、トリガー発動のたびに再生成する（内容の陳腐化を防ぐため）。
