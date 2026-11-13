# CBS Python guide

This guide describes how to manage Python environments on the CBS servers (VDI and SLURM). We recommend [Pixi](#Pixi-recommended) for new projects, and keeping your environments in your lab's datashare rather than in your home directory.

## Where to keep environments

- **Keep projects and environments in your lab's datashare**: `/nfs/<pi_last_name>` or `/cifs/<pi_last_name>`, depending on where your lab's data is. In the examples below, replace `<datashare>` with that path.
- **If you run jobs on SLURM, the environment must be on `/nfs`**: the SLURM cluster cannot access `/cifs` datashares. If your lab only has a `/cifs` datashare, your PI can request a `/nfs` datashare (see [CBS storage guide](https://hackmd.io/@CompCore/cbs_storage)).
- **Do not put large environments in your home directory.** It is limited to 20 GB, and your VDI and SLURM homes are separate, so an environment in your VDI home is not available on SLURM. See [Home directories](https://hackmd.io/@CompCore/cbs_storage#Home-directories).

## Pixi (recommended)

[Pixi](https://pixi.sh/) manages one environment per project, using Conda and PyPI packages. It records the exact package versions in a lock file, so the environment can be reproduced on another machine or later in time.

Pixi is available as a module. Load it in every terminal session and every job script where you use it:

```bash
module load pixi
```

### Create a project

```bash
cd <datashare>/projects
pixi init mri_analysis
cd mri_analysis
pixi add python=3.13 numpy scipy pandas nibabel nilearn
```

This creates `pixi.toml` (the list of packages you asked for) and `pixi.lock` (the exact versions installed). To add a package that is only available on PyPI, use `pixi add --pypi <package>`.

### Use the environment

From the project folder:

```bash
pixi shell                     # start a shell inside the environment (exit with `exit`)
pixi run python analysis.py    # or run a single command in the environment
```

### Use it in a SLURM job

`pixi run` works in job scripts without activating anything. The project must be on `/nfs`:

```bash
#!/bin/bash
#SBATCH --time=02:00:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G

module load pixi
cd /nfs/<pi_last_name>/projects/mri_analysis
pixi run python analysis.py
```

### Pixi cache

Pixi keeps downloaded packages in a cache, by default in your home directory. Projects do not need the cache to run, so it is always safe to clean:

```bash
pixi clean cache        # add --conda or --pypi to clean only one type, -y to skip the prompt
pixi info               # "Cache dir" shows where the cache is
```

To keep the cache out of your 20 GB home, move it to your datashare by adding this line to your `~/.bashrc`:

```bash
export PIXI_CACHE_DIR=<datashare>/pixi-cache
```

Your VDI and SLURM homes are separate, so set this in the `~/.bashrc` of each system where you use Pixi (on SLURM, use a path on `/nfs`).

## Other tools

### uv

[uv](https://docs.astral.sh/uv/) is a fast, pip-compatible package and project manager for Python packages from PyPI. It does not install Conda packages. It is available as a module:

```bash
module load uv
cd <datashare>/projects
uv init mri_analysis
cd mri_analysis
uv add numpy nibabel
uv run python analysis.py
```

uv keeps its cache in `~/.cache/uv`; clean it with `uv cache clean`.

### Python `venv`

`python3 -m venv` creates an environment with the operating system's Python (on the VDI, Ubuntu 22.04's Python 3.10). This version is old and cannot be changed, so we recommend Pixi or uv instead. If you need a plain virtual environment with a newer Python, uv can create one:

```bash
module load uv
uv venv --python 3.13 <datashare>/envs/mri_venv
source <datashare>/envs/mri_venv/bin/activate
uv pip install numpy scipy nibabel
```

### Conda / Mamba

There is no Conda or Mamba module on the CBS servers. If you already have Conda-based workflows, install your own copy of [Miniforge](https://github.com/conda-forge/miniforge) (which uses the community `conda-forge` channel and includes `mamba`) in your datashare, not in your home directory.

```bash
mamba create -p <datashare>/envs/mri_env python=3.13 numpy scipy nibabel
conda activate <datashare>/envs/mri_env
```

In a SLURM job script (the installation and environment must be on `/nfs`):

```bash
source /nfs/<pi_last_name>/miniforge3/etc/profile.d/conda.sh
conda activate /nfs/<pi_last_name>/envs/mri_env
python analysis.py
```

## Cleaning up

Package caches in your home directory can grow large over time. Clean them regularly:

| Tool | Command |
|---|---|
| Pixi | `pixi clean cache` |
| uv | `uv cache clean` |
| Conda / Mamba | `conda clean --all` |
| pip | `pip cache purge` |

Avoid `pip install --user`: it installs packages into `~/.local/lib/python3.*`, outside any environment, where they often conflict with your environments. If that folder exists and you do not need it, you can delete it (check its contents first):

```bash
rm -rf ~/.local/lib/python3.*
```

See [Home directories](https://hackmd.io/@CompCore/cbs_storage#Home-directories) for how to check your home usage and other things to clean up.

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

Back to the [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers), the main page of the CBS wiki.
