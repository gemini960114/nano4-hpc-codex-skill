# Nano4 提問範例

開始前請先在自己的 PowerShell 啟動 `ssh-proxy`、完成 OTP，並保持該視窗開啟。以下問題可直接貼給 Codex。

## 完整唯讀檢查

> 請先檢查 `nano4-proxy` 是否可用；連線後確認 hostname、帳號與遠端時間，再查詢我的 wallet、Slurm partitions 和 account associations，整理出目前真正可以提交工作的 `project + partition` 組合。只做唯讀檢查，不要提交任何工作。

## 連線與身分

- 「請檢查 `nano4-proxy` 是否可用，並確認遠端 hostname、帳號與時間。」
- 「請確認目前連到的是不是 Nano4，而且 `whoami` 與我的 SSH config 帳號一致。」
- 「如果本機 proxy 尚未啟動，請提供下載與啟動方式；不要自行下載或處理 OTP。」

## Slurm 資源

- 「請查看 Nano4 目前有哪些 Slurm partition、最長執行時間及節點狀態。」
- 「請區分我可以使用的 partition 與可見但受限制的 partition。」
- 「請查詢 `ngs62g` 的狀態、資源規格及允許使用的 project。」
- 「哪些 partition 適合執行需要 8 CPU、62 GB RAM、10 分鐘的 NGS 工作？」
- 「請比較各 partition 的時間限制及 idle、mixed、down 狀態，不要把重疊節點重複加總。」

## Project 與 SU 餘額

- 「請使用 `wallet` 查詢我目前仍有 SU 餘額的 project。」
- 「請交叉比對 wallet、Slurm associations 與 partition 限制，找出真正可提交的組合。」
- 「我的預設 Slurm account 是否還有可用餘額？若沒有，建議明確指定哪個 `-A`？」
- 「我現在可以用哪個 project 提交到 `ngs62g`？只查詢，不要提交。」

`sacctmgr` association 只代表授權或歷史資料，不能單獨證明 project 仍有 SU；判斷可用 project 時應以目前的 `wallet` 結果為主。

## Modules 與軟體

- 「請列出 Nano4 目前可用的生物資訊 modules 與版本。」
- 「請查詢是否有 FastQC、MultiQC、BWA、SAMtools 或 GATK。」
- 「請找出名稱包含 `blast` 的 modules，並標示預設版本。」
- 「請確認可用的 Python、compiler 與 CUDA modules，但不要修改 startup files。」

## 儲存空間

- 「請使用 `hfsquota` 查詢我在 `/home` 與 `/work` 的個人 quota、已用及剩餘空間。」
- 「我的個人儲存空間是否接近上限？請不要用 `df` 推測個人 quota。」
- 「請區分個人 quota 與整個共享檔案系統的容量。」
- 「請根據遠端 `whoami` 確認我的 `$HOME` 與 `/work/USERNAME` 是否存在及其權限，但不要掃描整個目錄。」

## 產生腳本但不提交

- 「請根據目前可用的 project 產生一份 `ngs62g` 的 10 分鐘測試腳本，但不要提交。」
- 「請產生 FastQC 的 Slurm script，明確指定 `-A PROJECT_ID` 與 `-p PARTITION`，但不要建立遠端檔案。」
- 「請檢查我提供的 batch script 是否符合目標 partition 的 CPU、memory 與 wall-time 限制，不要執行。」

這些問題應使用 `nano4-slurm-job-runner`，但只授權本地草擬或驗證，不包含遠端 setup、上傳或提交。

## Job 狀態與持續監控

- 「使用 `$nano4-hpc-inspector` 查看 Job `307732` 現在的狀態，只查詢一次。」
- 「使用 `$nano4-slurm-job-runner` 每分鐘監控 Job `307732` 直到完成；這是唯讀監控，不要取消、requeue、修改檔案或重新提交。」
- 「使用 `$nano4-slurm-job-runner` 幫我寫 Slurm 腳本但不要提交；只能在本地草擬。」
- 「使用 `$nano4-slurm-job-runner` 提交這個 Job 並監控到完成；只授權指定工作目錄的正常 setup 與單次提交。」

## 需要明確授權的操作

以下行為會改變遠端狀態，問題中必須清楚指定：

- 使用 `sbatch` 提交工作。
- 使用 `srun` 建立互動式或計算工作。
- 使用 `scancel` 取消工作。
- 建立、修改、移動或刪除遠端檔案及目錄。
- 修改 shell startup files 或永久環境設定。

明確授權的問題範例：

> 請先確認 project `PROJECT_ID` 仍有 SU，且允許使用 partition `PARTITION`；確認後建立 `logs` 目錄，將我提供的腳本寫入遠端並使用 `sbatch -A PROJECT_ID -p PARTITION job.sh` 提交。提交完成後回報 job ID。

> 請先顯示 job ID `123456` 的擁有者與目前狀態；確認它屬於目前的 `whoami` 後，使用 `scancel 123456` 取消並重新查詢狀態。

## 完整 Job lifecycle

- 「請使用 `$nano4-slurm-job-runner`，先重新驗證 wallet、partition、QoS 與資源限制，再於我指定的工作目錄建立腳本並提交一次。請回報 Job ID，使用結構化 `squeue` 和 `sacct` 監控到完成，最後檢查 stdout、stderr、ExitCode、Elapsed 與 MaxRSS。」
- 「請提交後只監控到 job 進入 running 狀態便停止，不要取消、重送或清理檔案。」
- 「這次只診斷 pending 原因，不要變更資源、requeue、cancel 或重新提交。」

若 `sbatch` 回應中斷或結果不明，Runner 應先用 job name、使用者與提交時間查詢是否已建立工作，不可直接重送。

## Nano4 檔案傳輸

- 「請先確認遠端目標不存在，再使用 Nano4 相容的 `scp -O` 將 `job.sh` 上傳到我指定的工作目錄；不要提交工作。」
- 「請使用 `scp -O` 下載指定的 stdout 檔案到目前資料夾，不要下載其他結果或目錄。」

當目前的 `nano4-proxy` login path 拒絕 SFTP subsystem，或一般 `scp` 出現 `subsystem request failed` 時，使用 `scp -O`。這不代表 Nano4 的 dedicated file-transfer service 不支援 SFTP，也不應當成所有 SSH 主機的通用設定。

不要使用模糊說法，例如「幫我處理這個 job」。若只想診斷，請明確寫「只查詢與說明，不要提交、取消或修改任何內容」。
