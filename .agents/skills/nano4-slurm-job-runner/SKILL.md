---
name: nano4-slurm-job-runner
description: Prepare, submit, monitor, and report explicitly authorized Nano4 Slurm jobs through nano4-proxy. Use for remote job setup, sbatch execution, job lifecycle monitoring, authorized job control, or job-related file transfer; do not infer mutation permission from an inspection request.
---

# Nano4 Slurm Job Runner

Run an authorized Nano4 Slurm workflow from preflight through final reporting. Keep authorization boundaries explicit and prevent duplicate or misdirected submissions.

## Authorization and safety

- Use this skill only when the user explicitly requests a remote mutation such as job setup, submission, file transfer, interactive allocation, or job control.
- A clear submission request authorizes normal setup for that named job in the agreed working directory. It does not authorize overwriting unrelated files, deleting data, cancelling jobs, modifying startup files, or submitting additional retries.
- `scancel`, destructive replacement, deletion, and resubmission after a failed or ambiguous attempt each require explicit authorization for the exact target.
- Never request, enter, capture, print, or store passwords, OTP values, SSH keys, or tokens. The user owns and authenticates the local `ssh-proxy` process.
- Keep login-node activity limited to inspection, file preparation, submission, and lightweight result review. Run computation through Slurm.
- Stop before mutation if identity, target path, project, partition, QoS, requested resources, or authorization is ambiguous.

## Required workflow

Before preparing or submitting a job, read [references/job-lifecycle.md](references/job-lifecycle.md) completely and follow its preflight, single-submission, monitoring, and reporting rules.

If the task needs upload or download, also read [references/file-transfer.md](references/file-transfer.md) completely before transferring anything. File-transfer authorization does not imply job submission, deletion, or overwrite permission.

## Proxy and identity

Check the default local proxy endpoint before SSH:

```powershell
Test-NetConnection 127.0.0.1 -Port 2222 -InformationLevel Quiet
```

If it is unavailable, ask the user to start `ssh-proxy` and complete password/OTP authentication in their own terminal. Do not repeatedly retry or treat a closed local port as proof Nano4 is down.

At the beginning of a new remote session, confirm:

```bash
ssh nano4-proxy 'hostname; whoami; date --iso-8601=seconds'
```

Compare the observed user with the project-level expected account before any account-specific mutation.

## Structured execution

- Prefer non-PTY commands with explicit fields for preflight, submission, monitoring, and accounting.
- Use `sbatch --parsable`, pipe-delimited `squeue -o`, and `sacct -P` so identifiers and state are not inferred from terminal formatting.
- Keep stdout, stderr, command exit status, Slurm Job ID, and application output paths distinct.
- Use a retained PTY only when shell state is genuinely needed. Wrap commands in unique begin/end markers and record exit status when parsing PTY output.
- Never retry `sbatch` merely because its response was interrupted or ambiguous. Query Slurm for a matching job first and ask the user before any possible duplicate submission.

## Dynamic resource validation

- Obtain the current `PROJECT_ID` from `wallet`; never use a hard-coded or previously reported project balance.
- Recheck partition state, `AllowAccounts`, `DenyAccounts`, QoS, node/task/CPU shape, memory, and wall-time limits immediately before submission.
- Treat `ngs62g` and any resource profile as an example only. Its current configuration must be verified before generating directives or submitting.
- Specify both `-A PROJECT_ID` and `-p PARTITION` for every submission unless the user explicitly requests another convention.

## Completion

- Follow the job until the user-requested stopping condition: accepted, running, terminal, or another clearly stated point.
- For terminal jobs, inspect Slurm state, exit code, elapsed time, available resource metrics, and the named stdout/stderr files.
- Report the observed Job ID, project, partition, working directory, final state, exit code, output paths, and any missing or inconclusive metrics.
- Do not claim success solely because `sbatch` accepted the job.
