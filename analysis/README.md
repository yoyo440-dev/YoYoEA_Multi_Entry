# Analysis Workspace

- `env_data/`: 市場環境（StateLog 等）の CSV をここにコピーまたはシンボリックリンクで配置します。
- `bt_results/`: TradeLog / TradeParams など BT 結果を格納します。
- `notebooks/`: 解析用 Jupyter Notebook を配置します。

## 推奨フロー
1. MT4 側で収集した `StateLog_YYYYMMDD.csv` を `analysis/env_data/` に置く。
2. 対応する `TradeLog_*.csv` などを `analysis/bt_results/` に置く。
3. `notebooks/` 内のノートブックで `env_data` と `bt_results` を読み込み、条件集計や突合せを実施。

`analysis/notebooks` は初回実行時に自動生成され、テンプレートノート `state_bt_analysis.ipynb` から解析を始められます。
