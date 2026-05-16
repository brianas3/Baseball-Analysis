# Baseball Data Science — Final Project

## Project Structure

```
baseball_project/
├── bb_pipeline/                        # Statcast season panel, Lahman positions, projection ML
├── scripts/
│   ├── build_pipeline_outputs.py       # CLI: rebuild batting_by_season + projection_panel
│   ├── create_presentation_figures.py  # 產出 5 張海報圖 → figures/poster/
│   └── build_word_report.py            # 產出 Word 報告 → docs/final_report.docx
├── notebooks/
│   ├── 01_data_collection.ipynb   # Statcast-only → batting_by_season + projection_panel
│   ├── 02_eda.ipynb               # EDA on season panel
│   ├── 03_analysis.ipynb          # Main: next-year wRC+ (RF / XGBoost); K-Means / summary
│   └── 04_visualization.ipynb     # Supporting figures (projection plots from 03)
├── data/
│   ├── raw/                       # batting_by_season_2018_2024.parquet（statcast_*.parquet 因體積過大不上傳）
│   └── processed/                 # batting_clean, projection_panel.parquet, …
├── figures/
│   ├── poster/                    # 海報專用 5 張高解析圖（fig_pipeline, fig_timesplit, …）
│   └── *.png                      # EDA 與模型輔助圖
├── docs/
│   ├── final_report.md            # 中文期末報告（Markdown）
│   ├── final_report.docx          # 中文期末報告（Word，含圖）
│   └── methodology.md             # 方法論：資料、模型、圖表解讀
└── README.md
```

完整研究方法與圖表說明見 **`docs/methodology.md`**（章節編號版）。

## Environment Setup

```bash
conda activate baseball   # or your venv
pip install -r requirements.txt
```

**macOS + XGBoost:** If `xgboost` fails to load (`libomp.dylib`), install OpenMP: `brew install libomp`, then reinstall xgboost. Random Forest runs without it.

## Execution Order

1. **`01_data_collection.ipynb`** — Loads/downloads `statcast_2018`…`2024` parquet, builds **`batting_by_season_2018_2024.parquet`**, then **`data/processed/projection_panel.parquet`** (lag features → target-season wRC+).  
   Or run: `python scripts/build_pipeline_outputs.py` from `baseball_project/` (requires raw Statcast files).

2. **`02_eda.ipynb`** — Explores the **season panel** (distributions, PA filters).

3. **`03_analysis.ipynb`** — **Primary:** time-based train/val/test for **next-year wRC+** (seasons 2021–2024 targets); RF permutation importance and scatter plots → `figures/projection_*.png`. **Supporting:** ranking comparison, K-Means on `batting_scaled.csv`.

4. **`04_visualization.ipynb`** — Legacy/cluster visuals; projection figures are produced in **03**.

## Data Sources

| Source | Usage |
|--------|--------|
| **Statcast** (`pybaseball.statcast`) | Only source for **batting rates**, PA, **BIP%**, **wRC+** (per-season league wOBA). |
| **Lahman** (via `pybaseball.lahman`, Chadwick register) | **Primary position** by season when zip download succeeds; else `UNK`. |

FanGraphs is **not** used for the batting pipeline.

## Key Dependencies

| Package | Notes |
|---------|--------|
| pybaseball | Statcast + Lahman/Chadwick |
| pandas, pyarrow | Parquet I/O |
| scikit-learn | RF, preprocessing, permutation importance |
| xgboost | Optional (needs OpenMP on some Macs) |
