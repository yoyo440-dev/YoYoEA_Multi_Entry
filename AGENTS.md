# Repository Guidelines (YoYoEA_Multi_Entry)

## CODEX
- 会話は日本語でお願いします。
- 実施したアクションは ISO8601 タイムスタンプ付きで `YoYoEA_Multi_Entry/Memo/memo.txt` に追記してください。アクション内容と結果、次アクションを簡潔かつ具体的に記録します。
- コード開発以外の雑談はメモ不要です。
- コード改良がまとまったら、コミット案（件名+概要）もセットで提案してください。

## プロジェクト構成
- EA 本体: `MQL4/Experts/YoYoEA_Multi_Entry.mq4`
- Config: `Config/`（ATR バンド CSV、`archive/` へ旧設定保管）。`{PROFILE}` は `InpProfileName` と連動。
- テスター Preset: `presets/`（BETR_* 命名）。
- スクリプト: `Scripts/`（自動コンパイル `compile_YoYoEA_Multi_Entry.ps1`、端末起動 `run_Rakuten_MT4.ps1`）。
- 成果物: `shared/Compile_log` および `shared/analysis` 配下へ保存。分析 CSV は `atr_indicator_band_summary_<期間>_...csv` 命名で管理します。
- 作業ログ: `Memo/memo.txt` に一本化し、常に最新状態を維持。

## ビルド・テスト
- 一括コンパイル: `pwsh ./Scripts/compile_YoYoEA_Multi_Entry.ps1`（MetaEditor path は `D:/Rakuten_MT4` 想定、必要に応じて書き換え）。
- EX4 配置先: Windows 側 `D:\Rakuten_MT4\MQL4\Experts`。
- テスター起動は `pwsh ./Scripts/run_Rakuten_MT4.ps1` で MT4 を `/portable` モード起動。
- 個別コンパイル: `metaeditor.exe /portable /compile:"<path>\YoYoEA_Multi_Entry.mq4" /log:"<log>"` を直接呼び出しても可。

## コーディング規約
- インデント 3 スペース、関数 CamelCase、構造体 PascalCase、列挙体は大文字スネーク。
- extern 入力は `Inp*`、グローバル状態は `g_*`、定数は `k*` 接頭辞。
- ファイル上部の `#property` と extern 入力順序は既存構成を踏襲。
- セクション区切りコメントは `//+------------------------------------------------------------------+` 形式を使用。
- 出力ファイル命名: `TradeLog_<Profile>.csv`, `AtrBandConfig_<Profile>.csv`。

## テスト指針
- Strategy Tester (Ctrl+R) で `presets/` の基準セットを読み込み、`InpUseAtrBandConfig` が true の場合は `MQL4/Files` に対象 CSV を配置して `InpAtrBandConfigFile` に指定。
- テスト後は `TradeLog_<Profile>.csv` を `shared/analysis/` 下へ整理し、損益ハイライトを `Memo/memo.txt` に記録。
- スプレッド制御、マルチポジション制限、トレーリング/BE 更新がログ通り動作するか必ず確認。

## コミット/PR
- コミット件名は実装内容とバージョンを明示（例: `v1.24 マルチポジション調整`, `Refine ATR band loader`）。
- 変更点が分かりづらい場合は本文に背景・影響範囲・テスト結果を記述。
- PR には調整パラメータ、影響する Config/Preset、使用 MT4 build、主要バックテスト結果を添付し、`Memo/memo.txt` を更新してからレビュー依頼してください。
