# 在手機體驗

手機與執行 Docker 的電腦連同一個可信任 Wi-Fi。預設只供本機使用；以下操作另外開啟手機入口。

**本頁路徑與 IP 都是示意，請換成自己的設定。** 例如 `C:\projects\AAIC2026_GovAid_Demo` 是 repo 位置，`192.168.50.100` 是電腦 Wi-Fi IPv4，`192.168.50.101` 是測試手機 IPv4。用電腦的 `ipconfig` 與手機 Wi-Fi 詳細資訊確認，不要直接沿用範例。埠號以 `.env` 為準，預設 Web 為 13028。

## 開啟

先依 README 啟動服務，再於 repo 根目錄執行：

```powershell
$env:GOVAID_LAN_IP = '192.168.50.100' # 示意：換成電腦的 Wi-Fi IPv4
docker compose -f compose.yaml -f compose.lan.yaml up -d --no-deps api web
if ($LASTEXITCODE -ne 0) { throw '手機入口啟動失敗' }
Write-Output "手機網址：http://${env:GOVAID_LAN_IP}:13028"
```

Windows 防火牆另需一次允許手機連入。以下在系統管理員 PowerShell 執行，先改兩個 IP 和介面名稱。同名規則若已存在，先檢查原設定，不重複建立：

```powershell
New-NetFirewallRule -Name 'GovAid-Demo-WiFi-13028' -DisplayName 'GovAid Demo Wi-Fi 13028' -Direction Inbound -Action Allow -Protocol TCP -LocalPort 13028 -LocalAddress 192.168.50.100 -RemoteAddress 192.168.50.101 -InterfaceAlias 'Wi-Fi' -Profile Any
```

手機以顯示的網址連線。`localhost` 與 `127.0.0.1` 指手機自己，不能用來連電腦。手機或電腦 IP 改變時，須更新 Compose 與防火牆；訪客 Wi-Fi 的裝置隔離也可能阻擋連線。

## 測試

新增匿名人物並重新整理，確認名稱保留。選桃園租補，載入合成範例、儲存試算、核對版本、下載 PDF。開啟聊天，確認鍵盤不遮住輸入與收合操作。只填表不呼叫模型；送聊天會依 `.env` 使用模型配額。

手機有獨立的匿名工作階段，不會自動看到電腦案件。不要複製 Cookie 來共用案件。電腦原網址與資料卷保持不變。

## 分享的範圍與風險

IP 是連線位址，不是密碼。同網段的人可能掃描到服務，沒有公開 IP 也不等於安全。

- HTTP 不加密。區網中的惡意裝置或不可信網路設備可能窺看或竄改傳輸，只用合成資料。
- Web 會代理 API。API 主機埠只綁本機，不代表手機只能看靜態網頁；能連 Web 的人仍可能建立自己的匿名案件、送出聊天並消耗模型額度。Origin 限制不能取代身分驗證。
- 案件憑證與逐案 MCP 授權限制讀取範圍。不要分享 Cookie、配對碼、瀏覽器 state 或含金鑰的 `.env`。撤銷不會收回已下載的 PDF。
- 只在可信任 Wi-Fi 測試，防火牆優先只允許測試手機的 IP。不開路由器轉發、DMZ 或公開隧道。正式對外服務仍需 HTTPS、帳號／存取控制與用量限制。

關閉瀏覽器不會關閉分享。結束時移除區網綁定，再停用防火牆規則。

## 關閉手機分享，電腦繼續使用

在同一 repo 根目錄執行，注意這次**不加入 `compose.lan.yaml`**：

```powershell
docker compose -f compose.yaml up -d --no-deps api web
if ($LASTEXITCODE -ne 0) { throw '關閉分享失敗，請檢查容器狀態' }
docker compose -f compose.yaml port web 3000
```

預設只應列出 `127.0.0.1:13028`，不應再有 Wi-Fi IP 或 `0.0.0.0`。手機關閉舊分頁後重新連線應失敗；電腦仍可用原本機網址。重建 API 可能讓 MCP 需要重新授權，案件資料不會刪除。

再以系統管理員 PowerShell 停用本頁建立的規則：

```powershell
Disable-NetFirewallRule -Name 'GovAid-Demo-WiFi-13028'
```

下次要分享可重新啟用同名規則，但應先確認 IP 是否仍正確。移除區網綁定才是關閉此入口的主要步驟，不只依賴防火牆。

## 全部停止

```powershell
docker compose -f compose.yaml down
```

這會停止此專案服務並移除容器，保留映像與案件資料卷。**不要加 `-v`**。下次依 README 啟動即可，無須重新下載映像。瀏覽器憑證仍有期限，保留資料卷不代表過期憑證可恢復存取。

傳輸安全說明參考：[OWASP TLS 指引](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)。

若沿用先前允許整個子網的同名規則，可在管理員 PowerShell 縮限為單一測試手機。以下 IP 仍為示意，請替換後執行：

```powershell
Set-NetFirewallRule -Name 'GovAid-Demo-WiFi-13028' -LocalAddress 192.168.50.100 -RemoteAddress 192.168.50.101
```
