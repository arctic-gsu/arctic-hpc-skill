# GSU ARCTIC HPC reference

Last verified: 2026-10. The ARCTIC wiki (https://arcwiki.rs.gsu.edu) is authoritative: if it disagrees with this file, follow the wiki. Each section ends with the wiki pages it comes from. Facts marked "tested 2026-10" come from test jobs run on the cluster in October 2026.

ARCTIC (Advanced Research Computing Technology Innovation and Collaboration) is Georgia State University's research computing system. It combines three clusters connected by InfiniBand:

- **ACIDS** (Slurm): open to any researcher with an ARCTIC allocation. Funded by NSF MRI grant CNS-1920024.
- **TReNDS** (Slurm): neuroimaging cluster for TReNDS members.
- **Adonis** (Kubernetes): AI workloads and ARCTIC's LLM service. Funded by NSF PROFLEX-CI grant 2430193.

ARCTIC provides about 75 million CPU hours and 1.2 million GPU hours a year.

Source: https://arcwiki.rs.gsu.edu/en/home, https://arcwiki.rs.gsu.edu/en/home/acids/specifications, https://arcwiki.rs.gsu.edu/en/policy/resource-allocations

---

## 1. Hardware

| Cluster | Scheduler | Physical cores | Slurm CPUs | Who can use it |
| --- | --- | --- | --- | --- |
| ACIDS | Slurm | ~2,400 | ~2,600 | Any researcher with an ARCTIC allocation (`qCPU*`, `qGPU*`, `qMEM*`) |
| TReNDS | Slurm | ~4,700 | ~5,200 | TReNDS members (`qTRD*`) |
| Adonis | Kubernetes | ~1,300 | — | Through ARCTIC's LLM service and Kubernetes GPU jobs |

Slurm CPUs (what `--cpus-per-task` counts) are higher than physical cores on the few nodes with hyperthreading.

### ACIDS nodes

| Nodes | Slurm CPUs per node | Memory per node | GPUs per node | Partitions |
| --- | --- | --- | --- | --- |
| acidscn001, acidscn003–005, acidscn008–011, acidscn028 | 64 | 256 GB | — | qCPU* |
| acidscn002, acidscn006 | 64 | 1 TB | — | qCPU*, qMEM* |
| acidscn012–020 | 24 | 192 GB | — | qCPU* |
| acidsint01–02 | 64 | 256 GB | — | qCPU* |
| acidsgcn001–006 | 40 | 384 GB | 4× V100 32 GB | qGPU* |
| acidsgcn007 | 32 | 768 GB | 8× V100 32 GB | qGPU* |
| acidsgcn011 | 40 | 512 GB | 8× V100 32 GB | qGPU* |
| acidsgcn010 | 64 | 256 GB | 6× L40S 48 GB | qGPU* |
| acidsgcn013 | 128 | 2 TB | 8× A30 24 GB | qGPU* |
| acidsgn001 | 64 | 256 GB | 1× TITAN V 12 GB | qCPU*, qGPU* |
| acidsgn002–006, acidsgn008 | 64 | 256 GB | 1× RTX 2080 Ti 11 GB | qCPU*, qGPU* |
| acidsgn007 | 64 | 1 TB | 1× RTX 2080 Ti 11 GB | qCPU*, qGPU*, qMEM* |
| acidsmn001 | 48 | 1.5 TB | — | qMEM* |
| acidsmn002 | 48 | 3 TB | — | qMEM* |
| acidsmn003 | 64 | 1 TB | 2× V100-PCIE 16 GB | qMEM* |
| acidsmn004–005 | 64 | 1 TB | — | qMEM* |

Run `sinfo -N -o "%N %c %m %G %P"` to see current values.

### GPU inventory

| GPU | Count | GPU memory | Where | Access |
| --- | --- | --- | --- | --- |
| V100 | 40 | 32 GB | ACIDS | `qGPU*` |
| V100-PCIE-16 | 2 | 16 GB | ACIDS | `qMEM*` |
| A30 | 8 | 24 GB | ACIDS | `qGPU*` |
| L40S | 6 | 48 GB | ACIDS | `qGPU*` |
| RTX 2080 Ti | 7 | 11 GB | ACIDS | `qGPU*`, `qCPU*` |
| TITAN V | 1 | 12 GB | ACIDS | `qGPU*`, `qCPU*` |
| A100 | 32 | 40/80 GB | TReNDS | TReNDS members |
| H100 | 4 | 80 GB | TReNDS | TReNDS members |
| A40 | 31 | 48 GB | TReNDS | TReNDS members |
| L40 | 4 | 48 GB | TReNDS | TReNDS members |
| L40S | 2 | 48 GB | TReNDS | TReNDS members |
| V100 | 4 | 32 GB | TReNDS | TReNDS members |
| RTX 2080 Ti | 17 | 11 GB | TReNDS | TReNDS members |
| H100 | 12 | 80 GB | Adonis | Kubernetes |
| RTX PRO 6000 (Blackwell) | 4 | 96 GB | Adonis | Kubernetes |
| L40S | 11 | 48 GB | Adonis | Ask ARCTIC staff |

Adonis GPUs run ARCTIC's LLM service and Kubernetes workloads; their availability for your own jobs varies. To request access, open a Hydra ticket.

**For researcher Slurm scripts, default to V100** (`--gres=gpu:V100:1`), the most numerous. A30, L40S, RTX and TITAN are also open to researchers. Suggest TReNDS GPUs only to TReNDS members.

Source: https://arcwiki.rs.gsu.edu/en/home/acids/specifications, https://arcwiki.rs.gsu.edu/en/hardware/node-config

---

## 2. Access and SSH

### Elpis and allocations

Elpis (https://elpis.rs.gsu.edu) is the portal for projects, allocations, members and SSH certificate signing. Log in with CILogon.

- **Faculty and staff** click **Request PI Status**, choose the `acids` project type if unsure, and wait for ARCTIC's review. GSU users then get a startup allocation (10,000 CPU hours and 1,000 GPU hours).
- **Students** can't request PI status. Their faculty advisor adds them to a project.
- **Project roles:** Manager (adds users, publications and grants), User, and Visitor. Only managers can add users.
- **External collaborators** need a GSU affiliate account for the VPN access required to log in.
- Members other than the PI can turn off Elpis notification emails on the project's Users page.

### SSH keys and certificates

1. Generate a key: `ssh-keygen -t ed25519`, saved as `~/.ssh/id_<campusid>`. macOS needs ed25519 keys (RSA keys fail with "no mutual signature algorithm").
2. In Elpis, sign the public key and save the certificate as `~/.ssh/id_<campusid>-cert.pub`, in the same folder as the key.
3. Add the key to your SSH agent: `ssh-add ~/.ssh/id_<campusid>` (macOS: `ssh-add --apple-use-keychain ~/.ssh/id_<campusid>`).

Certificates last about **3 months**. Then logins fail with `Permission denied (publickey)` or ask for a password. To renew: `ssh-add -D`, delete the old `-cert.pub` file, re-sign the public key in Elpis, save the new certificate, and `ssh-add` again. Check validity with `ssh-keygen -L -f ~/.ssh/id_<campusid>-cert.pub`.

**Windows:** the OpenSSH Authentication Agent is off by default. In an administrator PowerShell, run `Set-Service -Name ssh-agent -StartupType Automatic -Status Running` (without admin rights, ask GSU IT at help.gsu.edu). If `ssh-keygen` isn't found, install the OpenSSH client or add `C:\Windows\System32\OpenSSH` to `PATH`. MobaXterm's built-in agent doesn't support OpenSSH certificates; use PuTTY 0.78 or newer with Pageant (convert the key to `.ppk` in PuTTYgen and add the certificate).

### Connecting

- Log in with `ssh <campusid>@arclogin.rs.gsu.edu`. After the key check, Duo asks for a push or a call. TReNDS members use `arctrdlogin001.rs.gsu.edu`.
- Off campus, connect to the GSU VPN first.
- `~/.ssh/config`:
  ```
  Host arclogin.rs.gsu.edu
      User <campusid>
      ForwardAgent yes
  ```
  macOS users also add `UseKeychain yes` and `AddKeysToAgent yes`.
- **VS Code Remote-SSH** to a login node:
  ```
  Host arclogin-<campusid>
      HostName arclogin04.rs.gsu.edu
      ProxyJump arclogin.rs.gsu.edu
      User <campusid>
  ```
  You can also connect VS Code (or `ssh`) straight to a compute node where you have a running job: `HostName <node>.rs.gsu.edu`, no ProxyJump needed (tested 2026-10). See the VS Code wiki page.
- **scp and rsync** don't work through `arclogin.rs.gsu.edu` directly. Add a second entry that reaches `arclogin04` through `arclogin` (macOS users add the keychain lines here too), then use it as the host:
  ```
  Host arclogin-scp
      HostName arclogin04.rs.gsu.edu
      ProxyJump arclogin.rs.gsu.edu
      User <campusid>
  ```
  `scp results.csv arclogin-scp:~/` copies to ARCTIC; `scp arclogin-scp:~/results.csv .` copies back; `rsync -av myfolder/ arclogin-scp:~/myfolder/` works too. Duo asks for approval each time, as at login. Use Globus for large transfers. Full guide: https://arcwiki.rs.gsu.edu/en/data-transfer/scp-rsync

Source: https://arcwiki.rs.gsu.edu/en/home/elpis/startup-allocation, https://arcwiki.rs.gsu.edu/en/home/ssh/generating-keys, https://arcwiki.rs.gsu.edu/en/home/elpis/signing-keys, https://arcwiki.rs.gsu.edu/en/home/ssh/signing-in, https://arcwiki.rs.gsu.edu/en/home/ssh/authentication-issues, https://arcwiki.rs.gsu.edu/en/home/elpis/resign-key, https://arcwiki.rs.gsu.edu/en/home/acids/how-to/how-to-login-to-acids-cluster-using-vscode, https://arcwiki.rs.gsu.edu/en/home/elpis/add-users

---

## 3. Storage

| Location | Path | Size | Where it's available | Use |
| --- | --- | --- | --- | --- |
| HOME | `$HOME` = `/home/user/<first letter>/<username>` | 100 GB | Login and compute nodes | Scripts and configuration files |
| WORK | `/arctic/work/<campusid>`; in jobs `$IRODS_WORK/<campusid>` | 50 GB | Inside Slurm jobs only (mounted) | Personal research data |
| PROJECT | `/arctic/projects/<account>`; in jobs `$IRODS_PROJECT/<account>` | 200 GB, up to 2 TB on request | Inside Slurm jobs only (mounted) | Shared group data |
| SCRATCH | `/scratch`; per job `$SCRATCH` = `/scratch/users/<campusid>/<jobid>` | ~252 TB shared (BeeGFS) | Login and compute nodes | Temporary files for running jobs |
| USERAPP | `/userapp` | ~15 TB shared | Login and compute nodes | Conda and Python environments, user-installed software |

**Nothing is backed up.** ARCTIC doesn't back up HOME, WORK, PROJECT, `/userapp` or `/scratch`, and can't restore deleted or lost files. Keep copies of important data outside ARCTIC.

- **HOME** moved to `/home/user/<first letter>/<username>` with a 100 GB quota in the 2026 home migration (it was `/home/users/<username>`, 25 GB; old pages and scripts may still use that path). Use `$HOME` or `~` in scripts. Going over quota breaks iRODS logins.
- **WORK and PROJECT** are iRODS storage, mounted only inside Slurm jobs (batch, `srun`, Hemera apps). On a login node, `$IRODS_WORK` and `$IRODS_PROJECT` are empty.
- **`/userapp`** is visible to all users by default; only the owner can delete files.
- **Scratch** is temporary: it uses RAID 0, so one disk failure can destroy it, and staff clean it up manually. Slurm sets `$SCRATCH` but doesn't create the folder, and nothing deletes it when the job ends (tested 2026-10). In job scripts:
  ```bash
  mkdir -p $SCRATCH
  cd $SCRATCH
  # ... work ...
  cp results.txt $IRODS_PROJECT/{your_slurm_account}/
  rm -rf $SCRATCH
  ```
- **PROJECT storage** starts at 200 GB, PIs can request up to 2 TB free, and more costs $200/TB per 3 years.
- **Data rules (Acceptable Use Policy):** only de-identified data belongs on ARCTIC. Never process or store personally identifiable information there. Report suspected credential breaches to help@gsu.edu or 404-413-HELP (4357).

Source: https://arcwiki.rs.gsu.edu/en/iRODS/Storage, https://arcwiki.rs.gsu.edu/en/backup/backup-policy, https://arcwiki.rs.gsu.edu/en/home/acids/how-to/slurm, https://arcwiki.rs.gsu.edu/en/data-transfer/whats-changed-after-home-migration, https://arcwiki.rs.gsu.edu/en/policy/acceptable-use

---

## 4. Data transfer

### Globus

Use Globus (https://globus.org, GSU login) for transfers between your computer and ARCTIC.

1. Install Globus Connect Personal on your computer and create a personal collection.
2. Transfers between your computer and scratch use Globus Flows: in Flows, search "arctic" and choose **ARCTIC Transfer (Personal to Scratch)**, **ARCTIC Transfer (Scratch to Personal)** or **ARCTIC Transfer (Personal to S3)**. The first run asks for consent. Files sent to scratch land in `/scratch/users/<campusid>`.
3. Flows run in two stages through the **NVME-ARCTIC** collection. If stage two fails, resume it from the File Manager (NVME-ARCTIC → your campus ID → the run's folder → transfer to **ARCTIC-SCRATCH** or your personal collection).
4. Other collections: **S3-ARCTIC** (object storage), and for the 2026 home migration **ARCTIC-HOME** (old) and **ARCTIC-HOME-NEW** (new). Don't use the `-INTERNAL` versions of those two.

Hemera's file browser is for viewing files and light transfers; use Globus for large files or many files.

### Between scratch and iRODS

- **Small transfers:** in an interactive session (`srun -p qCPU24 -A {your_slurm_account} --time=01:00:00 --mem=4G --pty bash`), `cp` between `/scratch/users/<campusid>` and `$IRODS_WORK/<campusid>` or `$IRODS_PROJECT/<account>`.
- **Large transfers:** a batch job with `iput`, which verifies checksums, retries and can resume:
  ```bash
  #SBATCH --partition=qCPU24
  #SBATCH --time=10:00:00
  #SBATCH --mem=8gb
  #SBATCH --cpus-per-task=1
  SRC=/scratch/users/{campusid}/{folder}
  DEST=/arctic/projects/{your_slurm_account}
  iput -rvK -X "$HOME/iput_restart" --lfrestart "$HOME/iput_lfrestart" --retries 3 "$SRC" "$DEST"
  ```
  Run `iinit` on a login node before submitting. If the job runs out of time, submit it again; `iput` resumes.

### iRODS commands

- `iinit` logs in (use your GSU password as the PAM password). The login lasts **2 weeks**. GSU passwords expire every 6 months; after a change, `iinit` needs the new one.
- After the first `iinit`, start a new job or Hemera session to get the mounted folders.
- Commands: `ils`, `icd /arctic/work/<campusid>`, `imkdir`, `iput`, `iget`, `ils -A` (permissions). They work on login nodes and in jobs. ARCTIC uses iCommands 5.0.2: https://docs.irods.org/5.0.2/icommands/user/
- The mounted folders can take up to 5 minutes to show changes made with iRODS commands, Globus or another job. In one job, use either the mount or iRODS commands for the same files, not both.

### S3 object storage (Osiris)

Create access keys at https://osiris.rs.gsu.edu ("Login with SSO"); save the secret key when it's shown, because it can't be shown again. The S3 endpoint for Python (boto3) is `https://osiris.rs.gsu.edu`; the bucket console is https://console-osiris.rs.gsu.edu. Off campus, use the GSU VPN. To use S3 with Globus, add the keys to the S3-ARCTIC collection's credentials.

### Rclone

`module load rclone` (1.60.0) for Dropbox or S3 remotes: configure on the cluster and authorize on your own computer. Hemera also has an RClone Web GUI app.

Source: https://arcwiki.rs.gsu.edu/en/data-transfer/globus/transfer-from-personal-to-scratch, https://arcwiki.rs.gsu.edu/en/data-transfer/globus/scratch-to-personal, https://arcwiki.rs.gsu.edu/en/new-page, https://arcwiki.rs.gsu.edu/en/data-transfer/globus/migrate-to-new-home-folder, https://arcwiki.rs.gsu.edu/en/data-transfer/terminal/copy-from-scratch-to-irods, https://arcwiki.rs.gsu.edu/en/iRODS/navigating-irods-fusemounts, https://arcwiki.rs.gsu.edu/en/iRODS/Troubleshooting, https://arcwiki.rs.gsu.edu/en/minio/creating-access-keys, https://arcwiki.rs.gsu.edu/en/python/connecting-to-s3-buckets-using-python, https://arcwiki.rs.gsu.edu/en/backup/backup_data_to_dropbox, https://arcwiki.rs.gsu.edu/en/home/acids/how-to/browse

---

## 5. Software

ARCTIC uses Lmod modules. `module load <name>` without a version loads the default; `module avail <name>` marks the default with `(D)`. Pin versions in job scripts.

```bash
module avail                 # list modules
module avail python          # versions of one package
module spider <name>         # search, including hidden modules
module load python/3.13.3    # load a version
module list                  # loaded modules
module purge                 # unload all
```

### Main modules (October 2026)

| Module | Versions (default marked D) |
| --- | --- |
| `python` | 3.9.22, 3.10.14, 3.12.7, 3.13.3 (D) |
| `miniconda3` | 4.12.0, 23.5.2, 25.5.1 (D) |
| `R` (capital R) | 4.3.3, 4.4.3 |
| `r-studio-desktop` | 2025.05.0+496 |
| `pytorch` | 2.7.0 |
| `tensorflow` | 2.19.0 |
| `jupyterlab` | 4.4.3 |
| `cuda` | 10.2, 11.4, 11.6, 11.7, 12.1 (D) |
| `matlab` | R2019b, R2020b (D), R2022a, R2022b, R2023a, R2025a |
| `gcc` | 11.2.0, 12.4.0 |
| `openmpi` | 4.1.4, 5.0.0 (D) |
| `mvapich2` | 2.3.7 |
| `inteloneapi` | 2025.3 (components as `inteloneapi/<component>/<version>`, including Intel MPI 2021.17) |
| `singularity` | 4.3.1 |
| `apptainer` | 1.4.1 |
| `rclone` | 1.60.0 |
| `stata` | MP-17 |
| `nsight-systems` | 2022.2.1, 2024.4.1 |

Modules that old wiki pages mention but that no longer exist (October 2026): `dask`, `clearmap`, `dmtcp`, `aistore`, `nvhpc`, EasyBuild `Python/3.7.x`, `R-4/*`. Run `module avail` to confirm anything else.

To request new software, see the Application Installation Request page: requests are reviewed on the 1st and 15th of each month.

### GPU software (tested 2026-10)

- **CUDA compiler:** the `cuda` modules 11.4 through 12.1 (including the default, 12.1) do nothing, because their folders don't exist; `cuda/10.2` points into the NVIDIA HPC SDK. The V100, L40S and RTX nodes already have `nvcc` 12.0 in `/usr/bin`. The A30 node (`acidsgcn013`) has no `nvcc` on `PATH`; use `/usr/local/cuda/bin/nvcc` there. Login nodes have no `nvcc` and no GPUs: compile in a GPU job.
- **`pytorch/2.7.0`** (torch 2.7.0 + CUDA 12.8) supports A30, L40S and RTX GPUs but **not V100s**: `torch.cuda.is_available()` returns True on a V100, but GPU work fails. Request `--gres=gpu:L40S:1` (or A30, RTX) with it. For V100s, make your own venv from `python/3.13.3` and `pip install torch`: the PyPI build (CUDA 12.6) runs on V100s (tested 2026-10).
- **`tensorflow/2.19.0`** runs GPU work on V100 and L40S.
- **R torch** (R/4.4.3): the pre-built torch binaries need R 4.5 and don't load. What works, in a GPU job with 4 CPUs, 24 GB and about 10 GB of free space: in R, `options(timeout = 1800)`, `Sys.setenv(MAKEFLAGS = "-j4")`, `install.packages("cuda12.8", repos = c("https://mlverse.r-universe.dev", "https://cloud.r-project.org"))`, then `install.packages("torch")` (about 15 minutes), then `Sys.setenv(TORCH_INSTALL = "1"); library(torch)` (downloads 3.6 GB the first time). Tested on V100 and L40S.
- Compute nodes have internet access, so `pip install` and `install.packages` work in jobs.

### Conda

Keep environments and package caches out of HOME:

```bash
module load miniconda3
eval "$(conda shell.bash hook)"          # required before conda activate, also in job scripts
mkdir -p /userapp/<campusid>/conda-tmp
export CONDA_PKGS_DIRS=/userapp/<campusid>/conda-tmp
conda env create -f environment.yml --prefix /userapp/<campusid>/condaenv/myenv
conda activate /userapp/<campusid>/condaenv/myenv
```

Without the `eval` line, `conda activate` fails with "Run 'conda init' before 'conda activate'" (tested 2026-10).

### Python virtual environments

The `python/3.x` modules provide only the `python3` command (no `python`) and the standard library: no pandas or NumPy (tested 2026-10). Inside an activated venv, `python` works. The `pytorch/2.7.0` module is itself a venv; a new venv made from it with `--system-site-packages` doesn't see its packages (tested), so pip downloads PyTorch again.

Create venvs in an interactive job, under `/userapp`:

```bash
srun -p qCPU24 -A {your_slurm_account} --time=00:30:00 --pty bash
module load python/3.13.3
python3 -m venv /userapp/<campusid>/venv/myproject
source /userapp/<campusid>/venv/myproject/bin/activate
pip install --upgrade pip
pip install <package>
```

### Containers

`module load singularity` or `module load apptainer`. Convert Docker images: `singularity build python.img docker://python:3.10-slim`, or `singularity pull docker://IMAGE` (saves a `.sif`), or `singularity exec docker://IMAGE CMD` (runs without saving). Harbor (`harbor.rs.gsu.edu`) is ARCTIC's private registry for Kubernetes jobs; request a Harbor project with a Hydra ticket, then log in with `docker login harbor.rs.gsu.edu -u '<username>' --password-stdin`.

Source: https://arcwiki.rs.gsu.edu/en/software/software, https://arcwiki.rs.gsu.edu/en/software/load-modules, https://arcwiki.rs.gsu.edu/en/policy/application-installation-request, https://arcwiki.rs.gsu.edu/en/cuda/examples/hello-world, https://arcwiki.rs.gsu.edu/en/python/install-pytorch-package, https://arcwiki.rs.gsu.edu/en/python/install-tensorflow, https://arcwiki.rs.gsu.edu/en/Rstat/installing-torch-package, https://arcwiki.rs.gsu.edu/en/python/Conda, https://arcwiki.rs.gsu.edu/en/python/creating-a-venv, https://arcwiki.rs.gsu.edu/en/docker/running-docker-image-with-singularity, https://arcwiki.rs.gsu.edu/en/harbor/push-docker-image-to-harbor

---

## 6. Slurm

### Partitions for researchers

The number in a partition name is its maximum run time in hours. Without `--partition`, jobs go to **qCPU24**. Totals as of October 2026; run `sinfo -p <partition>` for current state.

| Partition | Max time | CPUs | Nodes | GPUs | Memory | Max CPUs/user | Max GPUs/user | Max memory/user |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| qCPU24 (default) | 24 h | 1,560 | 30 | 8 | ~9,200 GB | No limit | No limit | No limit |
| qCPU48 | 48 h | 1,560 | 30 | 8 | ~9,200 GB | 300 | 4 | 1,200 GB |
| qCPU120 | 120 h | 1,560 | 30 | 8 | ~9,200 GB | 100 | 2 | 400 GB |
| qGPU24 | 24 h | 1,016 | 18 | 62 | ~8,700 GB | No limit | No limit | No limit |
| qGPU48 | 48 h | 1,016 | 18 | 62 | ~8,700 GB | 80 | 8 | 600 GB |
| qGPU120 | 120 h | 1,016 | 18 | 62 | ~8,700 GB | 40 | 4 | 300 GB |
| qMEM24 | 24 h | 480 | 8 | 3 | ~10,500 GB | No limit | — | No limit |
| qMEM48 | 48 h | 480 | 8 | 3 | ~10,500 GB | 128 | — | 4,000 GB |
| qMEM120 | 120 h | 480 | 8 | 3 | ~10,500 GB | 64 | — | 2,000 GB |

- The 8 GPUs in `qCPU*` are 7 RTX and 1 TITAN. The 3 GPUs in `qMEM*` are 2 V100-PCIE-16 and 1 RTX.
- GPU types in `qGPU*`: `V100` (40), `A30` (8), `L40S` (6), `RTX` (7), `TITAN` (1). Request with `--gres=gpu:<TYPE>:<count>`, or `--gres=gpu:1` for any GPU.
- `qTRD*` partitions run on TReNDS and are for TReNDS members only. Some other partitions belong to specific groups or accounts; use them only if ARCTIC staff told you to.

### Service units and allocations

- **CPU SU:** one core for one hour. **GPU SU:** one V100 32 GB for one hour; other GPUs scale by their power. **Memory SU:** 1 GB reserved for one hour.
- Per the Resource Allocations page, the smallest memory unit is 3 GB, and a job is charged for its cores or its 3 GB memory units, whichever is larger (1 core + 5 GB for 1 hour = 2 SU; 1 core + 3 GB = 1 SU; 2 cores + 3 GB = 2 SU). For exact billing on a project, check Elpis or ask ARCTIC staff.
- Check limits and usage:
  ```bash
  sacctmgr show assoc where account={your_slurm_account} format=account,GrpTRES,GrpTRESMins
  sreport cluster AccountUtilizationByUser account={your_slurm_account} start=0101 -T cpu,gres/gpu
  ```

### Batch script template

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=4gb
#SBATCH --time=01:00:00
#SBATCH --account={your_slurm_account}
#SBATCH --partition=qCPU24
#SBATCH --output=output_%j.txt
#SBATCH --error=error_%j.txt
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user={email@gsu.edu}

mkdir -p $SCRATCH          # Slurm sets $SCRATCH but doesn't create it
cd $SCRATCH
cp $IRODS_PROJECT/{your_slurm_account}/input.py $SCRATCH

module purge
module load python/3.13.3
python input.py

cp results.txt $IRODS_PROJECT/{your_slurm_account}/
rm -rf $SCRATCH
```

For a GPU job, add `#SBATCH --partition=qGPU24` and `#SBATCH --gres=gpu:V100:1`.

Output file patterns: `%j` job ID, `%A` array master job ID, `%a` array index, `%N` node, `%u` user, `%x` job name.

### Interactive sessions

```bash
srun -p qCPU24 -A {your_slurm_account} --time=01:00:00 --pty bash
srun -p qGPU24 -A {your_slurm_account} --gres=gpu:V100:1 --mem=8G --time=01:00:00 --pty bash
```

### Arrays, dependencies, MPI, OpenMP, X11

- **Array jobs:** `#SBATCH --array=0-9`; each task gets `$SLURM_ARRAY_TASK_ID`. Each task has its own `$SCRATCH`, so copy its inputs in.
- **Dependencies:** `sbatch --dependency=afterok:<jobid> job2.sh` (options go before the script name). Types: `afterok`, `afterany`, `after`, `afternotok`.
- **MPI:** `module load openmpi`, then `mpirun ./prog` in a batch job. With MVAPICH2: `export MV2_SMP_USE_CMA=0`, `module load mvapich2`, then `srun --mpi=pmi2 ./prog`.
- **OpenMP:** set `OMP_NUM_THREADS` to `$SLURM_CPUS_PER_TASK`.
- **X11:** `ssh -X <campusid>@arclogin.rs.gsu.edu`, then `srun -p qCPU24 -A {your_slurm_account} --x11 --time=02:00:00 --pty bash`. On your computer, run an X server and `export DISPLAY=127.0.0.1:0` (PowerShell: `$env:DISPLAY="127.0.0.1:0"`).

### Monitoring

`squeue --me`, `sacct -j <jobid>`, `sinfo -p qGPU24`, `scontrol show node <node>`, `scancel <jobid>`.

Source: https://arcwiki.rs.gsu.edu/en/home/acids/queue, https://arcwiki.rs.gsu.edu/en/home/slurm/how-to-request-partitions, https://arcwiki.rs.gsu.edu/en/home/acids/how-to/slurm, https://arcwiki.rs.gsu.edu/en/policy/resource-allocations, https://arcwiki.rs.gsu.edu/en/home/slurm/troubleshoot, https://arcwiki.rs.gsu.edu/en/home/slurm/submit-array-jobs, https://arcwiki.rs.gsu.edu/en/home/slurm/running-dependent-slurm-jobs, https://arcwiki.rs.gsu.edu/en/home/slurm/x11-forwarding, https://arcwiki.rs.gsu.edu/en/home/acids/how-to/how-to-submit-interactive-jobs, https://arcwiki.rs.gsu.edu/en/mpi-omp/running-mpi-program, https://arcwiki.rs.gsu.edu/en/mpi-omp/running-omp-jobs

---

## 7. Hemera and Callisto

### Hemera (Open OnDemand)

https://hemera.rs.gsu.edu, GSU login. TReNDS members use https://hemera-staging.rs.gsu.edu since the 2026 home migration.

- **Desktop:** ACIDs Interactive Apps → ACIDs Desktop → set account, hours and cores → Launch. When you finish, click **Delete** on the session.
- **Interactive apps are for development only.** Don't use them for production runs longer than 1 hour; you can lose data. Run production work as batch jobs.
- **Job Composer:** Jobs → Job Composer → New Job → From default template, edit, submit.
- Apps (checked 2026-10): ACIDs Desktop, JupyterLab, VS Code Server, MATLAB, Stata, RClone Web GUI, JupyterLab-DeepLabCut and JupyterLab-PySpark. Netron is no longer a Hemera app. Check Hemera for the current list.
- **MATLAB on Hemera:** set Memory (GB) to 4 × the number of CPUs. For cluster jobs from MATLAB, run `configCluster` once, then `c = parcluster;` and set `c.AdditionalProperties.AccountName`.
- "Home directory not available" at login means no active allocation.

### Callisto

Teaching platform with Jupyter notebooks at https://callisto.rs.gsu.edu: GSU login, on campus or VPN, no allocation needed. Server types: Minimal, Datascience (Python, R, Julia), PySpark, MATLAB and Ray. One server at a time; stop it with File → Hub Control Panel → Stop My Server. Move real research to Hemera or batch jobs.

Source: https://arcwiki.rs.gsu.edu/en/OpenOnDemand/interactive-computing, https://arcwiki.rs.gsu.edu/en/home/acids/how-to/login-to-cluster-using-hemera, https://arcwiki.rs.gsu.edu/en/data-transfer/whats-changed-after-home-migration, https://arcwiki.rs.gsu.edu/en/matlab/parallel-server, https://arcwiki.rs.gsu.edu/en/VSCode-Hemera, https://arcwiki.rs.gsu.edu/en/python/callisto

---

## 8. AI services and GSU AI policy

### ARCTIC LLM service (LiteLLM)

- On-prem models through an OpenAI-compatible API at `https://litellm.rs.gsu.edu/v1`. Request an API key with a Hydra ticket. Needs the GSU network or VPN.
- **Alpha service:** models may be slow or unavailable without notice.
- Models (October 2026): **MiniMax M2.7** (`MiniMaxAI/MiniMax-M2.7`) and **Gemma 4 31B** (`google/gemma-4-31B-it`). Both are text-only on ARCTIC. List current models with:
  ```bash
  curl -s -H "Authorization: Bearer $LITELLM_API_KEY" https://litellm.rs.gsu.edu/v1/models
  ```
- Keep the key in an environment variable, never in scripts, notebooks or repositories:
  ```python
  import os
  from openai import OpenAI
  client = OpenAI(base_url="https://litellm.rs.gsu.edu/v1", api_key=os.environ["LITELLM_API_KEY"])
  ```
- **Data:** approved for university data that isn't regulated. Not for PII, student records (FERPA), health information (PHI/HIPAA), export-controlled data, CUI or anything with stricter protection.
- **Records:** ARCTIC records each request (user, time, model, usage) and may keep prompts and responses. AI inputs and outputs may be public records under the Georgia Open Records Act.
- The wiki's OpenCode guide sets up a coding assistant with these models: https://arcwiki.rs.gsu.edu/en/LiteLLM/OpenCode
- For LLM inference, use this service instead of hosting a model in a Slurm job. For needs it doesn't cover, open a Hydra ticket.

### Kubernetes GPU jobs (Adonis)

1. Have an active ARCTIC allocation; the Kubernetes API is reachable only from inside ARCTIC's network.
2. Open a Hydra ticket assigned to `arctic_developers` with your group or PI, workload, GPUs and storage needs. Staff create a namespace, storage (PVC), GPU quota and login, and send a kubeconfig (usually `~/.kube/config`). Run `kubectl` from a login node (`~/kubectl` if it isn't on `PATH`).
3. Build a container image and push it to Harbor.
4. Copy data to the namespace's drive through the always-on access pod: `kubectl cp ./data access-pod:/data/` (fine up to a few GB; for more, open a Hydra ticket).
5. Submit with `kubectl apply -f job.yaml`, watch with `kubectl get pods` and `kubectl logs -f job/<name>`, cancel with `kubectl delete job <name>`. The wiki's job template sets `backoffLimit: 0`, `activeDeadlineSeconds: 86400`, `ttlSecondsAfterFinished: 259200`, an `nvidia.com/gpu` limit and the PVC at `/data`. Use a unique job name per run, version-tag images, and keep data and credentials out of images. `OOMKilled` → Hydra ticket for more memory.

### GSU AI policy

Policy: https://technology.gsu.edu/ai/policy-and-guidance/ — link it whenever you discuss running, hosting or choosing models on ARCTIC.

- **Prohibited:** GSU's AI Tool Review Registry (https://technology.gsu.edu/ai/tools/) lists **DeepSeek and Qwen as Prohibited**, including Qwen embedding models. Don't install them on university systems (ARCTIC included), connect them to university systems, or use them with university data. Downloading and running their open weights on ARCTIC counts. If the model itself is what you're studying (benchmarking, evaluations), ask GSU IIT for an exception first.
- **Approved tools (registry, October 2026):** Microsoft Copilot Chat (all faculty, staff and students), Microsoft 365 Copilot and Copilot Studio (extra license through IIT), and Google Gemini and NotebookLM (faculty and staff) are approved under GSU's licenses with enterprise data protection, up to Confidential data. Other tools are limited to public data, or prohibited. Check the registry before you use any AI tool with non-public data.
- **Data types:** https://technology.gsu.edu/ai/using-ai-with-intention/
- **Research disclosure:** follow URSA's guidance on AI in research: https://ursa.research.gsu.edu/ai-guidance-in-research/
- **Questions:** arc@gsu.edu, or the University AI Committee at aiatgsu@gsu.edu.

Source: https://arcwiki.rs.gsu.edu/en/LiteLLM/OpenCode, https://arcwiki.rs.gsu.edu/en/k8s/k8s-gpu-jobs, https://arcwiki.rs.gsu.edu/en/software/software, https://technology.gsu.edu/ai/tools/

---

## 9. Allocations

| Type | How you get it | Preemption | Use |
| --- | --- | --- | --- |
| **SRCA** (startup) | Automatic for GSU PIs | Yes | Development and benchmarking |
| **ARCA** (research) | Proposal, reviewed quarterly | No | Sustained research |
| **CIRA** (condominium) | Buy a node | No | Long-term heavy use |
| **IA** (instructional) | Request any time in Elpis | — | Classes |
| **PA** (paid) | Request any time | — | Prioritized access, billed monthly |

- **SRCA:** 10,000 CPU SU and 1,000 GPU SU, 200 GB project space, 150 active jobs per project, one per researcher per allocation period, not renewable, no ECCS. Submit a research request during the startup period.
- **ARCA:** 12 months, renewable (submit about 3 months before it ends); unused SUs are forfeited each year. Qualifies for Extended Computational Collaboration Support (ECCS) with a separate request.
- **ARCA submission windows:**

  | Submit | Notified | Allocation starts |
  | --- | --- | --- |
  | Jan 15 – Feb 1 | Mar 15 | Apr 1 |
  | Apr 15 – May 1 | Jun 15 | Jul 1 |
  | Jul 15 – Aug 1 | Sep 15 | Oct 1 |
  | Oct 15 – Nov 1 | Dec 15 | Jan 1 |

- **Documents (PDF only):** Main Document (3 pages max; returned unreviewed if longer), Code Performance and Scaling (2 pages max), Progress Report for renewals (2 pages max), publications entered in Elpis, CVs for ECCS requests (2-page NSF or NIH format), optional references. 1-inch margins; Arial, Courier New or Palatino at 10 pt or more, or Times New Roman or Computer Modern at 11 pt or more; page numbers.
- **CIRA:** $13K for a CPU node (1,300,000 CPU SU) or $37K for a GPU node (1,300,000 CPU SU and 54,000 GPU SU), for 3 years.
- **IA:** lasts one semester; all data is removed at the end. In Elpis, use the Class tab → Request a Class.
- **Paid:** the wiki lists CPU $0.04/hour, V100 $0.40/hour, RTX $0.10/hour, TITAN $0.14/hour. Confirm rates with ARCTIC before budgeting. Request in Elpis → Request Resource Allocation → ACIDS Paid Allocation or Paid Storage, with a business manager contact.

Source: https://arcwiki.rs.gsu.edu/en/policy/resource-allocations, https://arcwiki.rs.gsu.edu/en/home/elpis/document-requirements, https://arcwiki.rs.gsu.edu/en/home/elpis/request-instructional-allocation, https://arcwiki.rs.gsu.edu/en/home/elpis/paid-resources

---

## 10. Good practice

- **Login nodes:** edit files, submit jobs, move files. Don't run computations, large compiles or data processing there.
- **Storage:** keep HOME for configuration and scripts; install environments in `/userapp/<campusid>/`; copy results out of scratch before the job ends; keep your own copies of important data.
- **Workflow:** test interactively with a short time limit, then run production work as batch jobs. Shorter time limits schedule sooner. Clean up scratch.
- **Logins:** re-sign your SSH certificate every 3 months; run `iinit` every 2 weeks; keep PuTTY at 0.78 or newer on Windows.

Source: https://arcwiki.rs.gsu.edu/en/home/acids/how-to/slurm, https://arcwiki.rs.gsu.edu/en/iRODS/Storage

---

## 11. Support and acknowledgment

- **Hydra** (https://hydra.gsu.edu): support tickets. Projects → your project → Issues → New Issue, assigned to **arctic_developers**. To reply, open the issue → Edit → add your update → Submit.
- **Email:** arc@gsu.edu. Credential security incidents: help@gsu.edu or 404-413-HELP (4357).
- **Data recovery:** not available; ARCTIC has no backups.
- **Acknowledgment text:** "We acknowledge the use of the Advanced Research Computing Technology Innovation and Collaboration (ARCTIC) resources at Georgia State University. These resources were supported by the National Science Foundation through the Major Research Instrumentation (MRI) grant CNS-1920024 and the Research Infrastructure: CC Compute-Campus award for Pioneering Research-Oriented Flexible Cyberinfrastructure (PROFLEX-CI), grant 2430193."
- **Crediting ARCTIC staff:** when staff give extended hands-on support (writing code or pipelines, building software, processing data, sustained optimization), also thank them by name: "We thank [name], [job title], ARCTIC, Georgia State University, for [contribution]." See the Acknowledge page for co-authorship and other details.

| Service | URL |
| --- | --- |
| ARCTIC wiki | https://arcwiki.rs.gsu.edu |
| Elpis (allocations, certificates) | https://elpis.rs.gsu.edu |
| Hemera (web interface) | https://hemera.rs.gsu.edu |
| Hydra (support tickets) | https://hydra.gsu.edu |
| Globus | https://globus.org |
| Osiris (S3 keys) | https://osiris.rs.gsu.edu |
| LLM service API | https://litellm.rs.gsu.edu/v1 |
| GSU AI Tool Review Registry | https://technology.gsu.edu/ai/tools/ |

### Common errors

| Error | Cause | Fix |
| --- | --- | --- |
| `Permission denied (publickey)`, or a password prompt at SSH login | SSH certificate expired | Re-sign the key in Elpis, then `ssh-add` again |
| `no mutual signature algorithm` (macOS) | RSA key | Generate an ed25519 key and sign it |
| `Error connecting to agent` (Windows) | ssh-agent service off | `Set-Service -Name ssh-agent -StartupType Automatic -Status Running` (admin PowerShell) |
| `(AssocGrpCPUMinutesLimit)` | Allocation's CPU time used up | Request a research allocation |
| `(AssocGrpGRESMinutes)` | Allocation's GPU time used up | Request a research allocation |
| `(MaxCPUPerAccount)` | Too many CPUs in use at once | Wait for jobs to finish |
| `(Dependency)` | Waiting on another job | Wait, or check the `--dependency` job ID |
| "Home directory not available" | No active allocation | Contact your PI or request an allocation |
| `-827000 CAT_INVALID_USER` | iRODS login expired | `iinit` |
| Asked for an iRODS password repeatedly | HOME over quota, or login expired | Free space in HOME, then `iinit` |
| `$IRODS_WORK` empty | On a login node | Start a Slurm job or Hemera session |
| Home folder empty after the 2026 migration | Files weren't copied to the new home | In Globus, copy from ARCTIC-HOME to ARCTIC-HOME-NEW, then `iinit` |
| `cd: $SCRATCH: No such file or directory` | Scratch folder not created | `mkdir -p $SCRATCH` first |
| "Run 'conda init' before 'conda activate'" | Conda shell hook not loaded | `eval "$(conda shell.bash hook)"` |
| "no kernel image is available" with `pytorch/2.7.0` | V100 GPU | Request an L40S, A30 or RTX GPU |

Source: https://arcwiki.rs.gsu.edu/en/hydra/new-page, https://arcwiki.rs.gsu.edu/en/policy/acknowledge, https://arcwiki.rs.gsu.edu/en/home/slurm/troubleshoot, https://arcwiki.rs.gsu.edu/en/home/ssh/authentication-issues, https://arcwiki.rs.gsu.edu/en/iRODS/Troubleshooting
