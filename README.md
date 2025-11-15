# YoYoEA_Multi_Entry
YoYoEA_Multi_Entry（旧 YoYoEntryTester）は複数ストラテジー（MA、RSI、CCI、MACD、Stochastic）のエントリー精度を検証するための MQL4 エキスパートアドバイザーです。各ストラテジーの有効／無効やリスク制御パラメータを柔軟に設定し、バックテストやフォワードテストで挙動を記録します。

## 主な機能
- 複数戦略の同時評価：5 種類のテクニカル指標シグナルを個別に監視して注文を発行
- ATR バンド設定：CSV からストラテジー別の停止・利確条件を読み込み、ATR・pips のいずれかで管理
- エントリー制御強化
  - スプレッド上限（`InpMaxSpreadPips`）の超過で発注をスキップ
  - ATR 値が取得できない場合は自動で pips ベースにフォールバック
  - リスク一定ロット（`InpUseRiskBasedLots`）によるポジションサイズ調整
  - ブローカー制約に合わせたロット正規化と証拠金チェック
- マルチポジション・ロット制御（v1.23 以降）
  - `InpMaxPositionsPerStrategy` で各ストラテジーの保有上限を設定し、`InpAllowOppositePositions` と併用して総ポジション数を管理
  - エクイティ閾値（`InpMultiPositionEquityThreshold`）を下回ると自動的にマルチポジションを停止
  - 追加の閾値（`InpLotReductionEquityThreshold`, `InpLotReductionFactor`）で資産減少時のロット縮小に対応
- 稼働ガード
  - 戦略ごとのクールダウン／連敗休止 (`InpCooldownMinutes`, `InpLossStreakPause`, `InpLossPauseMinutes`)
  - 既存ポジションと逆方向のシグナルを抑制 (`InpAllowOppositePositions`)
  - 取引時間帯フィルタと金曜カットオフ (`InpUseTradingSessions`, `InpSessionStartHour`, `InpSessionEndHour`, `InpSessionSkipFriday`, `InpFridayCutoffHour`)
- 詳細ログ出力：エントリー／決済を CSV に記録し、パラメータスナップショットや ATR バンド適用状況をコンソールに出力

## 入力パラメータの補足
- `InpUseRiskBasedLots = true` の場合、損切り距離（ATR／pips）から口座残高に対するリスク％（`InpRiskPercent`）でロットを算出します。
- `InpLossStreakPause` に達した損失連続数で休止し、`InpLossPauseMinutes` が 0 の場合は再開条件を外部で整えるまで停止します。
- 取引セッションはサーバー時間を基準とし、`InpSessionStartHour` ≤ 時刻 < `InpSessionEndHour` の間のみ発注します。金曜は `InpSessionSkipFriday = true` と `InpFridayCutoffHour` で早期停止が可能です。

## ログファイル
- トレードログ：`TradeLog_<Profile>.csv` – ENTRY/EXIT 双方を記録。価格やインジケータ値に加え、`atr_entry` / `atr_exit` 列でそれぞれの ATR を保持し、損益（`net`）、獲得 pips、`exit_reason`（`TAKE_PROFIT` / `STOP_BREAKEVEN` / `STOP_TRAILING` / `STOP_LOSS` / `MANUAL_*`）を追跡

## ビルドと配置
1. PowerShell で `Scripts/build_experts.ps1` を実行すると、MQL4/Experts 配下の EA をまとめてコンパイルできます。
   ```powershell
   pwsh ./Scripts/build_experts.ps1
   ```
2. 生成された `.ex4` を MT4 の `Experts` ディレクトリへ配置し、テスターでパラメータを調整してバックテストを実施してください。

## テストと運用のポイント
- 新規パラメータを組み合わせたバックテストで、スプレッド制御・連敗休止・セッション抑制が期待通りログに出力されるか確認してください。
- 取引サーバーのタイムゾーンと `InpSession*` の設定が一致しているか事前に確認してください。
- ATR バンド CSV の設定が不足している場合、デフォルト値にフォールバックします。テスト開始時のログで読み込み結果をチェックすることをおすすめします。

### 現在の標準設定（Prod）
- **設定ファイル**: `YoYoEA_Multi_Entry/Config/AtrBandConfig_YoYo_Next_v124_Prod.csv`
  - 低ATR帯（0.05-0.10）の Stoch は BE=1.0/1.3ATR & Trail=0.30/0.40ATR で安定寄りに調整。
  - CCI は 0.12 以上の帯域のみで稼働し、0.12-0.20 は SL=3.3ATR / TP=5.9ATR / Trail step 0.90。
  - MA/RSI/MACD は v4b 相当のバランス設定で、Prod ログでは合計 +2,115 USD（2025-01 期間）を記録。
- **テストログ**
  - `TradeLog_v124/AtrBandConfig_YoYo_Next_v124_Prod.csv.csv`: 2025-01-01 検証（基準期）。
  - `TradeLog_v124/AtrBandConfig_YoYo_Next_v124_Prod_202506.csv`: 2025-06 期間の比較ログ。短期サンプルとして利用。
