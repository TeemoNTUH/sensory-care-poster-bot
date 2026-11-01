# 🖼️ 感官衛教海報問答機器人 (Sensory Care Poster QA Bot)

**🔗 Cloudflare Workers Demo：<https://sensory-care-poster-bot.teemo-f15.workers.dev>**

醫院實習期間開發的圖片衛教問答系統。將病房張貼的感官照護衛教海報數位化，讓病人與家屬可以直接針對海報內容用自然語言提問，取代「站在海報前自己找重點」。

> 本專案是供來院實習生了解 RAG 智慧問答系統概念的展示專案，內容僅供學習與展示，不作為正式醫療服務或醫療建議。

## 功能

- 使用 LLM 進行衛教海報圖片內容分析，例如：「請輸入關於圖片或圖像內容的問題」
- 結合 Cloudflare AutoRAG 提供精準知識檢索與問答
- 對話式介面，可針對海報內容持續追問

## 系統運作方式

1. 衛教海報圖片與對應說明文字上傳到 Cloudflare R2
2. Cloudflare AutoRAG 對圖片內容與說明文字建立索引
3. 使用者提問時，AutoRAG 檢索最相關的內容，交給 Workers AI 上的 LLM（`gpt-oss-120b`）生成回答
4. Durable Objects 保存對話狀態，支援多輪追問

## 資料來源

測試與合成資料由 [@TeemoNTUH](https://github.com/TeemoNTUH) 提供。

## 新增海報

每張海報建議同時準備圖片與同名 Markdown 文字說明，方便 AI Search 依海報文字檢索；圖片本身可供視覺模型分析，因此 `.md` 不是必要檔案，但建議提供以提升文字內容的查找效果。說明應忠實轉錄海報內容，不要補入海報未提供的醫療建議。

請使用英數與連字號命名，並將圖片和 `.md` 放在 R2 bucket 根目錄，兩者 basename 相同，例如：

```text
hand-hygiene-poster.png
hand-hygiene-poster.md
```

從專案根目錄上傳至已設定的 `image-rag-bucket`：

```powershell
npx wrangler r2 object put image-rag-bucket/hand-hygiene-poster.png --file .\posters\hand-hygiene-poster.png --remote
npx wrangler r2 object put image-rag-bucket/hand-hygiene-poster.md --file .\posters\hand-hygiene-poster.md --remote
```

圖片建議小於 4 MiB。上傳後到 Cloudflare Dashboard 的 **AI Search → `image-rag` → Jobs** 手動觸發同步，確認新檔案索引完成；R2 外部資料來源預設每 6 小時同步一次。新增或更新海報資料不需要重新部署 Worker。

## 技術架構 (Cloudflare)

| 元件 | 用途 |
|---|---|
| Cloudflare Workers | Serverless 後端 / API，同時提供線上 Demo |
| Durable Objects | 對話狀態管理 |
| R2 Storage | 圖片與衛教文件儲存 |
| Workers AI | 模型推論（`@cf/openai/gpt-oss-120b`） |
| AutoRAG | 檢索增強生成 |
| Browser Rendering | 內容擷取 |

前端：React 18 + TypeScript + Emotion（CSS-in-JS）+ esbuild 打包

## 本機開發 / 部署

```bash
npm install

wrangler login

# Worker 設定已綁定 image-rag-bucket；只有尚未建立時才需要建立
wrangler r2 bucket create image-rag-bucket

# 到 Cloudflare Dashboard 建立/確認 AI Search instance 名稱為 image-rag，R2 來源指向 image-rag-bucket

npm run build
wrangler deploy
```

## 已知限制與未來規劃

- 目前針對單張海報內容問答，尚未支援跨多張海報的比較式提問
- 規劃：加入更多科別的衛教海報，擴充知識庫覆蓋範圍

## 專案背景

本專案以經原作者授權的 [Cloudflare Workers + AutoRAG 範例](https://github.com/elizabethsiegle/chat-w-taylor-on-newheights-and-travis-gq-autorag-openaioss) 為起點，在醫院實習期間改造為衛教海報圖片問答用途。
