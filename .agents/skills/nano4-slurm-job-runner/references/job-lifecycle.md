# Nano4 Slurm Job Lifecycle

Read this reference before job preparation, submission, or ongoing lifecycle monitoring. For a monitoring-only request, validate identity and job ownership, then apply the monitoring and reporting sections without preparing files or submitting anything.

Unless an agent-owned remote PTY is already active, execute every Nano4 command in this reference through `ssh nano4-proxy`. Never run a bare Slurm command in the local Windows shell.

## 1. Confirm authority and targets

Identify the requested computation, remote working directory, input files, output destinations, project, partition, and desired stopping condition. Resolve missing values with read-only checks when possible. Stop and ask before mutation if a choice would materially change cost, data placement, or execution.

A submission request does not authorize cancellation, deletion, overwrite of existing data, or an extra submission attempt.

## 2. Run current preflight checks

Confirm identity and time, then query the current project and scheduler state:

```bash
ssh nano4-proxy 'hostname; whoami; date --iso-8601=seconds'
ssh nano4-proxy 'wallet'
ssh nano4-proxy 'sinfo -h -o "%P|%a|%l|%D|%t"'
ssh nano4-proxy 'scontrol show partition -o'
ssh nano4-proxy 'remote_user=$(whoami); sacctmgr -n -P show assoc where user="$remote_user" format=Cluster,Account,User,Partition,QOS,DefaultQOS'
```

Use `wallet` to select an active project with positive SU. Cross-check partition account restrictions, state, QoS, maximum wall time, and current resource shape. Do not use a prior report or a `sacctmgr` association alone as proof of usability.

## 3. Prepare the working directory

Use a user-approved path under the observed account. Inspect before creating or writing:

```bash
ssh nano4-proxy 'test -e -- "REMOTE_WORKDIR"; printf "%s\n" "$?"'
```

Replace `REMOTE_WORKDIR` with the exact, safely shell-quoted path before running the check. Create only the agreed directory and required log directory. Do not overwrite an existing script, input, or output unless the user authorized that exact replacement. Slurm does not create missing parents for stdout or stderr.

## 4. Validate the batch script

The script should include or receive at submission time:

- explicit project and partition;
- job name;
- node/task/CPU configuration consistent with the current partition;
- requested memory and wall time within current limits;
- explicit stdout and stderr paths;
- strict or deliberate shell error handling appropriate to the application;
- module or environment setup required by the workload.

Keep `PROJECT_ID` as a placeholder until current `wallet` and access checks identify the chosen project. Recheck the selected partition's current QoS, CPU, memory, node shape, and wall-time limits before submission.

## 5. Submit exactly once

Use a parseable response and explicit project and partition:

```powershell
ssh nano4-proxy 'cd -- "REMOTE_WORKDIR" && sbatch --parsable -A PROJECT_ID -p PARTITION job.sh'
```

Replace `REMOTE_WORKDIR` with the exact, safely shell-quoted job directory. Require exit status zero and parse the returned `JOB_ID` or `JOB_ID;CLUSTER`. Store the numeric Job ID separately from other output.

If the command times out, disconnects, or returns ambiguous output, do not run `sbatch` again. First query `squeue` and `sacct` using the user, job name, and submission time window. If duplication cannot be ruled out, stop and ask the user.

## 6. Monitor with structured output

Use a specific Job ID:

```powershell
ssh nano4-proxy 'squeue -h -j JOB_ID -o "%i|%T|%P|%a|%j|%M|%l|%R"'
ssh nano4-proxy 'sacct -n -P -j JOB_ID --format=JobIDRaw,JobName,Partition,Account,State,ExitCode,Elapsed,AllocCPUS,ReqMem,MaxRSS'
```

Poll at a cadence proportional to the expected runtime; do not aggressively query the scheduler. A missing `squeue` row does not by itself prove success because completed jobs leave the active queue. Confirm the terminal state with `sacct`.

When a job is pending, report the scheduler reason. Do not alter resources, requeue, hold, release, or cancel unless explicitly requested.

## 7. Inspect outputs and metrics

After a terminal state, inspect only the named stdout and stderr files and lightweight metadata needed for the report. Do not recursively scan large result trees unless requested.

Record when available:

- job and step states;
- exit code;
- elapsed time;
- allocated CPUs and requested memory;
- `MaxRSS` or other reported memory metric;
- stdout and stderr paths and whether they contain errors;
- expected result-file existence.

Metrics can be delayed or unavailable. Label missing values rather than inferring them.

## 8. Report and stop

Report the Job ID, project, partition, working directory, submission time, final observed state, exit code, principal metrics, output paths, and unresolved warnings. Distinguish scheduler acceptance from application success.

Stop at the requested lifecycle point. Do not submit another job, cancel anything, or clean up files unless separately authorized.
