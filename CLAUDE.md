# CLAUDE.md (YoYoEA_Multi_Entry)

## Claude への指示
- 会話は日本語で行う。
- コード変更・調査を実施したら、実施内容・結果・次アクションを
  `Memo/memo.txt` へ ISO8601 タイムスタンプ付きで追記する。
  書き込みは必ず UTF-8（BOMなし）で行う。
- 雑談はメモ不要。
- コード改良がまとまったらコミット案（件名＋概要）を提案する。

## プロジェクト構成
- EA本体: `MQL4/Experts/YoYoEA_Multi_Entry.mq4`
- ATRバンドConfig: `Config/`（`{PROFILE}` は `InpProfileName` と連動）
- テスターPreset: `presets/`
- 作業ログ: `Memo/memo.txt`

## コーディング規約
- インデント3スペース、関数CamelCase、構造体PascalCase、列挙体は大文字スネーク。
- 入力パラメータは `Inp*`、グローバル変数は `g_*`、定数は `k*` 接頭辞。
- セクション区切りは `//+------------------------------------------------------------------+` 形式。
- 出力ファイル: `TradeLog_<Profile>.csv`, `AtrBandConfig_<Profile>.csv`。

## コミット方針
- 件名に実装内容とバージョンを明示（例: `fix: MACD EMPTY_VALUE未検査を修正`）。
- 変更の背景・影響範囲が分かりづらい場合は本文に補足する。
