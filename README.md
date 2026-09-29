# OverTranslate-assets

[OverTranslate](https://github.com/Hon-Lu/OverTranslate) 另外下載的資源。這些檔案太大、或只有部分使用者需要，所以不放進安裝包，
由 app 在使用者按下載時才從這裡的 Release 取得。

這個倉本身只放說明，檔案都在 Release 的附件裡。

## 規則

- **一個 Release＝一種資源的一個版本**，tag 是「資源名-v版本」，例如 `manga-vertical-v1`。
- 每種資源的檔名、大小與 SHA-256 寫在主倉的清單裡（例如 `src/OverTranslate/ocrmodels/manga-vertical.json`），
  app 下載後逐檔比對，不符就重抓。**Release 上的檔案一旦發布就不要替換**；內容有任何改變就開新版本。
- **舊版不要刪**：舊版 app 只認它自己那一版的清單與網址，只要還有人在用舊版 app，舊的 Release 就要留著。
- app 把下載的檔案放在 `%LocalAppData%\OverTranslate-assets\<資源名>\v<版本>\`，新版下載完成後會刪掉同一資源的舊版目錄。

## 資源

### manga-vertical-v1：漫畫直排模型

日文漫畫直排文字的偵測與辨識，在 DirectX 12 顯示卡上執行（ONNX Runtime DirectML）。沒有顯示卡或沒下載時，app 用內建的辨識。
對應主倉清單 `manga-vertical.json` 的 `version` 為 `1`。

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

授權全文見主倉的 `THIRD-PARTY-NOTICES.txt`。

#### 我們自己做的修改

這些檔案不是上游原檔，是 OverTranslate 轉出來的：

- **偵測器**：上游的 `detector.onnx` 轉成 fp16（`TopK`、`GatherElements`、`Cast` 保留 fp32），並刪掉轉換工具插入的重複 `Cast` 節點。
- **辨識器**：從上游 PyTorch 權重匯出 ONNX fp16。decoder 拆成兩張圖：一張算 cross-attention，一張帶 key/value cache 逐步產生。
  `vocab.txt` 是上游原檔，沒有修改。

匯出腳本與重做步驟在主倉 [`tools/MangaModelExport/`](https://github.com/Hon-Lu/OverTranslate/tree/main/tools/MangaModelExport)。
