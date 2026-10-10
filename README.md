# 📈 Quantitative Trading Backtesting & Data Scraping Engine

<p align="left">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.10+-blue.svg" />
  <img alt="Pandas" src="https://img.shields.io/badge/Pandas-Data%20Processing-150458.svg?logo=pandas" />
  <img alt="Quant" src="https://img.shields.io/badge/Quantitative%20Finance-Backtesting-success.svg" />
  <img alt="License" src="https://img.shields.io/badge/License-MIT-green.svg" />
</p>

> **專案簡介 / Project Overview**
> 此專案為一套從零建構的**量化交易回測引擎與資料抓取管線**，結合統計套利（配對交易）與傳統技術指標（SMA 均線交叉、DCA 定期定額）。本專案具備完整的前後期（Walk-forward）時間驗證機制以避免前視偏誤（Look-ahead Bias），並產出豐富的交易訊號、權益曲線及績效指標（Sharpe Ratio, Max Drawdown 等），最終將數據匯出供互動式儀表板（Dashboard）呈現。
> 
> *註：本專案為研究所推甄/專題之展示作品，展現包含數據處理、金融建模、演算法設計及軟體工程之綜合能力。*

---

## 🌟 核心功能 (Key Features)

### 1. 穩健的資料管線 (Robust Data Pipeline)
- 透過 `yfinance` 自動抓取長達 30 年 (1996 - 2026) 的美股歷史資料。
- 支援**離線快取機制 (Offline-first Cache)**：預設讀取本機 `stock_prices.csv`，大幅提升回測執行效率。
- 資料自動清洗與正規化，處理遺漏值與拆股/配息之還原價格 (Adjusted Close)。

### 2. 多策略回測引擎 (Multi-Strategy Backtesting)
引擎內建多種經典交易策略，並針對指定股票池 (MCD, KO, AAPL, MSFT, ORCL) 與基準指數 (SPY) 進行精確運算：
- **買入持有 (Buy-and-Hold)**：計算基準報酬。
- **公平定期定額 (Fair DCA)**：模擬紀律投資，精確分配資金至各期交易日。
- **簡單移動平均線交叉 (SMA Crossover)**：內建 `SMA 20/60`, `SMA 50/200`, `SMA 100/300`，採用 `shift(1)` 避免前視偏誤。

### 3. 統計套利 / 配對交易 (Statistical Arbitrage - Pairs Trading)
- 基於共變異數與訓練期相關性 (Correlation Ranking) 自動篩選具共整合潛力的配對。
- 實作 **Z-Score 價差交易模型** (60 日滾動視窗)：
  - 進場門檻：$|Z| \ge 2.0$
  - 出場門檻：$|Z| < 0.5$
- 動態計算等額多空部位 (50%/50% Long-Short Market Neutral)。

### 4. 嚴格的模型驗證 (Temporal Validation)
- 採用**擴增視窗 (Expanding Window)** 的前向時間驗證法。
- 確保模型參數（如 Hedge Ratio）僅基於過去資訊訓練，嚴格杜絕資料窺探 (Data Snooping)。

---

## 🛠️ 技術堆疊 (Tech Stack)

- **核心語言**: Python
- **資料科學**: `pandas` (時間序列處理), `numpy` (數值運算), `statsmodels` (統計檢定)
- **資料來源**: `yfinance` (API 串接)
- **前端視覺化**: HTML5, Vanilla JS, CSS (讀取回測輸出之 CSV/JSON 以呈現 Dashboard)

---

## 📊 實驗設定與假設 (Methodology & Assumptions)

- **股票母體 (Universe)**: McDonald's (MCD), Coca-Cola (KO), Apple (AAPL), Microsoft (MSFT), Oracle (ORCL)
- **市場基準 (Benchmark)**: SPDR S&P 500 ETF (SPY)
- **初始資本 (Initial Capital)**: 每檔股票/策略 USD $10,000
- **交易假設**:
  - 允許購買零股 (Fractional Shares)。
  - 不計算交易手續費 (Zero Commission)、滑價 (Slippage) 及稅賦。
  - 所有交易皆以還原收盤價 (Adjusted Close) 執行。
- **訊號生成**:
  - SMA 與配對交易訊號皆使用前一交易日 $(t-1)$ 之計算結果，於 $(t)$ 日收盤執行，確保完全無前視偏誤。

---

## 🚀 快速啟動 (Quick Start)

### 1. 安裝環境與依賴
請確認系統已安裝 Python 3.9+。
```bash
# 複製儲存庫
git clone https://github.com/renkai09/Financial-Data-Scraping-and-DCA-Comparison.git
cd Financial-Data-Scraping-and-DCA-Comparison

# 安裝所需套件
pip install -r requirements.txt
```

### 2. 執行資料生成與回測
```bash
# 執行回測引擎 (預設使用本機快取資料以加速)
python generate_data.py

# 若需從 Yahoo Finance 強制重新下載最新資料，請加上 --refresh 參數：
python generate_data.py --refresh
```
> 執行完畢後，所有回測結果將輸出至 `data/` 資料夾，包含交易明細 (`strategy_signals.csv`)、每日權益變化 (`equity_curves.csv`)、績效統整 (`strategy_performance.csv`) 等。

### 3. 啟動互動式儀表板 (Dashboard)
本專案自帶輕量化前端 Dashboard 視覺化回測數據：
```bash
# 啟動本地端伺服器
python -m http.server 8000
```
接著請開啟瀏覽器訪問 👉 [http://localhost:8000](http://localhost:8000)

---

## 📂 專案結構 (Project Structure)

```text
Financial-Data-Scraping-and-DCA-Comparison/
├── generate_data.py       # 核心回測引擎與資料處理腳本
├── requirements.txt       # Python 相依套件清單
├── README.md              # 專案說明文件
├── data/                  # 包含生成的價格資料、回測結果 (CSV/JSON)
├── report.docx            # 專案報告與深度分析
└── slide.pdf              # 專案簡報 (可用於面試/推甄展示)
```

---

## ⚠️ 免責聲明 (Disclaimer)

本專案及相關程式碼僅供**學術研究與教育目的**使用。專案內回測結果基於過去歷史數據與特定假設，**不代表未來投資績效**，亦不構成任何投資建議。實務交易中需考量滑價、流動性與手續費等摩擦成本。
