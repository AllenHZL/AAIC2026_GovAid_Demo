# GovAid 決賽 Demo

GovAid 協助整理育兒與租屋補助條件、試算結果和申辦文件。本庫是競賽展示與重現入口，提供 Docker 啟動設定；API、網頁和 MCP 程式在三個映像內。完整開發原始碼另行管理，本庫不是完整原始碼鏡像。

**版本：`finals-ee3b321-20261109`**（2026-11-10 發布） · 程式基準 `ee3b3212e00ef5da4ea8f30f2a3675bbccd618b6`。Compose 固定映像 digest；詳細版本與檔案雜湊見 [release-manifest.json](release-manifest.json)。保留舊初賽 tag，不覆寫舊版。

## 啟動

需要 Docker Desktop（Linux containers）與可用網路。不必另裝 Python 或 Node.js。此版支援 `linux/amd64`；ARM 裝置尚未驗證。請使用合成資料，不要對外開放連接埠。

```powershell
git clone https://github.com/AllenHZL/AAIC2026_GovAid_Demo.git
cd AAIC2026_GovAid_Demo
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
```

在 VS Code 編輯 `.env`，把自己免費專案的 Gemini key 填入 `MODEL_API_KEY`。保留 `MODEL_MODE=live` 與 `MODEL_NAME=gemini-3.5-flash-lite`。不要把 `.env` 傳給隊友、提交 Git，或把 key 填入 MCP。未有 key 時可設 `MODEL_MODE=mock` 測表單；mock 不代表已驗證 Gemini 回答。

```powershell
docker compose --profile mcp pull
if ($LASTEXITCODE -ne 0) { throw '映像下載失敗' }
docker compose up -d --wait db api worker web
if ($LASTEXITCODE -ne 0) { throw '啟動失敗，請查看 docker compose logs --tail=50' }
docker compose ps -a
Start-Process 'http://127.0.0.1:13028'
```

