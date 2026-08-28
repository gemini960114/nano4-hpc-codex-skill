# Nano4 連線與設定

## 連線架構

```text
使用者持有的 PowerShell
  └─ ssh-proxy（完成密碼與 OTP 驗證）
       └─ 127.0.0.1:2222
            └─ SSH alias: nano4-proxy
                 └─ Nano4
```

本機 `ssh-proxy` 與代理建立的 `ssh nano4-proxy` session 是兩個不同程序。使用者需保持 proxy 視窗開啟；代理不會接管該視窗，也不應要求或處理密碼及 OTP。

## 1. 下載執行檔

前往 [ssh-proxy Releases](https://github.com/gemini960114/ssh-proxy/releases)，Windows 10/11 x64 請下載：

```text
ssh-proxy-windows-x64.exe
```

下載後，對照所選 Release 頁面公布的 SHA-256：

```powershell
Get-FileHash .\ssh-proxy-windows-x64.exe -Algorithm SHA256
```

不要從不明來源下載同名程式。Release 版本與雜湊值會更新，因此本文件不固定寫入 `latest` 的雜湊值。

## 2. 設定 SSH aliases

將以下設定放入 `$env:USERPROFILE\.ssh\config`。若已有同名設定，請先確認內容，避免重複宣告。

```sshconfig
Host nano4
  HostName nano4.nchc.org.tw
  User c00cjz00

  PubkeyAuthentication no
  KbdInteractiveAuthentication yes
  PreferredAuthentications keyboard-interactive,password

  ServerAliveInterval 30
  ServerAliveCountMax 3
  IPQoS none

Host nano4-proxy
  HostName 127.0.0.1
  Port 2222
  User c00cjz00

  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  LogLevel ERROR

  ServerAliveInterval 30
  ServerAliveCountMax 3
```

`StrictHostKeyChecking no` 與 `UserKnownHostsFile /dev/null` 只能套用於本機的 `nano4-proxy` alias，不可套用於真正的遠端 host `nano4`。本機 proxy 每次啟動可能產生新的暫時 host key；真正 Nano4 的 host key 仍應嚴格驗證。

## 3. 啟動 proxy

在執行檔所在目錄開啟 PowerShell：

```powershell
.\ssh-proxy-windows-x64.exe nano4
```

依畫面提示完成密碼與 OTP 驗證。首次連到真正的 Nano4 時，應透過可信任的 NCHC 來源確認 SSH host-key fingerprint；遇到 host key 變更警告時，不要直接刪除舊紀錄來繞過驗證。

proxy 成功開始監聽後，保持此視窗開啟。

## 4. 檢查與連線

從另一個 PowerShell 檢查預設 port：

```powershell
Test-NetConnection 127.0.0.1 -Port 2222 -InformationLevel Quiet
```

- `True`：本機已有 listener，可以繼續嘗試 SSH。
- `False`：先檢查 proxy 視窗；這不代表 Nano4 本身故障。

連線並確認遠端身分：

```powershell
ssh nano4-proxy 'hostname; whoami; date --iso-8601=seconds'
```

本專案預期遠端帳號為 `c00cjz00`。若顯示其他帳號，應停止帳號相關查詢並先檢查 SSH 設定。

## 5. 與 Codex 協作

完成 OTP 後，可以告訴 Codex：

> 我已啟動 `nano4-proxy` 並完成 OTP。請先確認 hostname、whoami 與遠端時間，再執行唯讀查詢。

若 proxy 尚未啟動，Codex 應先檢查本機 port，提供啟動或下載指引，並等待使用者完成驗證，而不是反覆嘗試 SSH。

## 連線時效

proxy 具有 no-client idle timeout 與 maximum lifetime；實際預設值以所用 Release 的說明為準。即使先前曾成功連線，新任務開始或連線失敗後仍應重新檢查本機 port。

## 常見問題

### Port 2222 回傳 `False`

確認 proxy 程序仍在執行，並查看原 PowerShell 是否顯示 idle timeout、maximum lifetime 或遠端連線中斷。需要時重新啟動 proxy 並完成 OTP。

### `nano4-proxy` 找不到

檢查 `$env:USERPROFILE\.ssh\config` 是否包含 `Host nano4-proxy`，並確認設定檔名稱不是誤存成 `config.txt`。

### Remote host identification changed

不要為了繼續連線而直接移除真正 Nano4 的 host key。先向 NCHC 管理者或其他可信來源核對新的 SHA-256 fingerprint。

### Proxy 可連線但 Codex 查詢失敗

先用下列最小查詢確認身分與時間：

```powershell
ssh nano4-proxy 'hostname; whoami; date --iso-8601=seconds'
```

若 proxy 已失效，重新啟動並完成 OTP；若身分不符，先修正 SSH config。
