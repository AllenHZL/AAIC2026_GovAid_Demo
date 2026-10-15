# MCP：App 設定一次，每案網頁同意

先啟動 Docker Compose，確認 API 與網頁健康。MCP 容器與 API 共用網路命名空間，所以仍走 `127.0.0.1:8000`；這不是把 API 放上網際網路。

App 新增或修改 `govaid-readonly`，選標準輸入／輸出：

- 啟動指令：`docker`（Windows 找不到時填 Docker Desktop 的 `docker.exe` 完整路徑）。
- 工作目錄：啟動包完整路徑，例如 `C:\coding_project\GovAid_Demo`。
- 引數逐項填入：

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

路徑若不同，三處一起更換。清除舊 Python/server.py 引數、MCP API/WEB 環境變數與舊配對碼。環境變數與透傳清單留空；**不要加入任何模型金鑰**。Compose 只把 API/WEB 位址傳給 MCP，並不把 `.env` 整份掛進容器。

儲存並重連。請 App 列出實際工具，再呼叫 `list_policy_versions`。應有八個：`list_policy_versions`、`get_policy_details`、`search_policy_sources`、`begin_case_authorization`、`finish_case_authorization`、`get_case_snapshot`、`get_preparation_package`、`get_application_guide`。

讀案時 App 先要求授權，回傳網址與核對字串。使用建立案件的同一瀏覽器開啟網址，核對案件後勾選同意。回到 App 告知已同意，再取得最新版本及同版包／指南。授權到期或換案件，重做網頁授權即可。API 容器重建後，重新連線 MCP。

自檢（不呼叫模型）：

```powershell
docker compose run --rm --no-deps -T mcp --self-check
```

應回 `ok: true`、`model_mode: live` 與政策數。MCP 不會建立、修改、核對或送出案件；它提供證據，App 根據證據回答。