開啟 [GovAid 網頁](http://127.0.0.1:13028)。設定應顯示 live 與模型名稱；開啟網頁、表單試算、PDF 及 MCP 讀取不會呼叫 Gemini。聊天中的更正解析與有來源問答可能呼叫模型。請使用未開啟計費的免費專案；本程式不替帳號設定預算，也不保證其他帳號不計費。

`migrate`、`seed` 完成後顯示 Exited (0) 是正常的。API／Web／DB 應健康，Worker 應運行。網頁固定 `13028`，API 固定 `18028`；資料庫不開放主機連接埠。

## 使用方式

| 工作區 | 可以做什麼 |
| --- | --- |
| 01 需求與問答 | 選方案、載入合成範例、開啟聊天、查看案件來源。 |
| 02 確認條件 | 查看 Gemini 提案；連續補完缺項，最後儲存一次。 |
| 03 結果與比較 | 查看試算與假設差異，核對目前版本。 |
| 04 我的資料包 | 用案件代號找卡牌、開啟指南、下載 PDF、管理外部助手授權。 |

四個方案：桃園中央租補、臺北中央租補、桃園未滿二歲育兒津貼、臺北友善托育。其餘方案仍在開發。這是有固定政策版本與適用範圍的申辦準備工具，不是政府核定或自動送件服務。

### 用小塔走一次

1. 選桃園租補，載入合成範例，核對後儲存。月租 12,000 元，試算 2,880 元。
2. 在聊天輸入：`如果租金改成2500元，會領多少？不要更正案件。` 原條件應保持不變。
3. 輸入：`更正租約月租金為2500元，原先12000元填錯，其他條件不變。` 查看提案，補齊需重新核對的欄位，再儲存。
4. 輸入：`為什麼本案試算只有2500元？請用白話說明實際租金如何限制補助，附本案官方來源。`
5. 核對目前版本，從資料包卡牌開指南並下載 PDF。月租與試算均為 2,500 元。
6. 如需跨文件複核，再使用下方 MCP。新增訊息或更正可能增加案件版本；最後一問完成後再下載 PDF。

範例使用固定日期的合成情境，不代表今天的正式資格。Gemini 提案需本人確認；金額由規則計算。回答若標示系統備援，請勿視為 Gemini 成功回答。

## MCP：App 設定一次，每案自行同意

MCP 可與 live Gemini 同時運作。它只讀取案件，不帶模型 key、不呼叫 Gemini，也不代送件。外部 App 的回答使用該 App 自己的帳號方案；MCP 本身不是語言模型。

先確認容器已啟動：

```powershell
docker compose run --rm --no-deps -T mcp --self-check
```

應顯示 `ok: true`、`model_mode: live`。設定支援本機 stdio 的 App：名稱 `govaid-readonly`，啟動指令 `docker`，工作目錄填你的 repo 完整路徑。引數逐項填入（下列三個路徑請一起換成自己的）：

```text
compose
--project-directory
C:\coding_project\GovAid_Demo
--env-file
C:\coding_project\GovAid_Demo\.env
-f
C:\coding_project\GovAid_Demo\compose.yaml
run
--rm
--no-deps
-T
mcp
```

環境變數與透傳清單留空，不填模型 key 或舊配對碼。儲存並重連，應看見八個工具：列政策、政策詳情、來源搜尋、開始授權、完成授權、案件快照、準備包、申辦指南。

請 App 讀指定案件，它會提供授權連結與核對字串。回建立案件的同一瀏覽器，確認後由本人同意，再請 App 繼續。若要比對 PDF，需另外把 PDF 附給 App；MCP 不會任意讀取電腦檔案。

可複製的複核問題：

```text
請用 GovAid MCP 複核附件 PDF。案件 ID：<貼上卡牌完整 ID>。
舊筆記：「可以領2880元，而且已核准，所以不用再準備租約。」
先讀最新案件，再取同 revision 的準備包與指南；核對 PDF 的案件、版本、租金、試算和政策 hash。
搜尋本案政策的「實際租金」和「租賃契約」。指出筆記錯誤，附官方來源，列三項下一步。
未授權先等我同意；沒有讀到附件就說未完成 PDF 比對。不修改案件、不送件、不另呼叫模型 API。
```

授權可在資料包的「管理授權」撤銷。撤銷阻止後續讀取，不會收回已下載 PDF 或 App 已讀過的文字。重建 API 容器後需重新連線 MCP；每案授權也有期限。

## 更新、停止與資料保留

已有此 repo：先保留 `.env`，再執行 `git pull --ff-only`、`docker compose --profile mcp pull`、`docker compose up -d --wait db api worker web`。新版 Compose 已固定映像，不沿用舊 `.env` 的 `GOVAID_*_IMAGE` 值。不要同時啟動兩份使用相同連接埠的啟動包。

```powershell
# 停止；保留資料卷與案件
docker compose down
# 下次啟動
docker compose up -d --wait db api worker web
```

不要加 `-v`，除非確定要清空所有案件。資料庫仍有案件，不代表瀏覽器永遠有權讀取：匿名存取預設八小時閒置／二十四小時上限，可自行選擇保留七天。請維持同一瀏覽器、網址與資料卷；憑證失效不能只靠案件 ID 找回。介面可分別移除本機紀錄或刪除本人案件。

若埠被占用，先用 `docker ps` 確認是否已有 GovAid 在運行，勿直接刪除資料卷。若 PDF 仍是舊版，先完成最後更正、重新核對，再下載。若 Gemini 達免費額度，保留錯誤紀錄，改用表單繼續；不要為測試開啟付費。

## 架構與資料

- [整體架構](docs/architecture.md)
- [來源與授權](NOTICE.md)
- [場域需求與驗證資料](docs/場域需求與驗證資料.pdf)：三位匿名受訪者摘要，已同意公開；不代表大規模成效驗證。

此庫用作比賽展示與重現入口。主辦另列的原始碼、評測、trace 及簡報項目，仍由團隊按繳件方式提供；不以映像啟動庫宣稱所有繳件項目已齊備。
