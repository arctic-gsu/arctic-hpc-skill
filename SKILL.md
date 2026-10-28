---
name: arctic-hpc
description: "Georgia State University ARCTIC HPC cluster reference: Slurm partitions, GPUs, storage, iRODS, Globus, Elpis, Hemera, modules, AI policy. Use for any ARCTIC or GSU HPC question or job script."
metadata:
  version: "1.0.1"
  updated: "2026-10-28"
  source: "https://arcwiki.rs.gsu.edu"
---

# ARCTIC HPC reference

This skill answers questions about ARCTIC, Georgia State University's research computing system (the ACIDS, TReNDS and Adonis clusters), and writes job scripts and code that run there.

The ARCTIC wiki (https://arcwiki.rs.gsu.edu) is the authoritative source. This skill is a summary of it, last verified in October 2026.

## How to use this skill

1. **Read the reference first.** Before you answer an ARCTIC question, read `references/arctic-reference.md`. It has the partition names, GPU types, storage paths, module versions and policies. Don't substitute generic HPC knowledge where ARCTIC has its own values. If you can't read the file (for example, a permission prompt was declined), tell the user, and don't answer ARCTIC-specific details from memory.
2. **Use concrete ARCTIC values.** Write real partition names (`qGPU24`, not "gpu"), real GPU strings (`--gres=gpu:V100:1`), real environment variables (`$SCRATCH`, `$IRODS_WORK`, `$IRODS_PROJECT`) and real module names (`module load python/3.13.3`).
3. **Say when the reference doesn't cover something.** Don't guess. Point the user to the matching wiki page, or to a Hydra ticket (https://hydra.gsu.edu, assigned to `arctic_developers`) or arc@gsu.edu.
4. **Cite the wiki.** Each reference section ends with its source pages. Link the relevant page when you answer, so the user can check the current version.

## ARCTIC gotchas

These override generic HPC advice:

- **ARCTIC backs up nothing.** HOME, iRODS WORK and PROJECT, `/userapp` and `/scratch` have no backups. Tell users to keep their own copies of important data.
- **HOME is 100 GB, for scripts and configuration only.** Its path is `/home/user/<first letter>/<username>`; use `$HOME` or `~` in scripts. Install conda and Python environments under `/userapp/<campusid>/`, never in HOME.
- **iRODS WORK and PROJECT are mounted only inside Slurm jobs** (batch jobs, interactive `srun` sessions, Hemera apps). On a login node, `$IRODS_WORK` and `$IRODS_PROJECT` are empty. Use iRODS commands (`ils`, `iput`, `iget`) there instead.
- **Don't mix the iRODS mount and iRODS commands for the same files in one job.** The mounted folders can take up to 5 minutes to show changes made with `iput`, `irm`, Globus or another job.
- **Slurm sets `$SCRATCH` but doesn't create it.** Job scripts must run `mkdir -p $SCRATCH` before `cd $SCRATCH`. Nothing deletes it when the job ends: copy results out, then `rm -rf $SCRATCH`.
- **Scratch is temporary and fragile.** It uses RAID 0, so one disk failure can destroy it. Copy results to iRODS at the end of every job.
- **Logins expire.** SSH certificates from Elpis last about 3 months; the iRODS login (`iinit`) lasts 2 weeks.
- **Hemera interactive apps are for development.** Don't run production work longer than 1 hour in them. Use batch jobs.
- **Use Globus for large transfers,** not Hemera's file browser.
- **`scp` and `rsync` need a separate SSH host entry.** `scp` to `arclogin.rs.gsu.edu` fails. Use an `~/.ssh/config` entry such as `arclogin-scp` (`HostName arclogin04.rs.gsu.edu`, `ProxyJump arclogin.rs.gsu.edu`); see the reference's Connecting section.
- **Researchers submit to `qCPU*`, `qGPU*` and `qMEM*`.** These expose V100, A30, L40S, RTX, TITAN and V100-PCIE-16 GPUs. A100, H100, A40 and L40 GPUs are on TReNDS, for TReNDS members only (`qTRD*`). Adonis GPUs are run by Kubernetes, not Slurm.
- **`pytorch/2.7.0` doesn't run on V100s.** Use it on A30, L40S or RTX GPUs, or `pip install torch` in your own venv (that build runs on V100s). `tensorflow/2.19.0` works on V100s.
- **`module load cuda/12.1` (the default) and the other `cuda` modules from 11.4 up do nothing** (their folders don't exist). `nvcc` 12.0 is already in `/usr/bin` on the V100, L40S and RTX nodes; the A30 node has `/usr/local/cuda/bin/nvcc` instead.
- **The `python/3.x` modules provide `python3`, not `python`, and only the standard library.** Write `python3` after `module load python/...`, and install packages such as pandas in a venv under `/userapp`. A venv made with `--system-site-packages` from the `pytorch/2.7.0` module doesn't see the module's packages (tested 2026-10), so pip downloads PyTorch again.
- **`module load miniconda3` doesn't set up `conda activate`.** Run `eval "$(conda shell.bash hook)"` first, in job scripts too.

## Writing job scripts and code

- Start from the batch script template in the reference.
- Use `#SBATCH --account={your_slurm_account}` as a placeholder, and tell the user to replace it with their account from Elpis.
- Stage data in and out: `mkdir -p $SCRATCH; cd $SCRATCH`, copy inputs from `$IRODS_PROJECT/{your_slurm_account}/` or `$IRODS_WORK/<campusid>/`, run, copy results back, `rm -rf $SCRATCH`.
- For GPU jobs, use `--partition=qGPU24` and `--gres=gpu:V100:1` by default. Use `A30`, `L40S`, `RTX` or `TITAN` when they fit better (for example, `L40S` for the `pytorch/2.7.0` module). Suggest TReNDS GPUs only to users who say they're TReNDS members.
- Pin module versions in job scripts (`module load python/3.13.3`), so results don't change when a default changes.
- For conda, `module load miniconda3`, `eval "$(conda shell.bash hook)"`, then `conda activate /userapp/...`.
- For interactive work, use `srun -p qCPU24` or `-p qGPU24` with a short `--time`.
- Add `--mail-type=END,FAIL` and `--mail-user={email@gsu.edu}`.
- Put `sbatch` options before the script name: `sbatch --dependency=afterok:<jobid> job2.sh`.

## Troubleshooting shortcuts

Check the Common errors table in the reference first. The usual causes:

- `Permission denied (publickey)`, or a password prompt at SSH login: the SSH certificate expired. Re-sign the key in Elpis.
- `AssocGrpCPUMinutesLimit` or `AssocGrpGRESMinutes`: the allocation's CPU or GPU time is used up.
- `MaxCPUPerAccount`: too many CPUs in use at once. Wait for jobs to finish.
- `-827000 CAT_INVALID_USER`: the iRODS login expired. Run `iinit`.
- `$IRODS_WORK` is empty: you're on a login node. Start a job or Hemera session.
- "Home directory not available" in Hemera: no active allocation. Contact the PI or request an allocation.

## AI policy (GSU)

Follow GSU's AI policy (https://technology.gsu.edu/ai/policy-and-guidance/) whenever a request involves AI tools or models:

- **Prohibited models.** GSU's AI Tool Review Registry (https://technology.gsu.edu/ai/tools/) lists **DeepSeek and Qwen as Prohibited**, including Qwen embedding models. Don't help download, install or run them on ARCTIC or with university data. If the model itself is the subject of the research (for example, benchmarking), the user must ask GSU IIT for an exception first.
- **For LLM inference, use ARCTIC's LLM service** (`https://litellm.rs.gsu.edu/v1`, OpenAI-compatible, API key by Hydra ticket) instead of hosting a model in a Slurm job. Read the key from an environment variable; never hardcode it. ARCTIC's models are approved for university data that isn't regulated: not for PII, FERPA student records, PHI, export-controlled data or CUI.
- **Other AI tools.** As of October 2026, the registry approves Microsoft Copilot Chat, Microsoft 365 Copilot, Copilot Studio, Google Gemini and NotebookLM under GSU's licenses, with enterprise data protection, for data up to Confidential (Gemini and NotebookLM for faculty and staff). Other tools are limited to public data or prohibited. Tell users to check the registry and GSU's data types guide (https://technology.gsu.edu/ai/using-ai-with-intention/) before they use any AI tool with non-public data.
- When you discuss running, hosting or choosing models on ARCTIC, link the AI policy page.

## Reference file

| File | Contents |
| --- | --- |
| `references/arctic-reference.md` | Hardware and GPUs, access and SSH, storage, data transfer, software modules, Slurm partitions and scripts, Hemera and Callisto, AI services and policy, allocations, support, common errors. Read it before answering. |
