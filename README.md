# 金融科技導論 期末報告

基於機器學習之個股價格驅動因子解析：台積電三大面向模型預測誤差比較研究

Project Status: In Progress  
Author: [符秉義、何彥霖]  
Target Stock: 台積電 (2330)  
Data Source: FinLab Database  

## 專案概述 (Project Overview)

本專案並非為了開發出一個「穩賺不賠」的交易機器人，而是源於對股市運作機制的純粹好奇。我們想知道，被市場奉為圭臬的 技術面 (Technical)、基本面 (Fundamental) 與 籌碼面 (Chip)，對於股價波動存在著何種程度的干擾或因果關聯？

儘管課堂內容多次提及將 ML 應用於股價預測時，面臨著隨機漫步與低信噪比等本質上的困難，但本研究仍希望能透過實驗精神加以驗證。我們嘗試將影響因子的變數拆解為三大獨立面向，建立四組平行的 LSTM 模型進行測試。重點不在於追求完美的預測準確率，而是藉由觀察不同面向的數據在預測表現上的差異，試圖找出它們與股價之間真實的關聯性。

## 研究動機與目的 (Motivation & Objectives)

### 研究動機

雖然學術理論告訴我們金融時間序列具有低信噪比（Low Signal-to-Noise Ratio）的特性，但我們希望透過實作來驗證這個困難點的「邊界」在哪裡。

因此有了這些疑問:

- 如果將變數拆解得夠乾淨，是否能降低彼此的干擾，進而捕捉到微弱但存在的訊號？
- 外資買賣超、月營收成長、KD黃金交叉，這些指標在機器眼裡，哪些具有預測力，哪些只是隨機波動？
- 台積電在法說會前後，是否會從「隨機漫步」短暫轉變為「基本面主導」？在股災時，是否會轉變為「籌碼面主導」？
- 把所有資料都丟進去訓練真的比較準嗎？還是會因為雜訊過多而導致模型「暈船」？

## 資料來源與變數設計 (Data & Features)

本專案使用 finlab API 獲取台積電歷史數據，並將特徵分為以下四組：

|特徵集 (Set)|涵蓋變數 (Features)|資料特性|
|---|---|---|
|Set A技術面|開高低收、成交量、MA、RSI、KD、MACD|反應市場趨勢與情緒，更新最即時。|
|Set B基本面|月營收 (MoM, YoY)、季 EPS、毛利率、PE、殖利率|反應公司真實價值，需透過 Forward-Fill 對齊至日資料。|
|Set C籌碼面|三大法人買賣超、融資融券、大戶持股|反應市場資金流向與主力動態。|
|Set D混合式|包含 Set A + Set B + Set C 所有變數|用於測試全變數是否優於單一面向。|

## 研究方法與實驗設計 (Methodology)

### 實驗設計：模型競技場

1. 訓練模型 (Training)：分別建立四個 LSTM 模型，各自只看自己面向的數據（技術/基本/籌碼/混合）。

2. 數據對齊 (Alignment)：為了解決基本面資料更新慢的問題，我們嚴格執行 Forward-Fill，模擬真實投資人在當下只能看到「已公告」的舊財報，絕不偷看未來。

3. 測試模型 (Testing)：在 2022-2023 年（包含空頭修正與反彈）的充滿挑戰的市場環境下進行回測。

### 評判標準：誤差歸因 (Gap Analysis)

比起單純看準確率，我們更關注 「誰錯得比較少」。

針對每一天 $t$，比較四個模型的預測誤差 $E$：

$$Winner_t = \arg\min (E_{tech}, E_{fund}, E_{chip})$$

- 如果某段時間 $E_{chip}$ (籌碼面誤差) 顯著最小，代表那段時間台積電的股價是由資金流向決定的。

- 透過這種方式，我們試圖繪製出台積電股價的誤差分布圖。

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

我們預期（並準備接受）模型可能無法精準預測具體股價，但我們希望獲得以下洞察：

- 驗證困難點：親身體驗隨機漫步理論的威力，看模型在哪些時期會完全失效。

- 發現特定規律：找出三大面向各自的「好球帶」。例如：發現技術面在盤整期有效，而籌碼面在崩盤時有效。

- 自我成長：透過這個專案，從單純的「預測者」轉變為市場機制的「觀察者」。

## 未來展望 (Future Outlook)

- 增加更多面向組合：增加 set A+B、set B+C、set C+A，並比較其表現。
- 加入權重調整：在混合式面向中加入權重調整，並比較其表現。
- 納入第四面向（情緒面）：結合 NLP 自然語言處理 技術，將財經新聞或社群討論（如 PTT/Dcard）量化為情緒指標，驗證「市場信心」對股價的影響力。
- 跨標的分類與因果歸因：將此分析邏輯套用至不同屬性的公司（如小型飆股 vs. 大型權值股），並試圖將模型誤差的變化與現實重大事件（如升息、戰爭、法說會）進行自動化的因果對照。

## 技術棧 (Tech Stack)

Language: Python 3.14+  
Data Source: FinLab API  
Machine Learning: TensorFlow / Keras (LSTM), Scikit-learn  
Data Processing: Pandas, NumPy  
Visualization: Matplotlib, Seaborn  
