# Anti-Gravity 懶人包 #09：服務連接與工作流程設定

> 版本：v1.5 (實戰優化版)
> 更新日期：2026-07-02
> 語系偏好：繁體中文（Taiwan）

這份懶人包旨在引導 Anti-Gravity 用戶與 AI 協同工作時，安全、順暢地連接 NotebookLM、Firebase、GitHub、Obsidian，並實施開工、收工與專案初始化的標準作業。此版本基於實戰測試，針對 Windows 環境的權限問題與 AI 執行限制進行了優化。

---

## 🛠️ 一、基礎開發環境與 Windows 特殊處理

在 Windows 環境中，為避免 PowerShell 執行原則（Execution Policy）阻擋 `.ps1` 腳本（例如直接執行 `npm` 或 `npx` 失敗），請遵循以下規範：

### 1. 指令替代方案
*   **不要直接執行** `npm` 或 `npx`。
*   **務必改用** `npm.cmd` 與 `npx.cmd` 來繞過權限阻擋。

### 2. 環境檢測指令
```powershell
git --version
gh --version
node --version
npm.cmd --version
uv --version
```
*提示：若缺乏 `uv` 或 `node`，建議使用 `winget install astral-sh.uv` 或 `winget install OpenJS.NodeJS.LTS` 快速安裝。*

---

## 📂 二、各項服務連接指引

### 1. NotebookLM MCP 連接
*   **安裝指令**（建議使用 `uv`，速度最快且不污染全域環境）：
    ```powershell
    uv tool install notebooklm-mcp-cli
    ```
*   **登入驗證**：
    ```powershell
    # 請確保在可調用系統瀏覽器的環境下執行，這會啟動 Chrome 完成 Google OAuth 登入
    $env:PATH = "C:\Users\wzw11\.local\bin;" + $env:PATH
    nlm login
    ```
*   **驗證與檢查**：
    ```powershell
    nlm doctor
    nlm list notebooks
    ```
*   **MCP 註冊設定**：
    ```json
    "notebooklm": {
      "type": "local",
      "command": ["nlm", "mcp"],
      "enabled": true
    }
    ```

### 2. Firebase 連接
*   **環境與版本確認**：
    ```powershell
    npx.cmd -y firebase-tools@latest --version
    ```
*   **登入驗證（⚠️ 重要限制）**：
    *   **限制**：AI 的背景執行環境為非互動式（non-interactive），直接執行 `firebase login` 會報錯失敗。
    *   **手動步驟**：使用者**必須手動在自己的實體 PowerShell 視窗中執行** `npx firebase-tools login` 來完成 Google 授權，AI 之後方可接手。
*   **MCP 註冊設定**：
    ```json
    "firebase": {
      "type": "local",
      "command": ["npx.cmd", "-y", "firebase-tools@latest", "mcp"],
      "enabled": true
    }
    ```

### 3. GitHub & Git 設定
*   **GitHub CLI 登入**：
    ```powershell
    gh auth login --web --git-protocol https
    ```
    *如果在背景執行，AI 會輸出單次驗證碼（One-time code），使用者需前往 https://github.com/login/device 輸入授權。*
*   **Git 全域使用者設定（防錯必做）**：
    為了防止 Git Commit 時因身分未知而中斷，請確實設定使用者名稱與信箱：
    ```powershell
    git config --global user.name "您的GitHub帳號名稱"
    git config --global user.email "您的GitHub電子郵件"
    ```

### 4. Obsidian MCPVault 連接
*   **安裝 MCPVault**：
    ```powershell
    npm.cmd install -g @bitbonsai/mcpvault
    ```
*   **註冊與路徑確認**：
    由於 AI 無法自動猜測您的資料夾位置，請手動確認 Vault 根目錄（需包含 `.obsidian` 資料夾）。
*   **MCP 註冊設定**（請將路徑替換為您的實際路徑）：
    ```json
    "obsidian": {
      "type": "local",
      "command": [
        "C:\\Users\\<您的用戶名>\\AppData\\Roaming\\npm\\mcpvault.cmd",
        "C:\\Users\\<您的用戶名>\\Documents\\<您的Vault名稱>"
      ],
      "enabled": true
    }
    ```

---

## 🎨 三、AI 內建生圖技能整合

**不要**在專案中保存任何 OpenAI 等外部生圖 API 金鑰。Anti-Gravity 具備強大的內建生圖工具，請直接以自然語言命令 AI 進行圖像生成與後製。

### 1. AI 內建生圖指令範本
當需要產生圖片時，請直接在對話中呼叫：
> **「幫我生成一張圖片，主題是：[主題]，用途是：[用途]，色彩風格為：[風格]，比例為：[16:9 / 1:1]，並儲存至專案的 assets/ 目錄。」**

### 2. 生圖管理規範
*   所有的生成圖片均需存放在專案專屬的 `assets/` 目錄或 Obsidian 附件夾中。
*   避免將臨時生成的草圖、測試圖提交（Commit）至公開的代碼庫。

---

## 🔄 四、開工 / 收工與專案初始化技能

為了維持 AI 的記憶連貫並節省 Token 消耗，請嚴格遵守「開工、收工與初始化」SOP。

### 1. 開工 (Start Work)
當對對話發起「開工」時，AI 將執行以下動作：
1.  讀取專案根目錄的 `ANTIGRAVITY.md` 規則檔案。
2.  讀取 Obsidian 中的專案駕駛艙。
3.  執行 `git status` 與最近變更檢查。
4.  摘要當前進度，並向使用者回報下一步建議。

### 2. 收工 (Finish Work)
當對對話發起「收工」時，AI 將執行以下動作：
1.  **安全檢查**：自動掃描並排除 API Keys、個人憑證與敏感資料。
2.  **更新記錄**：將本日完成事項、未完成事項與下一步規劃寫入 Obsidian 駕駛艙。
3.  **藍圖更新**：若專案邊界或設定有變動，更新 `ANTIGRAVITY.md`。
4.  **代碼提交**：執行 `git diff` 審查，僅 stage 本次相關變更，禁止使用 `git add .` 無差別提交，隨後 Commit 並 Push 至遠端儲存庫。

### 3. 專案初始化
當發起「新專案初始化」時，AI 應主動引導使用者提供：專案名稱、用途、工作目錄與是否建立 GitHub 庫，並為其自動生成 `.gitignore`、`README.md`、`ANTIGRAVITY.md` 與 Git 初始化設定。

---

## 📝 五、ANTIGRAVITY.md 藍圖範本

請於專案根目錄建立 `ANTIGRAVITY.md`：
```markdown
# <專案名稱> - ANTIGRAVITY.md

## 專案入口
- 專案用途：
- 主要工作目錄：
- GitHub Repo: 
- 預設 Branch: main

## Obsidian 對應筆記
- 專案駕駛艙路徑：

## 本地工作規則
- 回應預設使用繁體中文 (Taiwan)。
- 涉及檔案操作時必回報完整路徑。
- Windows 指令改用 `*.cmd`。
- 開工時讀取本檔與駕駛艙，收工時更新駕駛艙並清理敏感資料後提交。
- 不得自動 commit 無關檔案。
```
