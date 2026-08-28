# Project Guidance

## Nano4 HPC

- This project connects to its remote Nano4 environment through the local SSH alias `nano4-proxy`. The alias depends on a separately running, OTP-authenticated `ssh-proxy` process. The expected remote personal account is `c00cjz00`.
- Before attempting `ssh nano4-proxy` or `ssh -tt nano4-proxy`, check whether the default local proxy endpoint is listening with `Test-NetConnection 127.0.0.1 -Port 2222 -InformationLevel Quiet`. If the result is `False`, do not repeatedly retry SSH or report Nano4 itself as unavailable.
- When the local proxy is not listening, ask the user to start `ssh-proxy` in a user-owned terminal, complete password/OTP authentication there, keep that terminal open, and notify the agent when the proxy is ready. If the standalone program is not installed, direct the user to the [ssh-proxy Releases page](https://github.com/gemini960114/ssh-proxy/releases) and identify `ssh-proxy-windows-x64.exe` as the Windows 10/11 x64 asset. The user should verify its SHA-256 checksum against the selected release page before running it.
- Do not automatically download or execute an `ssh-proxy` release binary without an explicit user request. Never request, enter, capture, print, or store the user's password or OTP.
- Treat the local proxy as temporary: it may stop after its configured idle timeout or maximum lifetime. Do not assume a proxy from an earlier task is still running. The local proxy process is distinct from an agent-owned persistent SSH session.
- When several Nano4 commands benefit from the same remote working directory or shell environment, an agent may open one agent-owned PTY with `ssh -tt nano4-proxy` and reuse that execution session for subsequent commands. Do not claim or attempt to attach to an SSH process that the user opened manually in another terminal.
- After opening a persistent SSH session, confirm `hostname`, `whoami`, and the remote time; send commands through the retained session; and close it with `exit` when it is no longer needed. Do not assume the session survives a task restart, process loss, or network interruption.
- A persistent SSH session does not broaden authorization. Keep login-node work lightweight, use Slurm for computation, and retain the explicit approval requirements below for remote mutations and job control.
- For Nano4 Slurm resources, wallet projects, environment modules, or storage quotas, use the project skill at `.agents/skills/nano4-hpc-inspector/SKILL.md` and read it completely before acting.
- Treat project balances, partition availability, module versions, and storage quotas as dynamic. Query the remote system again instead of relying on previously reported values.
- Use `wallet` to identify active projects with SU balance. Do not treat `sacctmgr` associations alone as evidence that a project remains usable.
- Use `hfsquota` for personal `/home` and `/work` limits. `df` describes the shared filesystem and is not a personal quota.
- Slurm examples and submissions should specify both `-A PROJECT_ID` and `-p PARTITION` unless the user explicitly chooses another convention.
- Default to read-only inspection. Do not run `sbatch`, `srun`, `scancel`, alter remote startup files, or edit/delete remote data unless the user explicitly requests that action.
- Do not store or expose SSH credentials, tokens, passwords, or unrelated users' cluster information.
