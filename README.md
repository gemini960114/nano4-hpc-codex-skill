# Nano4 HPC Workspace

本專案用於透過 Codex 查詢與操作 Nano4 HPC。連線經由本機 `ssh-proxy` 完成一次密碼與 OTP 驗證，之後終端機或代理可使用 SSH alias `nano4-proxy` 連線。

## 快速開始

1. 從 [ssh-proxy Releases](https://github.com/gemini960114/ssh-proxy/releases) 下載 Windows 10/11 x64 的 `ssh-proxy-windows-x64.exe`。
2. 對照所選 Release 頁面公布的 SHA-256，確認下載檔案：

   ```powershell
   Get-FileHash .\ssh-proxy-windows-x64.exe -Algorithm SHA256
   ```

3. 在自己的 PowerShell 啟動 proxy：

   ```powershell
   .\ssh-proxy-windows-x64.exe nano4
   ```

4. 自行完成密碼與 OTP 驗證，並保持該 PowerShell 視窗開啟。
5. 從另一個終端機確認本機 proxy 已開始監聽：

   ```powershell
   Test-NetConnection 127.0.0.1 -Port 2222 -InformationLevel Quiet
   ```

6. 結果為 `True` 後即可連線：

   ```powershell
   ssh nano4-proxy
   ```

密碼與 OTP 只應在使用者持有的 proxy 終端機輸入，不要貼到對話、程式碼或紀錄檔中。

## 文件

- [Nano4 連線與設定](docs/nano4-connection.md)
- [可直接使用的提問範例](docs/example-questions.md)
- [最近產生的環境報告](reports/)
- [代理操作規則](AGENTS.md)

## 使用原則

- Slurm partition、project SU 餘額、module 版本及 quota 都是動態資訊；需要時重新查詢。
- 查詢預設為唯讀。提交或取消工作、啟動互動工作，以及修改或刪除遠端檔案，都必須在問題中明確授權。
- Slurm 提交範例應同時指定 project 與 partition，例如 `sbatch -A PROJECT_ID -p PARTITION job.sh`。
- 登入節點只適合輕量檢查，實際計算應交給 Slurm。
