# CBS SLURM server guide

The CBS SLURM cluster is the CBS high-performance computing cluster for GPU and heavy computational workloads. Instead of running programs directly, you submit **jobs** to the SLURM scheduler, which runs them on the compute nodes when the requested resources (CPUs, memory, GPUs, time) are available.

The cluster is available to **Power user accounts** only, and can only access **OneFS 2025 datashares** (`/nfs`). See [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers) for accounts and fees. If you are new to SLURM, the Alliance's [What is a scheduler?](https://docs.alliancecan.ca/wiki/What_is_a_scheduler%3F) is a good introduction.

## Cluster overview

| Node type | Nodes | CPU model | CPU cores | RAM | Local scratch | GPUs per node | Partition |
|---|---|---|---|---|---|---|---|
| L40S GPU nodes | 7 | Intel Xeon Gold 6448Y | 128 | 500 GB | 7 TB | 2 × NVIDIA L40S | `hx` |
| A100 GPU nodes | 3 | Intel Xeon Gold 6142 | 64 | 300 GB | 1 TB | 2 × NVIDIA A100 | `vx` |

### Partitions

- **`hx`**: newer CPUs, more cores, RAM, and local scratch per node, and NVIDIA L40S GPUs. Best for large CPU and GPU workloads and data-heavy pipelines.
- **`vx`**: NVIDIA A100 GPUs. Best for GPU workloads that benefit from the A100.
- **`all`** (default): both `hx` and `vx` nodes. If you do not set `--partition`, your job can run on either node type, so different runs may use different hardware. If hardware consistency matters for your results, set `--partition` explicitly.

### Limits

| | Limit |
|---|---|
| Maximum job time | **48 hours** (all partitions) |
| Interactive sessions | **5 hours** |
| Home directory | **20 GB**, separate from your VDI home |
| `/nfs/scratch` | 25 TB shared; files older than **30 days** are deleted |
| `/localscratch` | About 7 TB (`hx`) or 1 TB (`vx`) per node; deleted when the job ends |
| Login node | CPU and memory per user are capped |
| Internet access | Available on the login node and all compute nodes |

## Using the CBS SLURM cluster

### Connecting

Connect to the login node with SSH:

```bash
ssh <westernid>@rri-cbs-slurm
```

