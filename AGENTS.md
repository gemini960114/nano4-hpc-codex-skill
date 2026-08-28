# Project Guidance

## Nano4 HPC

- Connect through the local SSH alias `nano4-proxy`. The expected remote personal account for this project is `c00cjz00`; stop before account-specific actions if the observed identity differs.
- The user owns the separately running `ssh-proxy` process and completes password/OTP authentication in their own terminal. Never request, enter, capture, print, or store credentials, OTP values, SSH keys, or tokens.
- Use `nano4-hpc-inspector` for read-only resource inspection and one-time job-status queries.
- Use `nano4-slurm-job-runner` for requested job preparation, authorized remote setup or submission, ongoing lifecycle monitoring, job control, and job-related file transfer.
- Monitoring an existing job is read-only and does not authorize cancellation, requeueing, file changes, or resubmission.
- Default to read-only inspection. Remote writes, `sbatch`, `srun`, file upload, and job control require explicit authorization. `scancel`, deletion, overwrite, and possible duplicate resubmission require authorization for the exact target.
- Treat wallet balances, partition state and limits, QoS, account access, module versions, job state, and quotas as dynamic. Requery them instead of relying on earlier reports or hard-coded examples.
- Keep login-node work lightweight and run computation through Slurm. Specify both `-A PROJECT_ID` and `-p PARTITION` for submissions unless the user explicitly requests another convention.
- Do not expose unrelated users, private cluster information, or protected biomedical data in reports or public files.
