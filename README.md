# GovAid 初賽 Demo

本目錄是啟動包。程式、政策快照及 MCP 已打包成三個映像，不需要安裝 Python 或 Node.js。

三個應用映像已發布至 GHCR（`preliminary-9319accce8cf`），啟動檔已固定映像 digest。從空白資料庫啟動後，小塔的 Gemini＋MCP＋PDF 流程已通過；驗收範圍與待辦見下方文件。

## 啟動

先安裝並啟動 Docker Desktop。切到本目錄，在 PowerShell 執行：

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
# 編輯 .env，填入自己的 Gemini Free tier MODEL_API_KEY。
docker compose --profile mcp pull
if ($LASTEXITCODE -ne 0) { throw '下載失敗，先不要啟動' }
docker compose up -d --wait db api worker web
if ($LASTEXITCODE -ne 0) { throw '啟動失敗，先檢查 docker compose logs' }
docker compose ps -a
```

開啟 [GovAid](http://127.0.0.1:13028)。畫面應顯示 live、gemini-3.5-flash-lite。
初始化會執行資料庫遷移與政策匯入；這兩個服務不用模型。Gemini 只在送出需要解析的訊息時使用 API。

- [四分鐘 demo 與預期結果](docs/demo.md)
- [MCP App 設定、每案授權與諮詢](docs/mcp.md)
- [驗收與發布狀態](docs/release-status.md)
- [競賽交付範圍](docs/competition.md)
- [場域需求與驗證資料](docs/場域需求與驗證資料.pdf)（已獲准公開）。

## 停止

```powershell
docker compose down
```

保留資料卷即可保留案件。不要加 `-v`，除非確定要刪除全部測試案件。

## 使用範圍

桃園中央租補、臺北中央租補、桃園育兒津貼、臺北友善托育可操作。其餘方案顯示開發中。
試算與準備包協助整理，不代表政府核定。臺北候選政策及特殊身分仍有適用限制，詳見 [限制與素材](docs/limitations.md)。

Gemini 使用你提供的 API key；MCP 只讀取本機案件，不帶模型 key、不呼叫 Gemini。App 的回答使用 App 本身的訂閱／帳號額度。Free tier 是否適用以你的專案設定為準，啟動包不會替你設定帳務或購買額度。
