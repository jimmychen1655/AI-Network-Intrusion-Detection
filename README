# AI-Based Network Intrusion Detection System (IDS)

本專案利用機器學習演算法（非監督式學習 - 孤立森林 Isolation Forest），針對真實網路流量數據進行異常偵測與分析。專案著重於網路安全大數據的特徵工程、資料清洗，以及模型在資安場景下的成效評估與優化思考。

## 📊 實驗資料集
* **來源**：加拿大網路安全研究院開源資料集 **CICIDS2017**（Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv）。
* **分析規模**：前 200,000 筆真實網路流量。
* **真實標籤分佈**：
  * `PortScan`（埠口掃描攻擊）：109,268 筆
  * `BENIGN`（正常安全流量）：90,478 筆

## 🛠️ 開發環境與套件
* **開發環境**：VS Code (Jupyter Notebook / Python 3.12+)
* **核心套件**：`pandas`, `numpy`, `scikit-learn`

## 🚀 核心實作流程

### 1. 特徵工程與資料清洗 (Data Cleaning)
網路流量原生資料常包含大量髒資料，本專案實作了以下清洗保險關卡：
* 清除欄位名稱前後包含的空白字元（修正 CICIDS2017 經典的欄位格式錯誤）。
* 將資料集中的無限大值（`inf`, `-inf`）強轉為 `NaN`，並利用欄位平均值進行補值。
* 篩選出 5 項關鍵數值行為特徵，排除 IP、Timestamp 等易導致過擬合（Overfitting）的 Meta Data：
  * `Flow Duration` (流量持續時間)
  * `Total Fwd Packets` (前向封包總數)
  * `Total Backward Packets` (後向封包總數)
  * `Flow Bytes/s` (每秒傳輸位元組)
  * `Flow Packets/s` (每秒傳輸封包數)
* 使用 `StandardScaler` 進行特徵標準化，將資料縮放至相同尺度以利非監督式模型收斂。

### 2. 模型部署與版本相容性踩坑紀錄
在部署 `scikit-learn` 的 `IsolationForest` 時，遇到了新版本底層嚴格校驗的 InvalidParameterError 挑戰：
* **問題 A**：新版限制 `contamination` 參數上限為 `0.5`，而本資料集異常佔比高達 54%。
  * **解法**：將 `contamination` 調整為符合新規的上限值 `0.5`。
* **問題 B**：`fit_predict()` 與底層多執行緒（`n_jobs`）引發相容性錯誤。
  * **解法**：將工作流重構為古典且穩定的「拆開執行」：先 `fit()` 訓練模型，再單獨進行 `predict()`，並強制將矩陣轉為標準 `np.float32`，成功解決報錯。

---

## 📈 成果評估 (Evaluation)

執行混淆矩陣（Confusion Matrix）交叉比對「原始真實標籤」與「AI 預測結果」，得出以下成果評估表：

| 原始真實標籤 (Label) | AI 判定為 Anomaly (異常) | AI 判定為 BENIGN (正常) |
| :--- | :---: | :---: |
| **BENIGN (正常流量)** | **82,299** | **8,179** |
| **PortScan (埠口掃描)** | **17,230** | **92,038** |

### 💡 統計結論與資安行為剖析
* **高誤報與漏報原因**：由於非監督式學習（孤立森林）高度依賴統計上的「疏離程度」。現代埠口掃描工具（如 Nmap）常採用低頻率、慢速掃描策略，其單一封包行為特徵與常規網頁瀏覽（BENIGN）極度相似，導致模型在有限特徵下難以完全切分。
* **後續優化方向**：
  1. 改用**監督式學習分類器**（如 Random Forest 或 XGBoost），利用標籤進行強監督訓練。
  2. 引入更有資安鑑別度的特徵，例如 `Destination Port`（目的地埠口變動率）或 `Bwd Packet Length Std`（後向封包長度標準差）。