You can connect from the CBS VDI servers or from the UWO network. From off campus, connect through the [CBS VDI servers](https://hackmd.io/@CompCore/cbs_vdi).

> [!Note] First connection
> New users are configured by a setup script that runs every hour. The first time you use the cluster, connect and disconnect once, wait one hour, then connect again.

#### The login node

The login node is a small virtual machine shared by all users. Use it only for:

- Editing files and scripts
- Managing and transferring data
- Submitting and monitoring jobs
- Light setup tasks, such as compiling small programs

> [!Caution]
> Do not run computations on the login node, including large software builds, data processing, or installing large Python environments. Submit them as jobs or run them in an [interactive session](#Interactive-jobs). CPU and memory on the login node are capped per user, so heavy work there will run slowly or fail.

### Storage for jobs

| Location | Use it for | Notes |
|---|---|---|
| `/nfs/<pi_last_name>` | Your lab's data, projects, and environments | OneFS 2025 lab datashare. Legacy `/cifs` datashares are **not** available on the cluster. |
| `/nfs/scratch` | Staging data before and after jobs, especially if your lab has no `/nfs` datashare yet | Shared by all users; files older than 30 days are deleted |
| `/localscratch` | Running your job | Fast local disk, private to each job, **deleted when the job ends**. `/tmp` points here. |
| Home (`~`) | Scripts and small configuration files | 20 GB; not for data |

Reading and writing large amounts of data directly on `/nfs` during a job is slow and affects other users. The recommended workflow is:

1. Copy the input data from `/nfs/<pi_last_name>` (or `/nfs/scratch`) to `/localscratch`.
2. Run the computation in `/localscratch`.
3. Copy the results back to `/nfs/<pi_last_name>` (or `/nfs/scratch`) **before the job ends**.

See the [CBS storage guide](https://hackmd.io/@CompCore/cbs_storage) for datashares, home directories, and how to request a lab datashare on `/nfs`.

### Writing a job script

A job script is a shell script with `#SBATCH` lines that tell SLURM what resources you need:

| Option | What it sets | Example |
|---|---|---|
| `--time` | Maximum run time (**required**, up to 48 h) | `#SBATCH --time=12:00:00` |
| `--cpus-per-task` | Number of CPU cores (**required**) | `#SBATCH --cpus-per-task=4` |
| `--mem` | Memory per node (**required**) | `#SBATCH --mem=16G` |
| `--gpus-per-node` | Number of GPUs (GPU jobs only) | `#SBATCH --gpus-per-node=1` |
| `--partition` | `hx`, `vx`, or `all` (default) | `#SBATCH --partition=hx` |
| `--job-name` | Name shown in the queue | `#SBATCH --job-name=myjob` |
| `--output` / `--error` | Log files (`%j` is replaced by the job ID) | `#SBATCH --output=slurm-%j.out` |

Example: run FreeSurfer `recon-all` on one subject, using `/localscratch` as described in [Storage for jobs](#Storage-for-jobs):

```bash
#!/bin/bash
#SBATCH --job-name=recon-all_sub-01
#SBATCH --time=24:00:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --output=slurm-%j.out

SUBJECT=sub-01
INPUT=/nfs/<pi_last_name>/myproject/rawdata/${SUBJECT}/anat/${SUBJECT}_T1w.nii.gz
OUTPUT=/nfs/<pi_last_name>/myproject/derivatives/freesurfer

module load freesurfer/8.2.0

# 1. Copy the input to the job's local scratch folder
export SUBJECTS_DIR=/localscratch/freesurfer
mkdir -p "$SUBJECTS_DIR"
cp "$INPUT" /localscratch/

# 2. Run the computation in /localscratch
recon-all -s "$SUBJECT" -i "/localscratch/${SUBJECT}_T1w.nii.gz" -all -threads "$SLURM_CPUS_PER_TASK"

# 3. Copy the results back to the lab datashare before the job ends
mkdir -p "$OUTPUT"
rsync -a "$SUBJECTS_DIR/$SUBJECT" "$OUTPUT/"
```

> [!Tip] Jobs longer than 48 hours
> If your workflow needs more than 48 hours, split it into steps or add checkpoints, so that a new job can continue from the saved intermediate results.

### Submitting and monitoring jobs

| Task | Command |
|---|---|
| Submit a job | `sbatch my_job_script.sh` |
| List your jobs | `squeue -u $USER` |
| Cancel a job | `scancel <jobID>` |
| Show your job history | `sacct -u $USER` |
| Show details of a job | `scontrol show job <jobID>` |
| Show nodes and their state | `sinfo` |
| Check how efficiently finished jobs used their resources | `reportseff <jobID>` (see [Using resources efficiently](#Using-resources-efficiently)) |

### Interactive jobs

For debugging or setup, you can request an interactive session on a compute node. When the resources are allocated, you get a shell on the node:

```bash
salloc --time=05:00:00 --cpus-per-task=2 --mem=4G                      # CPU only
salloc --time=05:00:00 --cpus-per-task=2 --mem=4G --gpus-per-node=1    # with a GPU
salloc --time=05:00:00 --cpus-per-task=2 --mem=4G --partition=vx       # on a specific partition
```

Interactive sessions are limited to **5 hours**.

#### Graphical applications (X11)

To run graphical applications, connect with X11 forwarding and add `--x11` to `salloc`:

```bash
ssh -X <westernid>@rri-cbs-slurm
salloc --time=05:00:00 --cpus-per-task=2 --mem=8G --x11
```

However, for visualization and other graphical work, we recommend using the [CBS VDI](https://hackmd.io/@CompCore/cbs_vdi) for a better experience.

### Software

The cluster uses modules (Lmod) for software. Load the modules you need inside your job script, always with a version:

```bash
module load freesurfer/8.2.0
```

See the [CBS software guide](https://hackmd.io/@CompCore/cbs_software) for module commands, containers, and how to request new software.

Notes:

- The cluster has the same modules and versions as the CBS VDI.
- Apptainer is loaded by default. Shared container images are in `/srv/containers`.
- For recommended Python practices, see the [Python guide](https://hackmd.io/@CompCore/python_cbs).
- The `kslurm` module is a wrapper with simpler commands for submitting jobs (`kbatch`) and starting interactive sessions (`krun`); load it with `module load kslurm`. See the [kslurm documentation](https://kslurm.readthedocs.io/en/latest/) for details. For example:
  ```bash
  kbatch 12:00 16 24G recon-all <recon-all-args>   # 12-hour job, 16 cores, 24 GB
  krun 4 3:00 15G gpu                               # interactive: 4 cores, 3 hours, 15 GB, 1 GPU
  ```

### Using resources efficiently

#### Request realistic resources

Request only the resources your job needs. Over-requesting CPUs, memory, GPUs, or time increases your queue time and keeps resources idle that other users could be using. The best way to size your requests is to check what your previous jobs actually used. You can do this with `reportseff`:

```bash
module load reportseff

reportseff <jobID>                          # one job (for an array job, all its tasks)
reportseff -u $USER                         # all your jobs from the last week
reportseff -u $USER --since d=1             # your jobs from the last day
```

Running `reportseff` with no arguments in a folder of SLURM output files (`slurm-<jobID>.out`) reports on those jobs.

| Column | Meaning | What to do |
|---|---|---|
| **CPUEff** | Share of the requested CPU time actually used | If it is low (e.g. below 50%), request fewer `--cpus-per-task`, or check that your program is set up to use multiple threads. |
| **MemEff** | Peak memory used, as a share of the memory requested | If it is low, lower `--mem` (keep a margin of about 20%). If it is near 100% or the job ran out of memory, increase it. |
| **TimeEff** | Elapsed time, as a share of the time limit | If it is low, lower `--time`. Jobs with shorter time limits can often start sooner. |

#### Prepare your environment in advance

Although all nodes have internet access, prepare your working environment before running jobs (download datasets, build containers, create Python environments) instead of downloading at the start of every job. Jobs then start faster, are more reproducible, and do not fail when an external site is slow or unavailable. If a setup step needs substantial resources, run it in an [interactive session](#Interactive-jobs) rather than on the login node.

## Additional resources

The Digital Research Alliance of Canada documentation has detailed explanations of SLURM:

- [What is a scheduler?](https://docs.alliancecan.ca/wiki/What_is_a_scheduler%3F)
- [Running jobs](https://docs.alliancecan.ca/wiki/Running_jobs)
- [Using GPUs with SLURM](https://docs.alliancecan.ca/wiki/Using_GPUs_with_Slurm)
- [MATLAB](https://docs.alliancecan.ca/wiki/MATLAB)

Some details (partitions, limits, storage paths) differ on the CBS cluster; when in doubt, this guide applies.

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

Back to the [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers), the main page of the CBS wiki.
