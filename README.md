# GovAid Copilot

把育兒與租屋補助的條件，整理成可以核對、更正、下載的申辦指南。

你可以直接填表，或先用對話描述情況。Gemini 整理條件、解釋政策；確認資料後，GovAid 用固定版本的規則試算。最後可下載 PDF，也可讓支援 MCP 的外部助手一起核對案件與文件。

**版本與固定映像：見 `release-manifest.json`** · 支援 Windows、macOS 或 Linux 上的 Docker；映像平台為 `linux/amd64`，ARM 裝置尚未驗證。

## 開始使用

先安裝並啟動 Docker Desktop（Linux containers）。不必另外安裝 Python 或 Node.js。

```powershell
git clone https://github.com/AllenHZL/AAIC2026_GovAid_Demo.git
cd AAIC2026_GovAid_Demo
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

以下指令以 Windows PowerShell 為例；macOS／Linux 可用 `cp -n .env.example .env` 複製設定，其餘 Docker 指令相同。

用編輯器開啟 `.env`：

- 使用 Gemini：保留 `MODEL_MODE=live`、`MODEL_NAME=gemini-3.5-flash-lite`，把自己的 key 填入 `MODEL_API_KEY`。
- 不使用模型：改為 `MODEL_MODE=mock`，即可體驗填表、試算和文件功能。此模式的對話不代表 Gemini 的回答能力。

請使用未啟用計費的免費專案；額度依 Google 帳號而定。GovAid 不會替帳號設定費用上限。不要提交或分享 `.env`，也不要把模型 key 放進 MCP 設定。

```powershell
docker compose --profile mcp pull
if ($LASTEXITCODE -ne 0) { throw '映像下載失敗' }
docker compose up -d --wait db api worker web
if ($LASTEXITCODE -ne 0) { throw '啟動失敗，請查看 docker compose logs --tail=50' }
```

開啟 **[GovAid 網頁](http://127.0.0.1:13028)**。網頁固定使用 `13028`，API 使用 `18028`；資料庫不開放主機連接埠。請用合成資料體驗，保持服務只在本機使用。

`docker compose ps -a` 中，`migrate` 和 `seed` 顯示 `Exited (0)` 代表初始化完成。API、Web、DB 應為健康狀態，Worker 應持續運行。

## 四個工作區

要用手機體驗，請依 [同 Wi-Fi 連線](docs/mobile.md) 開啟選用的區網入口。預設仍只接受本機連線。分享期間只用合成資料；IP 不是密碼，HTTP 不加密，能連線的人可能消耗聊天配額。教學另附關閉分享的方法。

| 工作區 | 用法 |
| --- | --- |
| 需求與問答 | 選擇方案，描述情況或載入範例，查閱政策來源。 |
| 確認條件 | 核對對話整理出的欄位，連續補完缺項，再一次儲存。 |
| 結果與比較 | 查看試算及原因；用假設條件比較，不更動正式案件。 |
| 我的資料包 | 依案件代號找到指南，下載 PDF，管理外部助手的讀取權限。 |

填完條件後，到表單底部「最後核對」確認資料，再按「儲存並試算」。日期或數值不正確時，畫面會標紅並帶到欄位；西元年份請填四位數。尚未確認的項目可留白儲存，系統會列為待補。

目前支援桃園中央租補、臺北中央租補、桃園未滿二歲育兒津貼、臺北友善托育的一般案件。其他方案標示為開發中。試算只供申辦準備，實際資格與核定金額由主管機關審查；GovAid 不代為送件。

## 操作範例：小塔的租屋補助

1. 選桃園中央租補，載入合成範例，核對後儲存。月租 12,000 元，試算 2,880 元。
2. 開啟聊天，先比較假設：

   ```text
   如果租金改成2500元，會領多少？不要更正案件。
   ```

3. 原條件仍為 12,000 元。接著提出正式更正：

   ```text
   更正租約月租金為2500元，原先12000元填錯，其他條件不變。
   ```

4. 查看提案，確認原值與新值，到表單底部「最後核對」確認受影響的項目，再儲存。月租與試算均為 2,500 元。
5. 詢問原因：

   ```text
   為什麼本案試算只有2500元？請用白話說明實際租金如何限制補助，附本案官方來源。
   ```

6. 完成最後一問後，核對目前版本，到「我的資料包」開啟指南並下載 PDF。想再核對文件，可接著使用 MCP。

範例採固定日期的合成情境，不代表今天的正式資格。Gemini 可以提出欄位與解釋，補助金額仍由規則計算。表單試算、PDF 和 MCP 讀取不呼叫 Gemini；聊天則可能使用 API 配額。回答若標示系統備援，表示本輪沒有取得可用的模型回答。

## 用 MCP 複核 PDF 與筆記

Gemini 協助你在 GovAid 內整理資料；MCP 則讓外部助手取得已授權的案件，將案件、PDF 和你的筆記放在一起核對。MCP 是資料連接工具，本身不是模型；外部助手使用它自己的帳號方案。

先確認連線：

```powershell
docker compose run --rm --no-deps -T mcp --self-check
```

結果應有 `ok: true`，`model_mode` 應與 `.env` 相同。在支援本機 stdio MCP 的 App 新增連線：

- 名稱：`govaid-readonly`
- 啟動指令：`docker`
- 工作目錄：此 repo 的完整路徑
- 引數：逐項填入下列內容，以下 `C:\projects\AAIC2026_GovAid_Demo` 是示意，請將三個路徑換成自己的位置

```text
compose
--project-directory
C:\projects\AAIC2026_GovAid_Demo
--env-file
C:\projects\AAIC2026_GovAid_Demo\.env
-f
C:\projects\AAIC2026_GovAid_Demo\compose.yaml
run
--rm
--no-deps
-T
mcp
```

環境變數及透傳清單留空。儲存並連線後，應有八個工具：列政策、政策詳情、來源搜尋、開始授權、完成授權、案件快照、準備包、申辦指南。

將 PDF 附給外部助手，並貼上：

```text
請用 GovAid MCP 複核附件 PDF。案件 ID：<貼上卡牌完整 ID>。
舊筆記：「可以領2880元，而且已核准，所以不用再準備租約。」
先讀最新案件，再取同 revision 的準備包與指南；核對 PDF 的案件、版本、租金、試算和政策 hash。
搜尋本案政策的「實際租金」和「租賃契約」。指出筆記錯誤，附官方來源，列三項下一步。
未授權先等我同意；沒有讀到附件就說未完成 PDF 比對。不修改案件、不送件、不另呼叫模型 API。
```

助手會提供授權連結與核對字串。請在建立案件的同一瀏覽器開啟連結，確認後自行同意，再請助手繼續。MCP 不會任意讀取電腦檔案，PDF 必須由你附上。

你可以在資料包的「管理授權」撤銷權限。撤銷會阻止後續讀取，但不會收回已下載的 PDF 或助手已讀過的內容。更新 API 容器後，請重新連線 MCP；案件授權也有期限。

## 更新與停止

保留原 `.env`，在 repo 內更新：

```powershell
git pull --ff-only
if ($LASTEXITCODE -ne 0) { throw '更新失敗，請保留現況' }
docker compose --profile mcp pull
if ($LASTEXITCODE -ne 0) { throw '映像下載失敗' }
docker compose up -d --wait db api worker web
```

Compose 固定映像版本，不沿用舊 `.env` 中的 `GOVAID_*_IMAGE` 值。不要同時啟動兩份使用相同連接埠的 GovAid。

```powershell
# 停止服務，保留案件
docker compose down
# 下次啟動
docker compose up -d --wait db api worker web
```

不要加 `-v`，除非確定要清空資料卷。匿名存取預設八小時閒置／二十四小時上限，可自行選擇保留七天。請維持同一瀏覽器、網址與資料卷；憑證失效不能只靠案件 ID 取回資料。你可在介面移除本機紀錄，或刪除自己的案件。

## 常見問題

- **連接埠已被占用：** 先用 `docker ps` 檢查是否已有 GovAid 在運行，勿直接刪除資料卷。
- **PDF 和網頁不同：** 完成最後更正與問答後，重新核對並下載最新 PDF。
- **Gemini 額度用完：** 可改用表單繼續整理資料，不必開啟付費。
- **資料包無法讀取：** 網路或服務暫時失敗可重試。憑證已失效時，請回到原本有權限的瀏覽器；若已無有效憑證，從條件表單建立新案。新案不會刪除舊案或恢復其權限。

## 架構與資料來源

- [系統架構](docs/architecture.md)
- [場域需求與驗證資料](docs/場域需求與驗證資料.pdf)（已獲准公開）。三位匿名受訪者的質性回饋，已同意公開供競賽使用。
- [來源與授權](NOTICE.md)
- 版本與映像清單：`release-manifest.json`。

此 repo 提供 Docker 啟動設定與使用文件，應用程式位於 API、Web、MCP 三個映像內；不是完整原始碼庫。
