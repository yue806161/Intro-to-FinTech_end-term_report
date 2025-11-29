# 金融科技導論 期末報告

基於機器學習之個股價格驅動因子解析：台積電三大面向模型預測誤差比較研究

Project Status: In Progress  
Author: [符秉義、何彥霖]  
Target Stock: 台積電 (2330)  
Data Source: FinLab Database  

## 專案概述 (Project Overview)

本專案旨在解決金融市場預測中「資訊雜訊」過大的問題。不同於傳統方法將所有變數混入單一模型，本研究將影響股價的因子拆解為 技術面 (Technical)、基本面 (Fundamental) 與 籌碼面 (Chip) 三大獨立面向。透過建立四組平行的 LSTM (Long Short-Term Memory) 預測模型，我們不只關注「預測準確度」，更聚焦於 **「模型誤差分析 (Error Analysis)**。藉由比較不同模型在特定時間點的預測誤差 (Gap)，試圖解析出台積電股價在不同時期是由何種力量（如：外資流向、營收財報或市場情緒）所主導。

## 研究動機與目的 (Motivation & Objectives)

### 研究動機去除雜訊

全變數模型容易過度擬合 (Overfitting)，透過分組訓練可釐清訊號來源。

- 動態歸因：股價驅動因子具有時變性 (Time-varying)。例如：法說會前後可能由基本面主導，而系統性風險發生時可能由籌碼面主導。
- 解釋性 AI：從單純的「預測價格」轉向「解釋市場行為」。

### 專案目標建構競賽模型

開發技術、基本、籌碼及混合式四組獨立預測模型。

- 資料對齊技術：解決月/季報與日交易資料的頻率不一致問題，嚴格防止未來數據洩漏。
- 誤差歸因分析：透過計算 MAPE (平均絕對百分比誤差)，找出不同市場情境下的「最佳解釋面向」。

## 資料來源與變數設計 (Data & Features)

本專案使用 finlab API 獲取台積電歷史數據，並將特徵分為以下四組：

|特徵集 (Set)|涵蓋變數 (Features)|資料特性|
|---|---|---|
|Set A|技術面開高低收 (Adj Close, OHLC)、成交量、MA(5,20,60)、RSI、KD、MACD|反應市場趨勢與情緒，更新最即時。|
|Set B|基本面月營收 (MoM, YoY)、季 EPS、毛利率、本益比 (PE)、殖利率|反應公司真實價值，需透過 Forward-Fill 對齊至日資料。|
|Set C| 籌碼面三大法人買賣超 (外資/投信/自營)、融資融券餘額、大戶持股比例|反應市場資金流向與主力動態。|
|Set D|混合式包含上述 Set A + Set B + Set C 所有變數|用於測試全變數是否優於單一面向。|

## 研究方法與實驗設計 (Methodology)

### 實驗流程

#### Data Ingestion：串接 FinLab，抓取 2018-2023 年資料  

#### Preprocessing

- Alignment: 將月/季資料向後填充 (Forward Fill) 至每日。
- Normalization: 使用 `MinMaxScaler` 將數值縮放至 [0, 1]。  

#### Model Training

- Algorithm: LSTM (Sequence Length = 10 days)  

- Training Period: 2018 - 2021 (多頭與盤整)  

- Testing Period: 2022 - 2023 (包含空頭修正與反彈)  

#### Evaluation: 計算每日 MAPE，產出誤差走勢圖  

### 歸因邏輯 (Attribution Logic)

針對每一天 $t$，比較四個模型的預測誤差 $E$：$$Winner_t = \arg\min (E_{tech}, E_{fund}, E_{chip})$$若 $E_{chip}$ 最小 $\rightarrow$ 判定當日為 籌碼面主導。若 $E_{fund}$ 最小 $\rightarrow$ 判定當日為 基本面主導。

## 專案結構 (Directory Structure)

``` text
TSMC-Price-Driver-Analysis/
├── data/                   # 存放原始與處理後的 csv 檔
│   ├── raw_data.csv
│   ├── train_data.csv
│   ├── test_data.csv
│   └── processed_data.csv
├── notebooks/              # Jupyter Notebooks (實驗區)
│   ├── 01_Data_Loader.ipynb    # 資料抓取與清洗
│   ├── 02_Feature_Eng.ipynb    # 特徵工程與對齊
│   ├── 03_Model_Train.ipynb    # LSTM 模型訓練
│   └── 04_Analysis.ipynb       # 誤差分析與視覺化
├── src/                    # 核心程式碼 (模組化)
│   ├── config.py           # 參數設定
│   ├── data_loader.py      # 資料處理函式
│   └── model.py            # LSTM 模型架構
├── results/                # 輸出結果 (圖表、模型權重)
│   ├── error_heatmap.png
│   └── comparison_plot.png
├── README.md               # 專案說明文件
└── requirements.txt        # Python 套件需求
```

## 預期成果 (Expected Results)

- 誤差比較圖：展示技術、基本、籌碼模型在 2022-2023 年間的預測準度消長。

- 公司性質歸類：統計台積電在測試期間，受「外資籌碼」影響的天數比例是否顯著高於「技術指標」。
- 關鍵事件分析：驗證在法說會或除權息前後，基本面模型的誤差是否顯著降低。

## 技術棧 (Tech Stack)

Language: Python 3.14+  
Data Source: FinLab API  
Machine Learning: TensorFlow / Keras (LSTM), Scikit-learn  
Data Processing: Pandas, NumPy  
Visualization: Matplotlib, Seaborn  
