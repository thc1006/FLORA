# FLORA: 為 O-RAN 適應性而生的聯邦學習框架 (Federated Learning for O-RAN Adaptation)

> FLORA: A Federated Reinforcement Learning framework for O-RAN Adaptation. Intelligently solving network slicing challenges through dynamic clustering, personalization, and privacy-preservation.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/release/python-390/)

**FLORA** 是一個專為學術研究與實務應用設計的開源框架，旨在探索如何利用**聯邦強化學習 (Federated Reinforcement Learning, FRL)** 的強大能力，來解決次世代 **O-RAN (開放無線接取網)** 環境中，極具挑戰性的**網路切片 (Network Slicing)** 資源管理問題。

這個專案不只是一個單一的模型，而是一個功能全面、可高度客製化的**實驗平台**，讓研究者可以輕鬆比較不同聯邦學習策略的優劣，並深入探討**適應性 (Adaptation)**、**個人化 (Personalization)** 與**隱私保護 (Privacy)** 等前沿議題。

## 專案簡介

在 5G/6G 的時代，網路不再只提供單一服務。我們需要同時滿足需要超大頻寬的影音串流 (eMBB) 和要求超低延遲的自駕車或工業物聯網 (URLLC) 等各式各樣的需求。網路切片技術因此而生，但要如何「智慧化」地動態分配資源給不同的切片，一直以來都是個大哉問。

傳統的集中式 AI 訓練需要將所有基地台的原始數據都傳回中央伺服器，這不僅帶來了巨大的數據傳輸成本，更有侵犯用戶隱私的風險。

**FLORA** 的誕生，就是為了解決這個問題。

我們採用聯邦學習的去中心化範式，讓每個基地台（在我們的框架中稱為「客戶端」）在本地端用自己的數據訓練模型，只將加密、匿名的模型更新資訊分享出來。FLORA 的核心目標，是訓練出一個能夠聰明判斷「在當下這個時間點，我應該更重視 eMBB 的吞-吐量，還是更優先保證 URLLC 的低延遲？」的智慧代理 (Agent)。

## 核心功能與特色 ✨

FLORA 框架經過精心設計，內建了許多為了方便研究比較的強大功能：

* **📈 多模式實驗平台:** 內建多種主流訓練架構，一鍵切換，輕鬆進行效能比較：
    * `Centralized`: 傳統的集中式訓練。
    * `Isolated`: 各客戶端完全獨立訓練，互不干擾。
    * `FedAvg`: 最經典的聯邦平均演算法。
    * `FedProx`: 專為處理客戶端之間數據差異（Non-IID）而設計的演算法。

* **🤖 智慧動態分群 (Dynamic Clustering):** 這是 FLORA 最具創新的功能之一！在 `ClusteredFL` 模式下，伺服器會定期分析所有客戶端的模型相似度，使用 K-Means 演算法自動將「行為模式」相近的客戶端分在同一群。聚合只在群內進行，讓模型能更好地適應不同區域的網路特性。

* **👤 個人化聯邦學習 (PFL):** 我們相信「因材施教」！在聯邦訓練結束後，FLORA 支援讓每個客戶端拿著訓練好的共用模型，在自己的本地數據上進行最後的「微調 (Fine-tuning)」，打造出最適合自己的專屬模型。

* **🛡️ 負遷移防護 (Negative Transfer Protection):** 微調不見得總是好事。FLORA 內建了智慧回退機制，如果個人化微調後的模型效能反而變差了（學歪了），系統會自動將其捨棄，保留原本效能較佳的模型。

* **🔒 差分隱私 (Differential Privacy):** 隱私是我們的重中之重。本框架整合了 Google 的 Opacus 函式庫，可以在聯邦學習過程中為模型更新添加數學上可證明的隱私保障，並能有效監控與管理隱私預算 (Epsilon) 的消耗。

* **🌐 現實世界模擬:** 為了讓研究更貼近真實場景，我們還模擬了：
    * **模型壓縮 (Compression):** 模擬行動裝置上傳模型時的有限頻寬。
    * **客戶端掉線 (Dropout):** 模擬不穩定的網路環境下，部分客戶端無法參與該輪訓練的情境。

## 資料來源

本專案的所有實驗，皆基於一個公開的、真實世界的 O-RAN 模擬數據集：

* **[CoLORAN: A Radio Access Network Slicing and Orchestration Open Dataset](https://github.com/wineslab/colosseum-oran-coloran-dataset)**

我們由衷感謝 WINES Lab 提供如此高品質的數據，讓學術研究社群能在此基礎上進行探索與創新。在您使用本框架時，也請記得引用他們的原始論文。

## 架構概覽

整個程式碼主要由以下幾個核心模組構成：

1.  `DataManager`: 負責讀取並預處理 `kpi_traces_final_robust0.parquet` 檔案，並為每個客戶端準備好專屬的數據軌跡。
2.  `PairedEnv`: 為強化學習代理量身打造的互動「環境」。
3.  `RLAgent`: 實現了 DQN 演算法的核心學習代理，並整合了隱私保護、FedProx 等邏輯。
4.  `FLServer`: 模擬聯邦學習中的中央伺服器，負責模型的分發、聚合與客戶端分群。
5.  `ExperimentRunner`: 實驗的總指揮，負責串連所有模組，執行完整的訓練與評估流程。

## 如何執行實驗

所有實驗的設定與執行都集中在 `main.ipynb` (或是 `.py` 檔案) 的最後一個區塊 (`Cell 8`)。

1.  **設定路徑:** 請先確認 `DATA_PATH` 指向您存放 `kpi_traces_final_robust0.parquet` 檔案的正確位置。
2.  **選擇模式與種子:** 您可以修改 `MODES_TO_RUN` 列表來決定要執行哪些實驗模式，並透過 `SEEDS` 列表來進行多次獨立實驗以確保結果的穩健性。
3.  **執行:** 執行該 Cell，實驗便會自動開始。所有結果（包含訓練歷史、評估數據、隱私成本和設定檔）都會被儲存在 `BASE_OUTPUT_DIR` 所定義的資料夾中。

## 授權 (License)

本專案採用 [MIT License](LICENSE) 進行授權。

## 如何引用我們 (Citing Us)

如果您在您的學術研究中使用了 FLORA 框架，請考慮引用我們的論文（論文發表後會在此提供 BibTeX 格式）。

```bibtex
@inproceedings{flora2025,
  title={FLORA: Federated Learning for O-RAN Adaptation},
  author={Your Name, et al.},
  booktitle={Conference Name},
  year={2025}
}
