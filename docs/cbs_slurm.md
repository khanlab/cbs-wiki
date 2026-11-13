# CBS SLURM server guide

This guide introduces how to use the CBS SLURM cluster for computational research. The cluster uses the Simple Linux Utility for Resource Management (SLURM) that allows multiple users to **share computing resources fairly and efficiently**. Work is submitted to SLURM in the form of **batch jobs** or **interactive sessions**, which are automatically scheduled and distributed to the compute nodes without interfering with other users.


The cluster includes **10 high-performance compute nodes** with varying CPU, memory, storage, and GPU configurations. These nodes are designed to support a wide range of compute-intensive and data-heavy research applications.

The CBS SLURM cluster consists of two partitions, **`hx`** and **`vx`**, which provide access to different node types optimized for high-performance and GPU-accelerated workloads:

| Node Type      | Nodes | CPU Model                      | CPU Cores | RAM     | Local tmp storage | GPUs per Node  | Partition |
| -------------- | ----- | ------------------------------ | --------- | ------- | ------------- | --------------- |  ----------- |
| L40 GPU Nodes  | 7     | Intel Xeon Gold 6448Y          | 128       | 500+ GB | 7 TB          | 2 × NVIDIA L40  |  hx           |
| A100 GPU Nodes | 3     | Intel Xeon Gold 6142           | 64        | 300 GB  | 1 TB          | 2 × NVIDIA A100 |  vx           |

> [!Note]
> Only `vx` nodes and the login node have internet access.

This guide explains how to:
- Connect to the cluster and understand SLURM.
- Request CPU, GPU, and memory resources.
- Store and manage data across `/nfs/<pi_last_name>`, `/localscratch`, and `/nfs/scratch`.
- Submit batch and interactive jobs.
- Use modules to manage software.

