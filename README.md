# OverTranslate-assets

[OverTranslate](https://github.com/Hon-Lu/OverTranslate) 另外下載的資源。這些檔案太大、或只有部分使用者需要，所以不放進安裝包，
由 app 在使用者按下載時才取得。

**主要來源是 Hugging Face，這裡的 Release 是備援**：app 依主倉清單的 `sources` 順序嘗試，Hugging Face 連不上、檔案雜湊不符或太慢
（低於 256 KB/s 持續 20 秒）時才改從這裡下載。GitHub Release 的下載速度曾長期只有約 40 KB/s，所以不當主要來源。

這個倉本身只放說明，檔案都在 Release 的附件裡。

## 規則

- **一個 Release＝一種資源的一個版本**，tag 是「資源名-v版本」，例如 `manga-vertical-v1`。
- **改版時三個地方一起更新**：Hugging Face 的模型倉（上傳並打新 tag `v版本`，模型卡同步）、這裡的新 Release、主倉清單的
  `sources` 與雜湊。三處的檔名、大小、SHA-256 必須完全相同。
- 每種資源的檔名、大小與 SHA-256 寫在主倉的清單裡（例如 `src/OverTranslate/ocrmodels/manga-vertical.json`），
  app 下載後逐檔比對，不符就重抓。**Release 上的檔案一旦發布就不要替換**；內容有任何改變就開新版本。
- **舊版不要刪**：舊版 app 只認它自己那一版的清單與網址，只要還有人在用舊版 app，舊的 Release 就要留著。
- app 把下載的檔案放在 `%AppData%\OverTranslate\models\<資源名>\v<版本>\`（和設定、logs 同一個資料夾），新版下載完成後會刪掉同一資源的舊版目錄。

## 資源

### manga-vertical-v2：漫畫直排模型

偵測器與 v1 相同；辨識器改為 [muscgab/manga-ocr-base-CPT-flex](https://huggingface.co/muscgab/manga-ocr-base-CPT-flex)（從 manga-ocr-base 接續預訓練、保持長寬比讀取），錯字比 v1 少。
對應主倉清單 `manga-vertical.json` 的 `version` 為 `2`。

| 來源 | 位置 |
|---|---|
| Hugging Face（主要） | 偵測器：[hon-lu/comic-text-and-bubble-detector-onnx-fp16](https://huggingface.co/hon-lu/comic-text-and-bubble-detector-onnx-fp16)（tag `v1`）；辨識器：[hon-lu/manga-ocr-base-CPT-flex-onnx-fp16](https://huggingface.co/hon-lu/manga-ocr-base-CPT-flex-onnx-fp16)（tag `v1`） |
| GitHub（備援） | 本倉 Release `manga-vertical-v2` |

| 檔案 | 大小（bytes） | SHA-256 |
|---|---:|---|
| `detector.fp16.onnx` | 84,725,998 | `2ed2ab1f56543d9844025a87a88a96f2ff8fba1f93d5c0c89b55c0f594b29c25` |
| `ocr-encoder.fp16.onnx` | 171,982,123 | `f86a0455d63f87d955fe56be1380762ad621ce0dc34d0e4f3e6c512e2562b418` |
| `ocr-decoder-cross.fp16.onnx` | 4,732,209 | `f96b76dc239d49978d9167b55c58bf65fc786aa9bf3b3792f18c60aa264047ca` |
| `ocr-decoder-step.fp16.onnx` | 56,062,492 | `3c5e51e18c4f96c0c71cc1ba0122c75067092a1ce269f703c143d3211352b4d5` |
| `vocab.txt` | 27,001 | `a380ec969e2d73858cd84f43cdadb78b8527b12194ea4f797368ac21a24fa98c` |
| 合計 | 317,529,823（約 303 MB） | |

#### 來源與授權

| 檔案 | 來源模型 | 授權 |
|---|---|---|
| `detector.fp16.onnx` | [ogkalu/comic-text-and-bubble-detector](https://huggingface.co/ogkalu/comic-text-and-bubble-detector)（RT-DETR-v2 r50vd） | Apache License 2.0 |
| `ocr-*.fp16.onnx`、`vocab.txt` | [muscgab/manga-ocr-base-CPT-flex](https://huggingface.co/muscgab/manga-ocr-base-CPT-flex)（初始權重來自 [kha-white/manga-ocr-base](https://huggingface.co/kha-white/manga-ocr-base)） | **CC BY-NC-SA 4.0** |

辨識器的權重**不是** Apache License 2.0：依上游以 **CC BY-NC-SA 4.0** 散布，不得商用，授權全文與上游的來源說明附在 Release 的 `LICENSE_MODEL`、`NOTICE.md`（上游原檔）。原作者同意本 app 的使用、轉檔與託管方式：[討論串](https://huggingface.co/muscgab/manga-ocr-base-CPT/discussions/1)。偵測器照舊依 Apache License 2.0（本倉 [`LICENSE`](LICENSE)）。

#### 我們自己做的修改

- **偵測器**：與 v1 同一個檔。
- **辨識器**：上游權重名稱對回 Hugging Face `VisionEncoderDecoderModel` 的格式後匯出 ONNX fp16。encoder 吃保持長寬比的 patch 輸入（固定 196 個 patch，補位的 patch 以遮罩排除），decoder 拆成兩張圖：一張算 cross-attention，一張帶 key/value cache 逐步產生，並加上記憶遮罩。
  `vocab.txt` 是 BERT 特殊 token 加上上游的 6,817 個字。

匯出腳本與重做步驟在主倉 [`tools/MangaModelExport/`](https://github.com/Hon-Lu/OverTranslate/tree/main/tools/MangaModelExport)，可從上游權重逐位元重現。

### manga-vertical-v1：漫畫直排模型

日文漫畫直排文字的偵測與辨識，在 DirectX 12 顯示卡上執行（ONNX Runtime DirectML）。沒有顯示卡或沒下載時，app 用內建的辨識。
對應主倉清單 `manga-vertical.json` 的 `version` 為 `1`。

| 來源 | 位置 |
|---|---|
| Hugging Face（主要） | 偵測器：[hon-lu/comic-text-and-bubble-detector-onnx-fp16](https://huggingface.co/hon-lu/comic-text-and-bubble-detector-onnx-fp16)（tag `v1`）；辨識器：[hon-lu/manga-ocr-base-onnx-fp16](https://huggingface.co/hon-lu/manga-ocr-base-onnx-fp16)（tag `v1`） |
| GitHub（備援） | 本倉 Release `manga-vertical-v1` |

| 檔案 | 大小（bytes） | SHA-256 |
|---|---:|---|
| `detector.fp16.onnx` | 84,725,998 | `2ed2ab1f56543d9844025a87a88a96f2ff8fba1f93d5c0c89b55c0f594b29c25` |
| `ocr-encoder.fp16.onnx` | 171,785,393 | `d58ead8c0074f6d9f6382abf29188c0ed885e394f9d29260fa86df633da640eb` |
| `ocr-decoder-cross.fp16.onnx` | 4,732,209 | `967c31b739ee9cdc11ec40381540efacfc8aca6f2fb83948e0a17fa281b2f045` |
| `ocr-decoder-step.fp16.onnx` | 53,977,443 | `db6becffe3f1a232299bf6d33d768824ded1430473a0d22668606f77a027b6d1` |
| `vocab.txt` | 24,072 | `344fbb6b8bf18c57839e924e2c9365434697e0227fac00b88bb4899b78aa594d` |
| 合計 | 315,245,115（約 301 MB） | |

#### 來源與授權

| 檔案 | 來源模型 | 授權 |
|---|---|---|
| `detector.fp16.onnx` | [ogkalu/comic-text-and-bubble-detector](https://huggingface.co/ogkalu/comic-text-and-bubble-detector)（RT-DETR-v2 r50vd） | Apache License 2.0 |
| `ocr-*.fp16.onnx`、`vocab.txt` | [kha-white/manga-ocr-base](https://huggingface.co/kha-white/manga-ocr-base)（[manga-ocr](https://github.com/kha-white/manga-ocr)） | Apache License 2.0 |

兩個模型與我們轉出來的檔案都依 **Apache License 2.0** 散布，授權全文見本倉的 [`LICENSE`](LICENSE)（官方原文）；修改內容見下一節。

#### 我們自己做的修改

這些檔案不是上游原檔，是 OverTranslate 轉出來的：

- **偵測器**：上游的 `detector.onnx` 轉成 fp16（`TopK`、`GatherElements`、`Cast` 保留 fp32），並刪掉轉換工具插入的重複 `Cast` 節點。
- **辨識器**：從上游 PyTorch 權重匯出 ONNX fp16。decoder 拆成兩張圖：一張算 cross-attention，一張帶 key/value cache 逐步產生。
  `vocab.txt` 是上游原檔，沒有修改。

匯出腳本與重做步驟在主倉 [`tools/MangaModelExport/`](https://github.com/Hon-Lu/OverTranslate/tree/main/tools/MangaModelExport)。
