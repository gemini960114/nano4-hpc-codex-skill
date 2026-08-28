# Nano4 File Transfer

Read this reference only when the user requests upload or download for a Nano4 job.

## Authorization and paths

- Resolve the exact local source and remote destination before transfer.
- Uploading creates or changes remote data. Confirm that the requested destination is within scope.
- Inspect whether the destination exists before overwriting. Overwrite, recursive replacement, move, and deletion require explicit authorization for the exact target.
- Do not transfer credentials, OTP values, SSH keys, unrelated user data, or files excluded from the public project for privacy reasons.

## Legacy SCP compatibility

Modern OpenSSH `scp` normally uses SFTP. When Nano4 rejects the SFTP subsystem or ordinary `scp` fails with a subsystem error, use legacy SCP mode with `-O` through the configured proxy alias:

```powershell
scp -O .\local-file nano4-proxy:/remote/path/
scp -O nano4-proxy:/remote/path/result.txt .\result.txt
```

For directories explicitly in scope:

```powershell
scp -O -r .\local-directory nano4-proxy:/remote/path/
```

Treat `-O` as a Nano4 compatibility method, not a universal SSH rule. If ordinary SCP succeeds, do not add `-O` merely by habit.

## Verification

- Require a successful local `scp` exit status.
- Verify the remote target with lightweight `ls -l` or `stat` after upload.
- For important or large files, compare a checksum such as SHA-256 when available on both sides.
- Do not interpret transfer success as authorization to submit a job.
