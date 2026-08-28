---
name: nano4-hpc-inspector
description: Inspect the Nano4 HPC environment through the nano4-proxy SSH alias, including Slurm partitions and account access, active wallet projects, environment modules, and /home or /work quotas. Use when the user asks what Nano4 resources, projects, software, or storage they can currently use.
---

# Nano4 HPC Inspector

Inspect current remote state and report it in a concise, decision-ready form. Treat balances, partitions, modules, and quotas as dynamic; never substitute values copied from an earlier conversation.

## Scope and safety

- Connect with the existing SSH alias `nano4-proxy`. Do not request, print, copy, or store SSH credentials.
- Start with read-only commands. Running `sbatch`, `srun`, `scancel`, changing quotas, loading modules into persistent startup files, or editing remote files requires an explicit user request.
- Confirm `hostname`, `whoami`, and the remote timestamp so results are attributable and current. The expected personal account for this project is `c00cjz00`; flag a different identity instead of silently continuing with account-specific conclusions.
- When invoking SSH from PowerShell, keep the remote payload single-quoted where possible so variables such as `$(whoami)` are expanded remotely rather than locally.
- A slow query is not evidence that no data exists. In particular, allow `hfsquota` time to contact the storage service.

## Local proxy prerequisite

`nano4-proxy` is a local SSH alias backed by a separately running `ssh-proxy` process. The user must authenticate that process to Nano4 with their password and OTP before this skill can connect.

Before attempting `ssh nano4-proxy` or opening a persistent SSH session, check the default local proxy endpoint from PowerShell:

```powershell
Test-NetConnection 127.0.0.1 -Port 2222 -InformationLevel Quiet
```

If the result is `True`, continue with the smallest relevant Nano4 query. This port check only establishes that a local listener exists; still validate the remote host, account, and timestamp after SSH connects.

If the result is `False`:

- Do not repeatedly retry SSH, and do not present the failure as evidence that Nano4 itself is unavailable.
- Ask the user to start the proxy in a terminal they own, complete password/OTP authentication there, and keep that terminal open:

  ```powershell
  .\ssh-proxy-windows-x64.exe nano4
  ```

