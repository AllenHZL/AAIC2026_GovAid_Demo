# GovAid Copilot

把育兒與租屋補助的條件，整理成可以核對、更正、下載的申辦指南。

你可以直接填表，或先用對話描述情況。Gemini 整理條件、解釋政策；確認資料後，GovAid 用固定版本的規則試算。最後可下載 PDF，也可讓支援 MCP 的外部助手一起核對案件與文件。

**版本與固定映像：見 `release-manifest.json`** · 支援 Windows、macOS 或 Linux 上的 Docker；映像平台為 `linux/amd64`，ARM 裝置尚未驗證。

## 體驗、影片與文件

[線上體驗 GovAid](https://govaid-web.onrender.com/)；若要在自己的電腦啟動，請依下方 Docker 說明操作。體驗請使用合成資料。

| 內容 | 說明 |
| --- | --- |
| [決賽提案簡報](docs/場域指定命題組_GovAid_Copilot_你愛我對不隊_提案.pdf) | 九頁介紹使用情境、系統架構、主要功能與操作示範。 |
| [場域需求與驗證資料](docs/場域需求與驗證資料.pdf) | 八頁整理三位匿名受訪者的需求與回饋、四案 Agent 評測、租金更正操作紀錄，以及 PDF／MCP 核對結果。 |
| [完整示範影片](https://youtu.be/-GIjQuoCsGA) | Gemini 更正、手機操作、雙城租補 PDF 與 Codex＋MCP 比較。 |
| [精簡示範影片](https://youtu.be/TLEnriSrgNM) | 2 分 30 秒無聲字幕版，依序呈現四個操作情境。 |

### 分段影片

| 片段 | 內容 |
| --- | --- |
| [01 臺北托育](https://youtu.be/Gw5--MtB6Lg) | 更正托育費、確認提案與說明試算原因。 |
| [02 手機操作](https://youtu.be/gyq0kAYSy6Q) | 建立桃園育兒合成案件、查看 PDF 與對話。 |
| [03 雙城租補](https://youtu.be/lNqZWKKS_qk) | 用相同個人條件與月租，比較桃園、臺北的試算與文件。 |
| [04 Codex＋MCP](https://youtu.be/ZduF2-lrxxo) | 逐案授權，核對兩份 PDF，說明兩地補助差異。 |

場域資料的主文可直接閱讀，不必理解程式欄位或案件編號。原始題庫、回答與操作紀錄另存於 PDF 內嵌附件，可用支援附件的 PDF 閱讀器查看。

四案共 40 題的結果是保存回覆重播後的流程檢查，不代表最後版本的模型重新回答 40 題全部正確。真人試用、模型請求與文件核對分開記錄，評測範圍與限制詳見場域資料。

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

目前支援桃園中央租補、臺北中央租補、桃園未滿二歲育兒津貼、臺北友善托育的一般案件。政策版本已由 AI 對照來源，尚未完成獨立人工政策覆核。其他方案尚未提供完整試算。試算只供申辦準備，實際資格與核定金額由主管機關審查；GovAid 不代為送件。

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

## 用 MCP 比較兩案與 PDF

Gemini 協助你在 GovAid 內整理資料；MCP 則讓外部助手取得已授權的案件，將兩案資料與本人提供的 PDF 放在一起核對。MCP 是資料連接工具，本身不是模型；外部助手使用它自己的帳號方案。

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

影片使用兩個獨立租屋情境：小塔的個人、家庭條件與月租 12,000 元相同，分別建立桃園與臺北案。各案核對日期、政策與文件版本，再下載 PDF。影片試算為桃園 2,880 元、臺北 3,600 元，兩筆補助不能合計；前節租金更正為 2,500 元是另一個操作範例。

將兩份 PDF 附給外部助手，並貼上：

```text
請用 GovAid MCP 比較附件的桃園與臺北租補 PDF。
桃園案件 ID：<貼上桃園完整 ID>
臺北案件 ID：<貼上臺北完整 ID>
先分別讀最新快照，再讀各案同 revision 的準備包、指南與政策依據。
核對 PDF 的案件、版本、日期、租金、條件與政策 hash，指出不一致之處。
確認條件可比較後，用表格說明基額、加碼、實租上限及月補助差額，附本案官方來源。
兩案是互斥的租屋選擇，不合計補助，也不只靠補助決定搬家。
未授權先提供連結等我同意；未讀到附件就說 PDF 比對尚未完成。
不要修改案件、代為送件或另呼叫模型 API。
```

這展示個案整理與跨文件核對的分工，沒有宣稱 Codex 比 Gemini 更準確。

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
- [決賽提案簡報](docs/場域指定命題組_GovAid_Copilot_你愛我對不隊_提案.pdf)
- [場域需求與驗證資料](docs/場域需求與驗證資料.pdf)（已獲准公開）。包含三位匿名受訪者的質性回饋、合成案例評測與操作 trace。訪談已同意公開供競賽使用。
- [來源與授權](NOTICE.md)
- 版本與映像清單：`release-manifest.json`。

此 repo 提供 Docker 啟動設定與使用文件，應用程式位於 API、Web、MCP 三個映像內；不是完整原始碼庫。
