# 👑🐱 喵皇陛下的御前問答｜貓咪飼養 RAG 對話機器人

一隻傲嬌的橘貓皇帝，根據農委會《貓隻飼養與照顧指南》等官方資料，教人類怎麼照顧貓。
喵皇自稱「朕」，叫使用者「鏟屎官」，只根據向量資料庫（御書房）裡的資料回答，每個重點都會附上出處。


## 專案內容

| 檔案 | 說明 |
|---|---|
| `notebooks/` 打造向量資料庫 | 把 38 份貓咪飼養知識文字塊轉成向量，建立 FAISS 向量資料庫並打包成 `rag-data.zip` |
| `notebooks/` RAG 對話機器人 | 讀入向量資料庫，以兩階段檢索加 Gemini 產生回答，用 Gradio 做對話介面 |
| `report/` | 書面報告 |

## 系統架構

```
38 份 .md 文字塊 ──► EmbeddingGemma ──► FAISS 向量資料庫（rag-data.zip）
                                              │
使用者提問 ──► EmbeddingGemma ──► FAISS 取 8 筆 ──► bge-reranker 重排取前 3
                                                          │
                              喵皇人設 Prompt + 最近 3 輪對話 ──► Gemini ──► Gradio
```

## 技術重點

| 功能 | 做法 |
|---|---|
| 文字切塊 | 一個主題一塊，自行改寫成 38 塊，避免語意被切斷 |
| Metadata | 標題、分類、來源、網址；Gradio 介面可以依分類篩選 |
| Embedding | `google/embeddinggemma-300m`，依官方建議加上 `title: {真實標題}` 前綴 |
| 兩階段檢索 | 先用向量取 8 筆候選，再用 Cross-Encoder `BAAI/bge-reranker-base` 重排取前 3 |
| 降低幻覺 | 相關度低於門檻就不提供資料，要求模型誠實回答「御書房沒有記載」 |
| 短期記憶 | 最近 3 輪對話放入 prompt；簡短追問會合併上一題一起檢索 |
| 安全護欄 | 偵測到急症關鍵字（尿不出來、誤食百合、抽搐等）時，先提醒立刻就醫 |
| 可解釋性 | 介面顯示用到的文字塊、分類、相似度、Rerank 分數與來源連結 |
| LLM | Google Gemini（透過 `aisuite`） |

## 如何執行

兩個 notebook 都是在 Google Colab 上執行，需要先在 Colab 左側的「密鑰」設定：

- `HuggingFace`：Hugging Face access token（EmbeddingGemma 需要先同意模型授權）
- `Gemini`：Google AI Studio 的 API key

執行順序：

1. 打開「打造向量資料庫」notebook，從頭執行，會自動下載原始資料並產生 `rag-data.zip`
2. 打開「RAG 對話機器人」notebook，從頭執行，會自動下載 `rag-data.zip` 並啟動 Gradio 介面

## 資料來源

- 行政院農業委員會（現農業部）《貓隻飼養與照顧指南》
- 農業部 貓咪納入寵物登記新聞稿、動物保護法修正說明
- Animal Trust, *Urinary obstruction in cats*
