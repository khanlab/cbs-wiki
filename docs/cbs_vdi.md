# CBS VDI server guide

The CBS VDI servers provide Linux virtual desktops (Ubuntu 22.04.5 LTS) that you access through the Omnissa Horizon Client (formerly VMware Horizon). Use them for data access, visualization, and light to medium computations. They are for research use only.

## VDI machine types

| Machine | Cores | Memory | Available to |
|---|---|---|---|
| **CBS-Basic** | 4 | 8 GB | All accounts |
| **CBS-Heavy** | 12 | 48 GB | Power user accounts only |

For heavy pipelines and GPU work, Power users should use the [CBS SLURM server](https://hackmd.io/@CompCore/cbs_slurm) instead. See [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers#CBS-servers-fees) for account types and fees.

## Accessing the VDIs

1. Download and install the [Omnissa Horizon Client](https://customerconnect.omnissa.com/downloads/info/slug/virtual_desktop_and_apps/omnissa_horizon_clients/8).
2. Open the Horizon Client and add a new server: `vdi.robarts.ca`.
3. Log in with your full Western email address (`<westernid>@uwo.ca`) and your Western password.
4. Select the machine you want to use: **CBS-Basic**, or **CBS-Heavy** (only shown to Power user accounts).

You are usually logged into the Ubuntu desktop automatically. If you are asked to log in again (for example, when reconnecting to a session), use the same credentials. Your password is your Western password.

## Managing your session

> [!Important]
> **Closing the Horizon client does not end your session.** Closing the window or quitting the app only *disconnects* you: your virtual machine and everything running in it keep using shared resources. To end your session, **log out of Ubuntu** inside the VDI.

- **Disconnecting (session keeps running):** Close the Horizon client window, quit the app, or select "Disconnect" from the client menu. Your session and any running processes stay active, and you can reconnect later to resume. Use this when you have processes that need to keep running. Disconnected sessions are not kept indefinitely (see the automatic logout below).
- **Logging out (session ends):** When you have no active processes, be considerate of shared resources and log out by selecting "Log Out" from the Ubuntu account menu, or "Disconnect and Log Off" from the Horizon client menu. Unsaved work in open applications will be lost.
- **Screen locking:** If your screen locks due to inactivity, your session remains active. Unlock it to continue and refresh your authentication ticket.

> [!Caution] Automatic logout of disconnected sessions
> If you stay disconnected and do not reconnect, your session is **automatically logged out**, and any running processes and unsaved work are lost:
> - **Basic user accounts:** after **24 hours** disconnected
> - **Power user accounts:** after **48 hours** disconnected
>
> Reconnect before the limit to keep your session. For computations longer than this, use the [CBS SLURM server](https://hackmd.io/@CompCore/cbs_slurm) (Power users).

### Authentication tickets and session management
Access to your datashares depends on a Kerberos authentication ticket from Western:

- Tickets last 10 hours and are renewed automatically (about every 7 hours), for up to 7 days.
- A new ticket is created when you log in or unlock your screen, which restarts the 7-day period.

> [!Important]
> After 7 days of continuous use, log out and log back in (or lock and unlock your screen) to get a new ticket. Otherwise you will lose access to your datashares during long-running sessions.

## Storage

- **Datashares** are mounted automatically at `/nfs/<pi_last_name>` (OneFS) or `/cifs/<pi_last_name>` (Legacy), depending on where your lab's datashare is. They do not appear in directory listings until you access them directly, e.g. `cd /nfs/<pi_last_name>`.
- **Your home directory** (`/home/UWO/<westernid>`) is for small files and configuration. It is limited to 20 GB and is separate from your SLURM home directory. See [Home directories](https://hackmd.io/@CompCore/cbs_storage#Home-directories).
- **Files saved anywhere else** (outside your datashares and home directory) are deleted when the virtual machine shuts down.
- **Deleted files go to the Trash** and keep using space until you empty it (see [Deleted files and the trash](https://hackmd.io/@CompCore/cbs_storage#Deleted-files-and-the-trash)).
- The VDI is not a backup location: keep only research data and files there.

See the [CBS storage guide](https://hackmd.io/@CompCore/cbs_storage) for access permissions, project shares, and mounting datashares on your own computer.

### Sharing files with your computer (`~/tsclient`)
The Horizon client can share folders from your own computer with the VDI. Shared folders appear inside the VDI under `~/tsclient`. This only works with the installed Omnissa Horizon Client, not when connecting through a web browser.

1. In the Horizon client settings, open **Drive Sharing** (called *Drive & Folder Sharing* in some versions) and add the folders you want to share.
2. Connect to the VDI (or reconnect, if you were already connected).
3. Open `~/tsclient` in the VDI file manager or terminal: each shared folder appears there by name.

> [!Caution] Use `~/tsclient` only for small files
> Transfers through `~/tsclient` are slow, so use it only for a few small files, such as documents or figures. To move larger files or datasets, mount your datashare directly on your computer instead (see [Local mounting](https://hackmd.io/@CompCore/cbs_storage#Local-mounting-accessing-storage-from-your-computer)). Do not run analyses directly on files in `~/tsclient`.

### External systems

**Digital Alliance integration:** Use `setup_sshfs_alias` from your home directory and follow the prompts to mount Digital Alliance systems (like Graham) on the VDI. You must have an active Digital Alliance account. After setup, use `graham` to mount and `unmount_graham` to unmount.

> [!Important]
> **If you're using sshfs to mount Digital Alliance storage:**
>
> Re-run the `setup_sshfs_alias` script to update your aliases after the system upgrade.
>
> Mounting now runs in the foreground (to support Duo 2FA), so you must keep the terminal window open after running the mount alias (e.g., `graham`)

## Available software
Software on the VDI is provided through environment modules (Lmod), and containers can be run with Apptainer. A default set of commonly used modules (including `apptainer`, which other modules need) is loaded automatically when you log in; run `module avail` to see everything else.

See the [CBS software guide](https://hackmd.io/@CompCore/cbs_software) for how to use modules and containers, and how to request new software.

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

Back to the [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers), the main page of the CBS wiki.