- 上記ログの集計は `../shared/analysis/atr_indicator_band_summary_YYYYMM_AtrBandConfig_YoYo_Next_v124_Prod.csv` に保存しています。異なる期間を比較する場合は同ファイルを追加作成し、Notion「YoYoEA_Multi_Entry 帯域×指標サマリ」に必要な行だけ登録してください。

### BT/運用手順のテンプレート
1. `AtrBandConfig_YoYo_Next_v124_Prod.csv` を `MQL4/Files` へコピーし、Strategy Tester の `InpAtrBandConfigFile` で参照。
2. テスト完了後は `TradeLog_<Profile>.csv` を `../shared/analysis/TradeLog_v124/` などに保管し、`../shared/analysis/atr_indicator_band_summary_<期間>_...csv` を生成。
3. 結果を Notion（データベース: *YoYoEA_Multi_Entry 帯域×指標サマリ*）へ追記し、主要ハイライトを `Memo/memo.txt` にISO時刻で記録。

## ATRバンドCSVフォーマット
- CSV は『minAtr,maxAtr,<Strategy>_Enable,<Strategy>_Mode,<Strategy>_SL,<Strategy>_TP』の列を戦略ごとに持ちます。
- デフォルトの列順: MA / RSI / CCI / MACD / STOCH。見出し名が一致すれば順番入れ替えや一部省略も可能。
- Mode は GLOBAL/ATR/PIPS を指定し、SL/TP は ATR 倍率または pips 値を入力します。
- Enable を OFF にすると該当ストラテジーは該当帯域でシグナルを無効化します。
- サンプル: minAtr,maxAtr,MA_Enable,MA_Mode,... を参照し、必要に応じて帯域を追加してください。

## v1.23 で追加された主な仕様
- Multi-Position サポート
  - `InpMaxPositionsPerStrategy` と有効ストラテジー数から総保有上限を算出し、`InpAllowOppositePositions=true` 時に複数ポジションを許可
  - `InpMultiPositionEquityThreshold` を下回ると自動的にシングルポジション運用へ切り替え
- ロット縮小ロジック
  - `InpLotReductionEquityThreshold` で指定したエクイティを割ると、固定ロット運用時に `InpLotReductionFactor` を乗じたロットで発注
- ログ拡張
- TradeLog に `atr_entry` / `atr_exit` 列を追加し、ENTRY/EXIT 両方の ATR 状態を記録。ENTRY ログは廃止済み
- 互換性
  - 新パラメータはすべて任意で、既存設定（シングルポジション・固定ロット）のままでも過去バージョンと同じ動作を維持します。

## 機械学習ベースのConfig最適化
- 市場状態ログの収集方法、ETL パイプライン、XGBoost による Config 推薦フローを `docs/ml_config_pipeline.md` にまとめています。
- 特徴量カテゴリ（市場状態／EA状態／Configメタデータ）や、StateLogger EA・SignalLog 拡張の仕様も同ファイルを参照してください。

## 今後の仕様拡張: ADX+Donchian フィルタ

### 目標仕様
- ATR バンド設定にトレンド強弱（ADX）を取り込む。
  - Config CSV に `ADX_STATE` 列を追加し、ATR 帯ごとに `HIGH` (トレンド) / `LOW` (レンジ) の 2 設定を用意。
  - EA はエントリー時に `entryAdxValue` を取得し、ATR 帯と ADX_STATE が一致する行のみ適用。
- Donchian 幅（narrow/mid/wide/ultra）も条件に使えるようにし、帯域によってストラテジーの有効/無効や SL/BE/Trail 値を切り替え。
- TradeLog に `adx_entry` や Donchian 幅などの列を追加して分析と連携。

### 実装計画
1. **設定構造の拡張**
   - `AtrBandConfig` のヘッダに `ADX_STATE` と必要なら `DONCHIAN_STATE` を追加。後方互換のため未記入時は `LOW` / `ANY` と扱う。
   - `StrategyBandSetting` に `adxState` / `donchianState` を追加し、既存の SL/TP/BE/Trail と併せて保持。
2. **ADX 判定ロジック**
   - `ResolveBandSetting()` へ `entryAdxValue` を渡し、行の `adxState` と `InpAdxThresholdHigh` を基に採用可否を決定。
   - ADX 閾値 (`InpAdxPeriod`, `InpAdxThresholdHigh`) を extern パラメータとして追加。
3. **Donchian 幅判定**
   - ATR ゾーン確定後に `donchian_width` 用の関数を呼び、CSV の `DONCHIAN_STATE` が `NARROW` / `WIDE` などで指定されていればフィルタ。
4. **ログ／メタデータ拡張**
   - `TradeMetadata` と `TradeLog` に ADX / Donchian 情報を保持し、後段の Notebook 解析と一致させる。
5. **テスト計画**
   - 旧 Config でも動作することを確認したうえで、新しい ADX_STATE を使った設定ファイルを生成。
   - バックテストログを Notebook で集計し、ATR×ADX フィルタ適用の効果を検証。

