## 從原始數據到 `kpi_traces_final_robust0.parquet`

這份文件旨在說明如何利用提供的 Python 腳本，處理來自 `colosseum-oran-coloran-dataset` 的原始數據，並產出一個經過整合、清理且可用於後續分析的 `kpi_traces_final_robust0.parquet` 檔案。整個流程主要分為環境設定、資料下載、初步處理與最終合併四個階段。

### **第一步：環境設定與初始化 (Cell 1 & 2)**

在資料處理開始前，腳本會先進行一系列的環境設定，確保後續步驟能順利執行。

1.  **掛載 Google Drive**：
    * 程式碼首先會嘗試掛載您的 Google Drive 到 Colab 環境的 `/content/drive` 路徑下。這是為了能夠讀取和儲存後續處理的檔案。
    * `WORK_DIR` 變數被設定為 `/content/drive/MyDrive/FRL_Slicing_Sim2`，作為整個專案的工作目錄，所有後續產生的檔案都會存放在此。

2.  **下載資料集**：
    * 接著，腳本會檢查 `colosseum-oran-coloran-dataset` 這個資料夾是否存在於工作目錄中。
    * 如果不存在，它會從 GitHub (`https://github.com/wineslab/colosseum-oran-coloran-dataset.git`) 自動下載資料集。這個資料集包含了所有實驗的原始 `.csv` 檔案。

3.  **設定全域變數**：
    * **路徑設定**：定義了原始資料路徑、中繼檔案（`kpi_traces_bs_intermediate0.parquet` 和 `kpi_traces_slice_intermediate0.parquet`）以及最終輸出檔案 `kpi_traces_final_robust0.parquet` 的儲存位置。
    * **業務邏輯參數**：建立一個名為 `scheduling_policy_map` 的字典，將資料夾名稱（如 `sched0`）對應到更具可讀性的排程策略名稱（如 `RR` - Round Robin）。

---

### **第二步：資料初步處理與轉換 (Cell 3)**

這個階段的核心任務是**讀取分散的原始 CSV 檔案**，並將它們轉換成兩個結構化的中繼 Parquet 檔案：一個給**基地台 (BS)**，另一個給**網路切片 (Slice)**。

1.  **檔案探索**：
    * 使用 `glob` 函式，程式會掃描 `rome_static_medium` 資料夾下所有符合特定命名規則的 CSV 檔案。
    * 它會分別找出所有**切片層級** (`slices_bs*/*_metrics.csv`) 和**基地台層級** (`bs*.csv`) 的數據檔案。

2.  **解析與處理**：
    * 程式會遍歷找到的每一個 CSV 檔案，並透過一個名為 `parse_path_info` 的輔助函式從檔案的路徑中**提取關鍵的元數據 (metadata)**。這些元數據包括：
        * `Scheduling_Policy_Active` (排程策略)
        * `Training_Config_ID` (訓練配置 ID)
        * `exp_id` (實驗 ID)
        * `BS_ID` (基地台 ID)
    * **針對基地台 (BS) 數據**：
        * 讀取 CSV 檔案。
        * 將 `time` 欄位更名為 `timestamp`。
        * 將前面提取的元數據作為新的欄位加入 DataFrame。
    * **針對網路切片 (Slice) 數據**：
        * 讀取 CSV 檔案，並移除任何無用的 `Unnamed:` 欄位。
        * **重新命名**多個關鍵欄位以增加可讀性與一致性，例如：
            * `Timestamp` -> `timestamp`
            * `slice_id` -> `Slice_ID`
            * `tx_brate downlink [Mbps]` -> `Throughput_DL_Mbps`
        * 同樣地，將提取的元數據加入 DataFrame。
    * 程式碼加入了**錯誤日誌**機制，若有任何檔案在讀取或處理過程中發生錯誤，將會被記錄下來，而不會中斷整個流程。

3.  **儲存中繼檔案**：
    * 所有處理過的基地台數據會被整合成一個大的 DataFrame (`df_bs_all`)，並儲存為 `kpi_traces_bs_intermediate0.parquet`。
    * 所有處理過的切片數據也會被整合成另一個大的 DataFrame (`df_slice_all`)，並儲存為 `kpi_traces_slice_intermediate0.parquet`。
    * 使用 Parquet 格式能大幅提升後續讀取和處理的效率。

---

### **第三步：最終合併與產出 (Cell 4)**

這是整個流程的最後一步，目標是將前一階段產生的兩個中繼檔案 (`df_bs` 和 `df_slice`) **智慧地合併**成一個最終的、乾淨的資料集。

1.  **載入中繼檔案**：
    * 首先，從硬碟讀取 `kpi_traces_bs_intermediate0.parquet` 和 `kpi_traces_slice_intermediate0.parquet`。

2.  **資料清理與型別轉換**：
    * 為了確保合併的準確性，程式會對合併的鍵 (`by_keys` 和 `on_key`) 進行嚴格的型別轉換。
    * `timestamp` 欄位被轉換為標準的日期時間格式。
    * `exp_id` 和 `Training_Config_ID` 中的文字（如 'exp', 'tr'）被移除，並轉換為數值整數型別。
    * 任何在這些關鍵欄位中存在空值或轉換失敗的資料行都會被**丟棄**，以確保資料的完整性。

3.  **排序與去重**：
    * 為了提升合併效能並確保資料的唯一性，兩個 DataFrame 都會根據合併鍵（包含 `timestamp`）進行排序。
    * 接著，會**移除完全重複的資料行**，只保留第一次出現的紀錄。

4.  **分組時間序列合併 (Group-wise Time-Series Merge)**：
    * 這是最關鍵的一步。直接對兩個龐大的 DataFrame 進行時間序列合併 (merge_asof) 可能會非常慢或導致記憶體不足。
    * 因此，程式採用了一個更高效的策略：
        * 它先將 `df_slice` 根據 `by_keys`（`BS_ID`, `exp_id` 等）進行**分組**。
        * 然後，它**逐一處理**每一個小的分組，對每個分組執行 `pd.merge_asof`。
    * `pd.merge_asof` 是一種特殊的時間序列合併方式，它會根據 `timestamp` 將 `df_slice` 中的每一筆紀錄與 `df_bs` 中**時間上最接近且不大於它**的紀錄進行配對。
        * **`direction='backward'`**：這是一個**關鍵修正**，確保了每個切片的狀態只會與其**過去或當下**的基地台狀態相關聯，避免了使用未來數據的「資料洩漏」問題。
        * **`tolerance=pd.Timedelta('150ms')`**：設定了一個時間容忍度。如果切片紀錄的時間戳與基地台紀錄的時間戳相差超過 150 毫秒，則不進行合併。

5.  **最終儲存**：
    * 所有經過分組合併後的數據會被重新整合成一個最終的 DataFrame (`final_df`)。
    * 這個 DataFrame 最後被儲存為 `kpi_traces_final_robust0.parquet` 檔案，這就是我們最終需要的、整合了基地台與切片效能指標的乾淨資料集。

總之，這份程式碼透過一個**分階段、由粗到精**的流程，將散亂的原始 CSV 數據，系統性地轉化為一個結構化、經過清理且可以直接用於機器學習模型訓練或深度分析的 Parquet 檔案。
