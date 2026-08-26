# Skills（このセッションで作成したClaudeスキルのバックアップ）

セッション/PC消失に備え、`~/.claude/skills` に作成したカスタムスキルをGitに保存したもの。
Claude Code等で使うには、各フォルダを `~/.claude/skills/<name>/` に配置する（またはパッケージ化して読み込む）。

## 収録スキル

### lp-builder
参考サイト/画像をもとに、日本語のB2B/サービス系ランディングページを単一HTMLで作るスキル。
- `SKILL.md` … 発動条件・ワークフロー・参照ガイド
- `references/build-guide.md` … デザイントークン＋共通部品の統合スキャフォールド
- `references/hero-patterns.md` … ヒーロー構図A〜D（切り抜き人物×背景色／極太数値／写真カード／背景動画）
- `references/image-pipeline.md` … フリー素材取得・背景除去(rembg)・ヘッドレス検証
- `references/decorative-motifs.md` … 装飾モチーフ7種の活用ガイド

### carbonated-kv
炭酸飲料ブランド風の「シュワシュワ」WebGL製KV（液体屈折＋泡）を単一HTMLで作るスキル。
- `SKILL.md` … 進め方・守るべき原則
- `assets/kv-prompt.md` … 実装プロンプト全文

## 関連する成果物（このリポジトリ内）
- `../index.html` … CareShift LP（採用支援サービス風・オレンジ/切り抜きヒーロー/No.1）
- `../bside-saga/index.html` … B-SIDE SAGA 旅LP（背景動画ヒーロー＋動画差し込み）
