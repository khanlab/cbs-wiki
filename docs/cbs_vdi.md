# CBS VDI server guide
## What is the CBS-VDI servers?
The CBS VDI server provides virtual desktop infrastructure (VDI) using VMware technology to create virtual clones of Linux machines for user sessions. It is intended for research use only and offers different account types to accommodate various computational requirements.

The CBS VDI server runs Ubuntu 22.04.5 LTS and provides access to a variety of pre-installed software packages through environment modules. Users access the system through virtual machines that can be customized based on their computational needs, from basic research tasks to high-performance computing.

## VDI machines types
CBS offers two types of server usage, each designed to meet different computational needs:

- **CBS Basic:** Access to 4 cores and 8 GB of memory, ideal for regular computational tasks.
- **CBS Heavy (Only for Power User Account):** Access to 12 cores and 48 GB of memory for more demanding workloads.

For heavy pipelines and GPU acces, we recommend power users to consider the CBS [SLURM server](https://hackmd.io/@CompCore/cbs_slurm) instead. Choose the account type that best suits your workload. See [CBS servers information]([#Fee-schedule](https://hackmd.io/@CompCore/cbs_servers)) page for account fees.

## Accessing the VDIs
The CBS server uses VMware Virtual Desktop Infrastructure to provide virtual machines:

1. Download and install the [Omnissa Horizon Client](https://customerconnect.omnissa.com/downloads/info/slug/desktop_end_user_computing/omnissa_horizon_clients/8) (previously VMware Horizon Client).
2. Launch the Omnissa Horizon Client software and set up a new connection server
3. Use vdi.robarts.ca as the connection server address
4. Log in with your full UWO email address (e.g., 'username@uwo.ca') and your UWO password
5. Use the same credentials to log into the Ubuntu virtual machine (Usually not neccesary).

>[!Important] 
>If you were previously connected to vsecurity.robarts.ca, you must delete the old connection and add a new one pointing to vdi.robarts.ca.

## Managing your session

- **Disconnecting:** If you need to keep your virtual machine running (e.g., active processes), quit the Omnissa Horizon application or select "Disconnect" from the Connection menu. Your session will remain active and you can reconnect later
- **Screen locking:** If your screen locks due to inactivity, you can unlock it to continue your session and refresh your authentication ticket
- **Shutting down:** If you have no active processes, be considerate of shared resources and shut down your virtual machine by selecting "Log Out" from your Ubuntu account menu or the Omnissa Horizon Connection menu

> [!important]
> Session behavior depends on how you exit:
> - **Closing the Horizon client window:** This will terminate your VDI session
> - **Disconnecting properly:** Your session continues running and can be resumed
> - **Screen lock:** Session remains active and can be unlocked to continue

### Password management
Use your existing Western University credentials to access the CBS server. Password changes are managed through Western University's standard password change procedures, not through the CBS server directly.

### Authentication tickets and session management
The CBS server uses Kerberos authentication tickets from UWO that have specific lifespans:

- **Ticket lifespan:** 10 hours with automatic renewal
- **Maximum renewal period:** 7 days
- **Automatic renewal:** Tickets are automatically renewed approximately every 7 hours
- **Ticket reset:** New tickets are generated when you log in or unlock your console, resetting the 7-day renewal window

> [!Important]
> After 7 days of continuous use, you will need to log out and log back in (or unlock your screen if it's locked) to generate a new authentication ticket. This prevents loss of access to mounted storage during long-running sessions.


## Storage
### Storage locations
There are two primary locations for storing files:

1. **PI datashare (`/cifs/<pi_last_name>` or `/nfs/<pi_last_name>`):** Main storage locations for research data and files.
2. **Home directory (`/home/UWO/<username>`):** For small files, configuration files, or Python environments
    
See [CBS storage info](https://hackmd.io/@CompCore/cbs_storage) page for more details about the storate datashares.

### Important storage guidelines:

- Home directories should not exceed 20GB in size
- All files stored outside of PI datashares or home directories will be deleted when the virtual machine shuts down
- The system should only be used for research-related data and files, not as a primary backup.
- `/cifs` and `/nfs` folders are **automounted**, so they will not appear until accessed.
- Note: `/srv` is deprecated.

### Monitoring home storage usage
- To check your home directory size: `du -h /home/UWO/<username>`
- You can also use the command `gdu` to navigate/clean your home folder for unused files.

### Accessing datashares
Project datashares are automatically mounted and accessible at `/cifs/<pi_last_name>` or `/nfs/<pi_last_name>` (Depening in which datashare you lab have). The folders are automounted, so they will not appear in directory listings until you access them directly.

To access your PI's datashare:

1. Navigate directly to it: e.g. `cd /cifs/<pi_last_name>` or `cd /nfs/<pi_last_name>`
2. The folder will be automatically mounted when accessed
3. Files remain accessible as long as your authentication ticket is valid

For more information about accesing the datashares, see the [CBS storage info](https://hackmd.io/@CompCore/cbs_storage) wiki entry.
    
### External systems

**Digital Alliance integration:** Use `setup_sshfs_alias` from your home directory and follow the prompts to mount Digital Alliance systems (like Graham) on the VDI. You must have an active Digital Alliance account. After setup, use `graham` to mount and `unmount_graham` to unmount.

> [!Important]
> **If you're using sshfs to mount Digital Alliance storage:**
> Re-run the `setup_sshfs_alias` script to update your aliases after the system upgrade.
> Mounting now runs in the foreground (to support Duo 2FA), so you must keep the terminal window open after running the mount alias (e.g., `graham`)

## Available software
### Environment modules
Most software is installed in `/srv/software` and managed through environment modules. CBS VDI servers have a default profile with the most common modules already active, but you can further load/unload modules tailored for your research needs.

Basic module commands:
- List available modules: `module avail`
- Load a module: `module load software`
- Load specific version: `module load software/1.0.0`
- Show loaded modules: ‍‍`module list`
- Unload all modules: `module purge`
- Get help: `module`
    
## Contact
Need help?  
📧 Contact: support-cbs-server@uwo.ca