- If the standalone executable is not installed, direct the user to the official [ssh-proxy Releases page](https://github.com/gemini960114/ssh-proxy/releases). For Windows 10/11 x64, identify `ssh-proxy-windows-x64.exe` as the relevant asset and recommend verifying its SHA-256 checksum against the value on the selected release page.
- Wait for the user to confirm that the proxy is listening before continuing the Nano4 inspection. Communicate these instructions in the user's current conversation language.
- Do not automatically download or execute a release binary without an explicit user request. Never request, enter, capture, print, or store the user's password or OTP.

The proxy is temporary and may stop after its configured no-client idle timeout or maximum lifetime. Check it again for a new task or after a connection failure rather than assuming an earlier proxy process remains available. A user-owned `ssh-proxy` process is distinct from an agent-owned persistent `ssh -tt nano4-proxy` session.

## Persistent interactive SSH

Use one agent-owned interactive SSH session when a sequence of commands benefits from retained shell state, such as the current directory, loaded modules, environment variables, or decisions based on earlier output. For a fixed set of independent checks, prefer one non-interactive call such as `ssh nano4-proxy 'hostname; wallet'` because it has simpler completion and output boundaries.

To open a reusable interactive session, run `ssh -tt nano4-proxy` in a PTY-capable execution tool and retain the returned execution session ID. The doubled `-t` forces remote pseudo-terminal allocation even when the controller's stdin is not a terminal; it is unrelated to `tmux` and is not SSH `ControlMaster` multiplexing.

```bash
ssh -tt nano4-proxy
```

After the remote prompt appears, send commands through that same execution session rather than launching another `ssh` process. For example, send these as three successive inputs:

```text
hostname\n
echo 123\n
date\n
```

On a new session, first confirm `hostname`, `whoami`, and the remote timestamp. Send one logical command at a time when later actions depend on its result. If prompt detection is ambiguous, wrap the command with unique begin/end markers and report its exit status before sending the next command.

An agent cannot take control of an ordinary SSH process that the user opened manually in a separate Windows Terminal; the agent must own the PTY/session handle from creation. The session remains usable only while its local SSH process, the `nano4-proxy` path, and the network connection remain alive. Do not promise survival across task restarts or reconnects. If it dies, create a new session and revalidate identity and state.

Close an agent-owned session cleanly when it is no longer needed:

```text
exit\n
```

This technique reuses one interactive shell but does not replace the existing proxy, create Slurm resources, or broaden permission to run mutating commands. Continue to require explicit user authorization for `sbatch`, `srun`, `scancel`, remote edits, and deletions, and keep computation off the login node.

## Choose the smallest query set

### Slurm resources and project access

Use the relevant subset of:

```bash
ssh nano4-proxy 'hostname; whoami; date --iso-8601=seconds'
ssh nano4-proxy 'wallet'
ssh nano4-proxy 'sinfo -h -o "%P|%a|%l|%D|%t"'
ssh nano4-proxy 'scontrol show partition -o'
ssh nano4-proxy 'sacctmgr -n -P show assoc where user=c00cjz00 format=Cluster,Account,User,Partition,QOS,DefaultQOS'
```

Interpret the results as follows:

- Use `wallet` as the primary list of currently usable projects with positive SU balance.
- Treat `sacctmgr` associations as account history or authorization metadata; an association alone does not prove that a project still has spendable SU.
- `sinfo` shows cluster availability, not user authorization. Cross-check `AllowAccounts`, `DenyAccounts`, partition state, and an active wallet project before saying the user can submit there.
- Partitions may expose the same nodes under different limits. Do not add their idle-node counts unless they are known to represent distinct hardware.
- Report partition, maximum wall time, idle/mixed/down state, and compatible active project IDs. Separate visible-but-restricted partitions.
- Note the Slurm default account when relevant, but recommend explicit `-A PROJECT_ID` and `-p PARTITION` if the default account is absent from `wallet`.

### NGS batch submissions

Treat NGS partition names as lowercase and case-sensitive. Before submitting, check the selected project's current wallet balance and confirm that the partition's `AllowAccounts` includes it. For `ngs62g`, use the queue's full resource profile: one node, one task, 8 CPU cores, and 62 GB of memory.

Use this batch-script pattern for a short `ngs62g` job:

```bash
#!/usr/bin/env bash
#SBATCH --job-name=fastq_qc_stats
#SBATCH --partition=ngs62g
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=62G
#SBATCH --time=00:10:00
#SBATCH --output=logs/fastq_qc_%j.out
#SBATCH --error=logs/fastq_qc_%j.err

sleep 600
```

Create the log directory before submission because Slurm does not create missing parent directories. Specify the active wallet project explicitly at submission time:

```bash
mkdir -p logs
sbatch -A GOV115088 -p ngs62g job.sh
```

If a different project is requested, replace `GOV115088` only after confirming it is active in `wallet` and permitted by `ngs62g`. Do not silently substitute a similarly named project or partition.

### Environment modules

```bash
ssh nano4-proxy 'ml av 2>&1'
```

Group results by cores/toolchains, general tools, biology applications, and other applications. Preserve version numbers and identify entries marked `(D)` as defaults. Suggest `ml -d av`, `ml spider NAME`, or `ml keyword TERM` for narrower follow-up queries.

### Storage and quota

Use:

```bash
ssh nano4-proxy 'hfsquota 2>&1'
ssh nano4-proxy 'df -hT /home /work'
ssh nano4-proxy 'ls -ld /home/c00cjz00 /work/c00cjz00'
```

- `hfsquota` is authoritative for the user's used space, hard limit, usage percentage, and status. It may take more than a minute; wait for its result when possible.
- `df` describes the capacity of the whole WekaFS filesystem and must not be presented as the user's personal allowance.
- Calculate remaining personal space as hard limit minus used bytes. Label binary conversions as GiB or TiB.
- Use `du` only when the user requests directory-level usage; large `/work` trees can take a long time to scan.
- If `hfsquota` fails or times out, report that the personal limit was not confirmed. Do not infer it from `df` or traditional `quota -s` output.

## Reporting

- State the remote host, user, and query time.
- Prefer compact tables for projects, partitions, modules, and quotas.
- Distinguish observed facts from inferred compatibility.
- Include only the commands that help the user reproduce or act on the result.
- Never expose unrelated users, the full cluster association list, credentials, or private environment values.
