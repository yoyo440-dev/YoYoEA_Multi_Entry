# YoYoEA_Multi_Entry Runbook

1. `Config/` で対象 `AtrBandConfig_<Profile>.csv` を選定し、必要なら `archive/` から複製して編集します。
2. `presets/` の `.set` を Strategy Tester へ読み込み、`InpProfileName` と `InpAtrBandConfigFile` を更新。
3. `pwsh ./Scripts/compile_YoYoEA_Multi_Entry.ps1` で最新 EX4 を生成し、MT4 へ配置。
4. テスト完了後は `TradeLog_<Profile>.csv` と補助ログを `../shared/analysis/<YYYYMMDD>/` へ整理し、`Memo/memo.txt` に結果を記入。
5. 追加分析は `shared/analysis` 内のサマリ CSV (`atr_indicator_band_summary_*.csv`) を更新し、Notion へ反映。
