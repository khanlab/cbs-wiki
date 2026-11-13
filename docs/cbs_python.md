# Python Environment Best Practices on the CBS Cluster

This guide describes **recommended practices for managing Python environments** on the CBS servers (VDI and SLURM cluster). The goal is reproducibility, performance, and keeping shared storage healthy.

---

## TL;DR (Strong Recommendations)

* **Use `pixi` for new projects** (recommended default).
* Store **project environments outside your home directory** for medium/large projects.
* Use **`/nfs/<lab>` for persistent environments and projects**.
* Use **`/tmp` for temporary, node-local computations**.
* Regularly **clean caches** to avoid bloating your home directory.

---

## Storage Overview (Important)

| Location        | Purpose                      | Notes                                       |
| --------------- | ---------------------------- | ------------------------------------------- |
| `$HOME`         | Small configs, scripts       | Shared, quota-limited, fast for small files |
| `/nfs/<lab>`    | Projects & environments      | Persistent, shared within lab (recommended) |
| `/nfs/scratch`  | Temporary multi-node storage | Not permanent                               |
| `/tmp` | Fast node-local I/O          | Deleted when job ends                       |

⚠️ **Do not place large Python environments in `$HOME`**. This causes quota pressure and performance issues for everyone.

---

## Module System (Lmod) – Important

The cluster uses **Lmod** to manage software. Tools such as **Pixi** and **uv** are provided as modules.

Before using Pixi or uv, you **must load the corresponding module**:

```bash
module load pixi
```

(If you see multiple versions, use `module avail pixi` and follow lab recommendations.)

This applies both to **interactive sessions** and **SLURM jobs**.

---

## Recommended Tool: Pixi (Default Choice)

`pixi` is a modern, reproducible environment and project manager built on Conda packages. It is fast, lockfile-based, and well suited for scientific workflows.

### Why Pixi?

* Reproducible environments via `pixi.lock`
* One environment **per project** (best practice)
* Easy activation (including in SLURM)
* Clean separation between projects

---

### Creating a New Pixi Project

```bash
# Load Pixi via Lmod
module load pixi
```

```bash
# Move to your lab storage
cd /nfs/<lab>/projects

# Create a new project
pixi init mri_analysis
cd mri_analysis
```

This creates:

* `pixi.toml` – project configuration
* `pixi.lock` – exact, reproducible package versions

---

### Adding Packages

```bash
# Add common scientific packages
pixi add python=3.11 numpy scipy pandas nibabel nilearn

# Add PyTorch or other heavy deps if needed
pixi add pytorch torchvision -c pytorch
```

---

### Activating the Environment (Interactive)

Make sure the Pixi module is loaded:

```bash
module load pixi
```

```bash
pixi shell
```

You are now inside the project environment.

---

### Running Commands Without Activating a Shell

Useful for scripts and SLURM jobs:

```bash
pixi run python analysis.py
```

---

## Using Pixi in a SLURM Script

Always load the Pixi module inside your job script.

```bash
#!/bin/bash
#SBATCH --job-name=mri_analysis
#SBATCH --time=02:00:00
#SBATCH --mem=16G
#SBATCH --cpus-per-task=4

# Initialize module system (usually automatic, but safe to include)
source /etc/profile

# Load Pixi
module load pixi

cd /nfs/<lab>/projects/mri_analysis

# Option A: run directly (recommended)
pixi run python analysis.py

# Option B: activate shell (less common)
# pixi shell
# python analysis.py
```

✅ No `source activate` required.

---

## Alternative Tools (If You Prefer)

### Conda / Mamba

Good option if you already have Conda-based workflows.

#### Create Environment (Recommended Location)

```bash
mamba create -p /nfs/<lab>/envs/mri_env python=3.11 numpy scipy nibabel
```

#### Activate (Interactive)

```bash
source activate /nfs/<lab>/envs/mri_env
```

#### SLURM Script Example

```bash
source ~/miniconda3/etc/profile.d/conda.sh
conda activate /nfs/<lab>/envs/mri_env
python analysis.py
```

---

### Python `venv`

Best for **lightweight**, pure-Python environments.

#### Create Environment

```bash
python -m venv /nfs/<lab>/envs/mri_venv
```

#### Activate

```bash
source /nfs/<lab>/envs/mri_venv/bin/activate
pip install numpy scipy nibabel
```

⚠️ Not ideal for compiled neuroimaging packages.

---

## UV (Optional, Advanced Users)

`uv` is also provided via the **module system**:

```bash
module load uv
```

`uv` is a **very fast pip-compatible tool** for dependency resolution.

* Excellent for pure-Python projects
* Not Conda-based
* Limited support for compiled neuroimaging stacks

Use only if you know your dependency constraints well.

`uv` is a **very fast pip-compatible tool** for dependency resolution.

* Excellent for pure-Python projects
* Not Conda-based
* Limited support for compiled neuroimaging stacks

Use only if you know your dependency constraints well.

---

## Tool Comparison Summary

| Tool        | Best For                   | Pros                      | Cons                     |
| ----------- | -------------------------- | ------------------------- | ------------------------ |
| **Pixi**    | New projects (recommended) | Reproducible, clean, fast | Newer tool               |
| Mamba/Conda | Heavy scientific stacks    | Mature ecosystem          | Can bloat home           |
| venv        | Small projects             | Simple, standard          | Limited packages         |
| uv          | Fast installs              | Very fast                 | Not ideal for MRI stacks |

---

## Cache and Disk Usage Management (Very Important)

### Where Python Stuff Accumulates

* `~/.cache/`
* `~/.conda/`
* `~/.local/lib/pythonX.Y/`
* `~/.pixi/`

These can grow **very large** over time.

---

### Cleaning Pixi Cache

```bash
pixi cache prune
```

You may also remove the cache directory manually:

```bash
rm -rf ~/.pixi/cache
```

✅ Safe: Pixi projects **do not depend** on the cache at runtime.

---

### Cleaning Conda / Mamba

```bash
conda clean --all
```

Removes:

* Downloaded tarballs
* Unused package caches

---

### Cleaning pip Cache

```bash
pip cache purge
```

---

### About `~/.local/lib/pythonX.Y`

This directory appears when:

* `pip install --user` is used
* No virtual environment was active

⚠️ Packages here are **not isolated** and often cause conflicts.

Safe cleanup (after checking):

```bash
rm -rf ~/.local/lib/python3.*
```

---

## Pixi Cache Location (Recommended)

### Setting `PIXI_CACHE_DIR`

For large workflows, move the cache out of `$HOME`:

```bash
export PIXI_CACHE_DIR=/nfs/<lab>/pixi-cache
```

Add this to your shell init or a system profile script.

### Does Pixi Require the Cache?

* ❌ No, projects do **not** require cache contents to run
* ✅ Cache is only for downloaded packages
* Safe to delete anytime (Pixi will re-download)



## Final Recommendations

* One project = one environment
* Prefer Pixi unless you have a strong reason not to
* Keep `$HOME` clean
* Use `/nfs/<lab>` for persistent work
* Clean caches regularly

If in doubt, ask your cluster admins before installing large environments.

