# CBS storage guide

Research data on the CBS servers is stored in **datashares** on two file servers:

| | OneFS 2025 (`/nfs`) | Legacy 2018 (`/cifs`) |
|---|---|---|
| In use since | September 2025 | 2018 |
| Available on | CBS VDI and SLURM | CBS VDI only |
| Mount from your own computer | SMB (recommended) or NFS | SMB |
| Storage fees | Standard rate | Reduced rate, because of the hardware's age and limited SLURM compatibility |

See [Storage fees](https://hackmd.io/@CompCore/cbs_servers#Storage-fees) for the rates.

> [!TIP] Do I need to migrate all my data to the new file server?
> No, unless you are going to make heavy and repeated use of the data on the SLURM cluster.


## Lab datashares and project shares
There are two kinds of shared storage:

- **Lab datashare (PI_data):** the main datashare of a lab, named after the PI (`<pi_last_name>`). Members of the lab have access to it.
- **Project share (PI_projects):** a folder meant to be shared with other groups besides the PI's own lab, for collaborative projects. Project shares are on the Legacy 2018 storage, at `/cifs/<pi_last_name>_projects`, so they are available on the VDI but not on the SLURM cluster. A PI's project shares and lab datashare share the same requested storage allocation (and fees).


## Requesting datashares and access
Datashares and access to them are managed by the IT Helpdesk, not by CBS support staff. All requests use the forms in [CBS servers guide: Account changes](https://hackmd.io/@CompCore/cbs_servers#Account-changes), which create a ticket with the IT Helpdesk automatically:

- **New PIs** get their lab datashare with their account (PI account form).
- **Existing PIs** request a new datashare (e.g., on OneFS 2025), more storage, or a project share with the PI account changes form.
- **Users** request access to a lab datashare or project share (their own PI's or a collaborator's) with the user account changes form. Be ready to provide your Western ID, the datashare you need (e.g., `/nfs/<pi_last_name>`), and a brief explanation of the project.

If you have trouble with the forms, contact [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca).


## Accessing datashares on the CBS servers
On the CBS VDI and SLURM servers, datashares are mounted automatically at `/nfs/<pi_last_name>` (OneFS) or `/cifs/<pi_last_name>` (Legacy). Because they are automounted, they do not appear in directory listings until you access them directly:

```bash
cd /nfs/<pi_last_name>
```

Files remain accessible as long as your Kerberos authentication ticket is valid (see the [CBS VDI guide](https://hackmd.io/@CompCore/cbs_vdi#Authentication-tickets-and-session-management)).

The SLURM cluster can only access OneFS (`/nfs`) datashares. If your lab does not have one, you can copy data to the shared scratch space `/nfs/scratch` before using it on the cluster (files there are deleted after 30 days). See the [CBS SLURM server guide](https://hackmd.io/@CompCore/cbs_slurm).


## Home directories
Every user has a home directory (`~` or `$HOME`) on each CBS system. It is meant for scripts, configuration files, and small Python environments, **not** for research data.

> [!Important]
> - **VDI and SLURM have separate home directories.** Files in your VDI home are not visible on SLURM, and vice versa. To use the same files on both systems, keep them in a `/nfs/<pi_last_name>` datashare.
> - Each home directory is limited to **20 GB**.

### Checking your usage

Two disk usage tools are available as modules on both the VDI and SLURM:

- **`dua`** — quick summary of how much space you are using:
  ```bash
  module load dua
  dua ~                  # total size of your home directory
  dua ~/.[!.]* ~/*       # size of each file and folder, including hidden ones
  ```
- **`gdu`** — browse your home interactively to find (and delete) what takes up space:
  ```bash
  module load gdu
  gdu ~
  ```

### What not to keep in your home

Store these in your PI's datashare instead:

- Research data and datasets
- Large Python or Conda environments, and Conda/Miniforge installations (see the [Python guide](https://hackmd.io/@CompCore/python_cbs))
- Container images (`.sif` files). Check the shared images in `/srv/containers` first (available on both the VDI and SLURM).
- Software source code and builds

> [!Note] Need software?
> Do not download or install software in your home directory. Most research software is already installed on the VDI and SLURM and available through environment modules: run `module avail` to see what is there. If you need something that is missing, contact [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca) and we can install it for everyone. See the [CBS software guide](https://hackmd.io/@CompCore/cbs_software) for details.

### Cleaning up

These folders (most of them hidden) are the most common reasons a home directory fills up:

| Location | What it contains | How to clean it |
|---|---|---|
| `~/Downloads` | Files downloaded with the web browser (installers, archives, datasets) and then forgotten | Delete what you no longer need, or move it to your PI's datashare |
| `~/.local/share/Trash` | Files deleted with the file manager on the VDI | Empty the trash (see [below](#Deleted-files-and-the-trash)) |
| `~/.cache` | Download caches from pip, Pixi, web browsers, and other tools | `pip cache purge`, `pixi clean cache`; other contents can usually be deleted, as tools re-create what they need |
| `~/.conda`, `~/.local/lib/python3.*` | Conda package caches; packages installed with `pip install --user` | `conda clean --all`; see the [Python guide](https://hackmd.io/@CompCore/python_cbs) |
| `~/.apptainer/cache` | Downloaded container layers | `apptainer cache clean` |


## Deleted files and the trash

> [!Caution] Empty your trash
> On the VDI, deleting files with the file manager does **not** free up space: the files are moved to the **Trash** and keep using storage until you empty it.
> - Files deleted from your home directory go to `~/.local/share/Trash` and still count against your 20 GB home quota.
> - Files deleted inside a datashare are moved to a hidden `.Trash-<user_id>` folder in that datashare, where they still count toward your lab's storage (and fees).
>
> Empty the trash regularly: right-click the Trash icon and select **Empty Trash**, or run `gio trash --empty` in a terminal.

Files deleted with `rm` in a terminal are removed immediately and do not go to the trash.


## Local mounting (accessing storage from your computer)
You can mount your datashares on your own computer when it is connected to a Western network (wired, uwosecure-v2 Wi-Fi, or eduroam). From off campus, see [Off-campus access](#Off-campus-access).

Connect with **SMB** (also called CIFS), using these addresses. Replace `<pi_last_name>` with your PI's datashare name:

| Datashare | macOS | Windows | Linux |
|---|---|---|---|
| OneFS 2025 lab datashare | `smb://smb-cbs.schulich.uwo.ca/<pi_last_name>` | `\\smb-cbs.schulich.uwo.ca\<pi_last_name>` | `//smb-cbs.schulich.uwo.ca/<pi_last_name>` |
| Legacy 2018 lab datashare | `smb://rri-cbssrv.schulich.uwo.ca/<pi_last_name>_data$` | `\\rri-cbssrv.schulich.uwo.ca\<pi_last_name>_data$` | `//rri-cbssrv.schulich.uwo.ca/<pi_last_name>_data$` |
| Legacy 2018 project share | `smb://rri-cbssrv.schulich.uwo.ca/<pi_last_name>_projects$` | `\\rri-cbssrv.schulich.uwo.ca\<pi_last_name>_projects$` | `//rri-cbssrv.schulich.uwo.ca/<pi_last_name>_projects$` |

Log in with `<westernid>@uwo.ca` and your Western password.

### macOS
1. In Finder, select **Go → Connect to Server…**
2. Enter the macOS address from the table and select **Connect**.
3. Log in with `<westernid>@uwo.ca`.

### Windows
1. Open File Explorer and type the Windows address from the table in the address bar.
2. Log in with `<westernid>@uwo.ca`.

### Linux

Mount each file server in its own folder, so that Legacy and OneFS datashares never use the same mount point:

| Datashare | Mount point |
|---|---|
| Legacy 2018 | `/mnt/cbs-legacy/<pi_last_name>` |
| OneFS 2025 | `/mnt/cbs-onefs/<pi_last_name>` |

#### SMB (recommended)

```bash
sudo apt install cifs-utils                              # if not already installed
sudo mkdir -p /mnt/cbs-onefs/<pi_last_name>              # or /mnt/cbs-legacy/<pi_last_name>
sudo mount -t cifs \
  -o username=<westernid>,domain=UWO,uid=$(id -u),gid=$(id -g),vers=3.1.1 \
  //smb-cbs.schulich.uwo.ca/<pi_last_name> \
  /mnt/cbs-onefs/<pi_last_name>
```

For a Legacy datashare, use its Linux address from the table and the `/mnt/cbs-legacy/<pi_last_name>` mount point. All mounted files appear as owned by your local user.

#### NFS (OneFS only)

NFS is faster for heavy processing, but only works on lab Linux machines that have been pre-registered for NFS access, and requires a Kerberos ticket. Mount a OneFS datashare with SMB **or** NFS, not both.

```bash
sudo apt install nfs-common                              # if not already installed
sudo mkdir -p /mnt/cbs-onefs/<pi_last_name>
kinit <westernid>@UWO.CA                                 # get a Kerberos ticket
klist                                                    # check that the ticket is valid
sudo mount -t nfs -o vers=4.1,sec=krb5 \
  nfs-cbs.schulich.uwo.ca:/ifs/cbs/<pi_last_name> \
  /mnt/cbs-onefs/<pi_last_name>
```

> [!Note]
> On Linux, mounts do not survive a reboot: repeat the mount command after restarting, or add the datashare to `/etc/fstab`.


## Off-campus access

- **All users:** access your data through the [CBS VDI servers](https://hackmd.io/@CompCore/cbs_vdi).
- **Local mounting** from off campus requires Western's VPN ([WROAMS](https://remote.uwo.ca/), using the Fortinet client). WROAMS is currently limited to Western staff and faculty, so students should use the CBS VDI servers instead.

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

Back to the [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers), the main page of the CBS wiki.