If you are already familiar with SLURM servers you can go directly to the [SLURM CBS server Notes and Best Practices](#SLURM-CBS-server-Notes-and-Best-Practices) section.

## 1.What is SLURM and a Job Scheduler?


- **SLURM** (Simple Linux Utility for Resource Management) is a **job scheduler**.  
- A job scheduler manages **when, where, and how jobs run** on a shared computing cluster.  
- Instead of running programs directly, you **submit jobs to SLURM**, which allocates resources (CPU, memory, GPU, time) and executes your workload on the compute nodes.  

This ensures **fairness**, **efficiency**, and **stability** in a multi-user research environment.


## 2. Connecting to the Cluster

To log in to the **login node**:

```bash
ssh your_uwo_username@rri-cbs-slurm
```
__Note__: An automatic setup script runs every hour to configure new users. If this is your first time connecting to the SLURM server, please:
- Connect and disconnect once.
- Wait one hour.
- Then reconnect.

> [!Caution]
> **NEVER** run computations directly on the **login node**.  
> The login node is only for **submitting jobs, file management, and light setup**.  All computational work should be submitted via SLURM.

> [!Important]
> - Direct ssh access is only possible through the CBS VDI servers or when you are connected to the UWO network.
> - For access off-campus you should connect through the CBS VDI servers.

> [!Tip]
> You can use `-X` to enable **X11 forwarding** if you require GUI applications in interactive sessions (see [section 9](#9.-Interactive-Jobs-(with-X11-Forwarding))).
---

## 3. Submitting a Job

Jobs are submitted using:

```bash
sbatch my_job_script.sh
```

Where `my_job_script.sh` is a shell script that describes:
1. The resources you need (CPUs, memory, GPUs, runtime).  
2. The commands to run.  



See [section 7](#7.-Job-Script-Template) for an example template of a job script.

> [!TIP] Best practice
> Jobs have a maximum runtime of **48 hours**. If your workflow is longer, design your code with checkpoints so you can restart from intermediate results.

## 4.  SLURM Flags for Resource Requests

Add these to your job script as `#SBATCH` directives:

| Flag              | Purpose                                       | Example                           | Requirement          |
| ----------------- | --------------------------------------------- | --------------------------------- | -------------------- |
| `--time`          | maximum runtime (required for all jobs)       | `#SBATCH --time=12:00:00`         | must                 |
| `--cpus-per-task` | number of CPU cores per task                  | `#SBATCH --cpus-per-task=4`       | must                 |
| `--mem`           | memory required per node                      | `#SBATCH --mem=16G`               | must (if applicable) |
| `--gpus-per-node` | number of GPUs required                       | `#SBATCH --gpus-per-node=1`       | must (if GPU job)    |
| `--partition`     | select compute partition (`hx` or `vx`)       | `#SBATCH --partition=hx`          | should               |
| `--ntasks`        | number of tasks (useful for parallel jobs)    | `#SBATCH --ntasks=1`              | optional             |
| `--job-name`      | job name for tracking                         | `#SBATCH --job-name=myjob`        | optional             |
| `--output`        | save standard output log                      | `#SBATCH --output=slurm-%j.out`   | optional             |
| `--error`         | separate stderr log file                      | `#SBATCH --error=slurm-%j.err`    | optional             |

See “Cluster partitions” section for guidance on when to use hx vs vx.


## 5. Managing Software with **Modules**

The cluster uses **modules** (Lmod) to manage software environments.

- See available software:
  ```bash
  module avail
  ```

- Load a module:
  ```bash
  module load freesurfer/7.4.1
  ```

- Check loaded modules:
  ```bash
  module list
  ```

- Remove a module:
  ```bash
  module unload freesurfer/7.4.1
  ```

Load modules in your script or interactive session to have access to the software (See example in [section 7](#7.-Job-Script-Template)). For reproducibility, always specify exact module versions (e.g., freesurfer/7.4.1) and avoid relying on defaults.


## 6. Data Storage: `/nfs/<pi_last_name>`, `/nfs/scratch`, and `/localscratch`

> [!Caution] Home folder
> - Do not save data in your home folder. 
> - Home folder is meant to be used for scripts or small python environments. 
> - Data or large python environments should be saved in a persistent lab share.

### 🔒 Lab Shares: `/nfs/<pi_last_name>`

- Labs can request a **dedicated fileshare** for storing and processing project data:
  ```
  /nfs/<pi_last_name>
  ```
- These shares are provisioned on the **2025 OneFS fileserver** and are accessible from all **compute nodes**.
- **Note:** Older lab shares mounted via `/cifs/...` from the legacy **2018 fileserver** are **not accessible from the compute nodes**. All compute jobs should use the new `/nfs/<pi_last_name>` shares or `/nfs/scratch`.

> 💡 For details on the CBS servers storage, see the [CBS Server Storage Document](https://hackmd.io/@CompCore/cbs_storage).



### 📂 Shared Temporary Space: `/nfs/scratch`

- `/nfs/scratch` is a **shared, high-capacity (25 TB) space** accessible to all users on the compute nodes.
- Use it to **stage input/output data** before and after job runs — especially if your lab doesn’t yet have a `/nfs/<pi_last_name>` share.

> [!Caution] Scratch Policy
> - Files older than **30 days** are **automatically deleted**.  
> - This storage is **not backed up** and is intended **only for temporary use**.


### ⚡ Fast Local Storage: `/localscratch` on Compute Nodes

- Each compute node has a **7TB fast SSD** local disk.
- When your job starts, a **personal `/localscratch` folder** is automatically created **just for that job**.
- This folder is:
  - **Private to your job**
  - **Faster than network storage** (/nfs/...)
  - **Deleted at job completion/termination** (all contents are wiped)

> [!Tip] Best practice
>  Copy your data to `/localscratch`, run your job there, and move outputs back to `/nfs/<pi_last_name>` or `/nfs/scratch` **before the job finishes**.

Note: `/tmp` is mapped to `/localscratch` on compute nodes.

### 💬 Requesting Storage

- Contact [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca) to request a **new lab share** on `/nfs/<pi_last_name>`.
- For billing and quota details, refer to the [CBS Servers Document](https://hackmd.io/@CompCore/cbs_servers) or email: support-cbs-server@uwo.ca


## 7. Job Script Template

📋 **Example Workflow**

A typical script might:
1. **Copy data** from `/nfs/<pi_last_name>` or `/nfs/scratch` to `/localscratch`.
2. **Process data** using data in `/localscratch` and write logs locally.
3. **Checkpoint** outputs periodically (e.g., after each step in a pipeline) to `/nfs/scratch`. (Optional but recommended).
4. **Copy final results** from `/localscratch` to `/nfs/<pi_last_name>` or `/nfs/scratch` before the job exits.

Example: `my_job_script.sh`

```bash
#!/bin/bash
#SBATCH --job-name=myanalysis
#SBATCH --time=12:00:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --output=slurm-%j.out

# Optional: select partition (hx or vx). Default is all
# #SBATCH --partition=hx

# Load software
module load software/1.0

# Copy input data to local scratch (/localscratch)
rsync -av /nfs/lab/myproject/data/input.csv /localscratch

# Run computation
software_command /localscratch/input.csv > /localscratch/results.txt

# Copy results back to lab share
rsync -av /localscratch/results.txt /nfs/lab/myproject
```

Submit it with:
```bash
sbatch my_job_script.sh
```

## 8. Monitoring and Managing Jobs

- List your jobs:
  ```bash
  squeue -u <your_uwo_username>
  ```
- Cancel a job:
  ```bash
  scancel <jobID>
  ```
- Show job history:
  ```bash
  sacct -u <your_uwo_username>
  ```
- Detailed info on a running or completed job:
  ```bash 
  scontrol show job <jobID>
  ```
- List nodes on the cluster, what is up and down:
  ```bash 
  sinfo
  ```

## 9. Interactive Jobs

Sometimes you need an **interactive session** (e.g., for debugging or set-up).

> [!Caution] 
> Interactive sessions are limited to **5 hours**.

- Request an interactive session:
    ```bash
    salloc --time=05:00:00  --cpus-per-task=2 --mem=4G
    ```
- With GPUs:
    ```bash
    salloc --time=05:00:00 --cpus-per-task=2 --mem=4G --gpus-per-node=1
    ```
- Selecting a partition (default is 'all')
```bash
salloc --partition=vx --time=05:00:00 --cpus-per-task=2 --mem=4G
```

Once allocated, you will be placed inside a shell on a compute node where you can run commands interactively.

### GUI (X fowarding)
In case you need GUI or windows you need to start your interactive session with X fowarding enabled (using --x11 flag). The above example would become:

```bash
salloc --time=05:00:00  --cpus-per-task=2 --mem=8G --x11 
```
Make sure you connect to the SLURM cluster with X forwarding enabled:
```bash
ssh -X your_uwo_username@rri-cbs-slurm.fmd.uwo.pri
```


# SLURM CBS server Notes and Best Practices

### Cluster partitions
The cluster provides two partitions (`hx` and `vx`) that corresponds to different hardware environments. If no partition is specified, SLURM will assign the job to default 'all' partition that include both `hx` and `vx` partitions. This may result in different hardware being used across runs. If hardware consistency is relevant for your job, explicitly setting --partition is strongly recommended to ensure reproducibility and correct hardware selection.

The table at the beginning of this document provide hardware partitions details, but the key differences are:
#### `hx` partition:
- Newer **Intel Xeon Gold 6448Y** CPUs
- NVIDIA L40 GPU
- More CPU cores and RAM per node
- Larger local SSD storage for localscratch
- **No internet access** 
- Best for large-scale CPU + GPU workloads and data-heavy pipelines
        
#### `vx` partition:
- **Intel Xeon Gold 6142** CPUs 
- NVIDIA A100 GPUs 
- **Has internet access**  
- Best for: 
    - Environment setup (pip/conda installs, container pulls)  
    - GPU workloads optimized for A100   
    - Workflows requiring external downloads


### Internet access and initial setup

Compute jobs should **not require internet access**. Only the **`vx` partition nodes** and the login node have internet connectivity.

It is good practice to prepare your working environment beforehand (e.g., downloading datasets, building containers, or creating Python environments) so that jobs can run without external dependencies.

If a setup step requires both **internet access and compute resources**, it should be run in an interactive SLURM session on the **`vx` partition**, rather than on the login node, which is a small virtual machine not intended for workloads.

You can request an interactive session using `salloc` and explicitly selecting the `vx` partition:

```bash
salloc --time=05:00:00 --cpus-per-task=2 --mem=8G --partition=vx
```

This ensures you are placed on a node with:

* internet access (required for installations and downloads)
* appropriate compute resources for setup tasks



### **Avoid running heavy jobs on the login node**

The login node is intended for editing files, compiling code, and submitting jobs. Do not run computationally intensive workloads directly on the login node. Always submit jobs through SLURM.

### Use the `/localscratch` folder for computation

Reading and writing large amounts of data directly to the NFS filesystem during job execution can generate heavy network traffic and degrade performance for other users. Instead, **perform computations locally on the compute node using the `/localscratch` directory**, as described in the *Data Storage* section. Local disk access is typically much faster than NFS.

A common workflow is to copy input data to `/localscratch` at the start of the job, run all computations there, and copy the results back to `/nfs/<pi_last_name>` or `/nfs/scratch` before the job finishes. The `/localscratch` directory is automatically cleared after the job ends, so ensure all required outputs are copied before termination.

### Request realistic resources
Request only the resources required by your job. Over-requesting CPUs or memory can increase queue time and reduce cluster efficiency.

After a job finishes, you can check how many resources it actually used:

```bash
sacct -j <jobID> --format=JobID,Elapsed,MaxRSS,AllocCPUS
```
- Elapsed → total runtime
- MaxRSS → peak memory usage
- AllocCPUS → CPUs requested

(ToDo: Give users options/ideas in how to monitor their jobs used resources.)




### GUI and graphical interfaces

Although SLURM supports GUI applications through **X forwarding**, we recommend using the **CBS VDI** for visualization tasks or graphical applications. The VDI environment is generally more stable and provides a better user experience for GUI workflows.


### Containers

The CBS SLURM cluster uses **Apptainer** (formerly **Singularity**) for containerized workflows. Apptainer is available through the module system.

Commonly used containers are stored in:

```
/srv/containers
```

You are encouraged to reuse these images when possible to avoid unnecessary downloads. If a container is outdated or a commonly used tool is missing, please contact the administrators so it can be added for the community.


### Python

For recommended Python practices on the cluster, see the **Python guide**:

[https://hackmd.io/@CompCore/python_cbs](https://hackmd.io/@CompCore/python_cbs)
*(Work in progress)*

### kslurm (Simplified SLURM Wrapper)

**kslurm** is a wrapper that simplifies SLURM commands.

It provides two main commands:

* **kbatch** — submit batch jobs (non-interactive)
* **krun** — request interactive sessions

These commands use simplified argument syntax, allowing you to request resources without writing a full SLURM script or specifying multiple `--options`.

Examples:

Schedule a **12-hour job** with **16 cores** and **24 GB of memory**:

```bash
kbatch 12:00 16 24G recon-all <recon-all-args>
```

Request an **interactive session** with **4 cores**, **3 hours**, **15 GB of memory**, and **a GPU**:

```bash
krun 4 3:00 15G gpu
```

`kslurm` will be available as a module *(TODO)*.
See the full documentation for details:

[https://kslurm.readthedocs.io/en/latest/](https://kslurm.readthedocs.io/en/latest/)




# General information

## Billing Rates

CBS SLURM server is only available to Power users. Note that CBS SLURM only support the OneFS Storage datashare (/nfs). For further details and fees please check the [CBS servers](https://hackmd.io/@CompCore/cbs_servers) wiki entry.


## Additional resources

For detailed explanations and further reading we highly recommend Canada Alliance documentation on SLURM. 

https://docs.alliancecan.ca/wiki/What_is_a_scheduler

https://docs.alliancecan.ca/wiki/Running_jobs

https://docs.alliancecan.ca/wiki/Using_GPUs_with_Slurm

https://docs.alliancecan.ca/wiki/MATLAB

# Need help?
📧 Contact: support-cbs-server@uwo.ca