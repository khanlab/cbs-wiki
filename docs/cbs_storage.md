# CBS storage info

CBS servers offer two filesystem:

- **OneFS 2025 storage (/nfs)**: Since September 2025, we have a new file server which offers improved performance, and compatibility to be used with the new SLURM cluster (i.e., being capable of mounting as NFS as well as CIFS).
- **Legacy 2018 storage (/cifs)**: This file server was the default before September 2025, which will continue to be available for users and backed up. However, due to the hardware age and limited compatibility with the SLURM cluster, this type of storage will be charged at a lower rate (see [Storage fees](https://hackmd.io/@CompCore/cbs_servers#Storage-fees)).

> [!TIP] Do I need to migrate all my data to the new filesystem? 
>No, unless you are going to be making heavy and repeated use of the data on the SLURM cluster. 

See [**CBS servers info**](https://hackmd.io/@CompCore/cbs_servers) for the corresponding fees. To request a datashare please contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

> [!IMPORTANT] 
> - **VDI servers** are capable of mounting both **Legacy and OneFS**.
> - **SLURM server** cluster can **only** mount **OneFs** (See below).


### SLURM cluster filesystems:
The SLURM cluster is only capable of mounting **NFS shares**, i.e., OneFS 2025 filesystem. We have made a scratch share (/nfs/scratch) for any users that do not have dedicated shares on the 2025 fileserver. Users can copy data from old shares to scratch, prior to making use of the data on the cluster. Check the [CBS SLURM wiki](https://hackmd.io/@CompCore/cbs_slurm) guide for more information.


## Managing datashare access permissions
Access to shared PI data (whether your own PI's data or collaborating with other PIs on projects) is managed by the IT Help Desk, not CBS support staff. CBS support staff do not have access to modify datashare permissions.

**To request access to datashares:**

- Submit a ticket in the Jira system: http://servicedesk.schulich.uwo.ca/
- Alternatively, contact support-cbs-server@uwo.ca and ask them to submit a ticket on your behalf
- What to include in your request:
    - Your UWO username
    - The datashare you need access to `/cifs/<pi_last_name>` or `/nfs/<pi_last_name>`
    - Brief explanation of the collaboration or project need


## Access data shares in the CBS servers
Project datashares are automatically mounted and accessible at `/cifs/<pi_last_name>` or `/nfs/<pi_last_name>` (Depending on your lab [datashare](https://hackmd.io/@CompCore/cbs_storage)). The folders are automounted, so they will not appear in directory listings until you access them directly.

> [!IMPORTANT]
> - CBS SLURM server only supports /nfs file system.

To access your PI's datashare:

1. Navigate directly to the folder, e.g. `cd /cifs/<pi_last_name>` or `cd /nfs/<pi_last_name>`
2. The folder will be automatically mounted when accessed
3. Files remain accessible as long as your authentication ticket is valid

## Local mounting (accessing storage from your computer)
You can mount your PI's datashare on your local machine when connected to Western networks (wired, uwosecure-v2 wifi, or eduroam). This guide explains how to access two datashares used in CBS infrastructure:

1. **Legacy datashare** – CIFS/SMB only
2. **OneFS 2025 datashare** – CIFS/SMB (recommended) and NFS (HPC / lab machines with Kerberos)

Replace `<pi_last_name>` and `<westernid>` with your PI folder and Western ID.


### Legacy datashare (CIFS/SMB only)

#### **Mac OS**
1. Go to `Finder → Go → Connect to Server…`
2. Enter the server address:
   ```text
   smb://rri-cbssrv.schulich.uwo.ca/<pi_last_name>_data$
   ```
3. When prompted, log in with:
   ```text
   <westernid>@uwo.ca
   ```


#### **Windows**

1. Open File Explorer → type in the address bar:

   ```text
   \\rri-cbssrv.schulich.uwo.ca\<pi_last_name>_data$ 
   ```
2. Log in with:

   ```text
   <westernid>@uwo.ca
   ```


#### **Linux**

1. Install CIFS utilities (if not already installed):

```bash
sudo apt install cifs-utils
```

2. Create a mount point:

```bash
sudo mkdir -p /cifs/<pi_last_name>
```

3. Mount the share:

```bash
sudo mount -t cifs \
  -o username=<westernid>,domain=UWO,uid=$(id -u),gid=$(id -g),vers=3.1.1 \
  //rri-cbssrv.schulich.uwo.ca/<pi_last_name>_data$ /cifs/<pi_last_name>
```

4. Access files at `/cifs/<pi_last_name>`.

   * Local user will own the mounted files.

---

### OneFS 2025 datashare

For general conecting with **CIFS/SMB is recommended**. NFS is faster and better for HPC, but requires Kerberos and pre-approved client machines.


#### **Mac OS**

CIFS/SMB (recommended)

1. Go to `Finder → Go → Connect to Server…`
2. Enter:

```text
smb://smb-cbs.schulich.uwo.ca/<pi_last_name>
```

3. Log in with:

```text
<westernid>@uwo.ca
```
#### **Windows**
CIFS/SMB (recommended)

1. Open File Explorer → type:

```text
\\smb-cbs.schulich.uwo.ca\<pi_last_name>
```

2. Log in with:

```text
<westernid>@uwo.ca
```


#### **Linux**

##### Option 1: CIFS (recommended)

1. Install CIFS utilities:

```bash
sudo apt install cifs-utils
```

2. Create a mount point:

```bash
sudo mkdir -p /cifs/<pi_last_name>
```

3. Mount the share:

```bash
sudo mount -t cifs \
  -o username=<westernid>,domain=UWO,uid=$(id -u),gid=$(id -g),vers=3.1.1 \
  //smb-cbs.schulich.uwo.ca/<pi_last_name> /cifs/<pi_last_name>
```

4. Files are accessible at `/cifs/<pi_last_name>`.


##### Option 2: NFS (HPC/lab machines with Kerberos)

> Only use if your machine is pre-registered for NFS access. Requires a valid Kerberos ticket.

1. Install NFS client (usually already installed on Linux):

```bash
sudo apt install nfs-common
```

2. Create a mount point:

```bash
sudo mkdir -p /mnt/<pi_last_name>
```

3. Authenticate with Kerberos:

```bash
kinit <westernid>@UWO.CA
klist   # verify ticket is valid
```

4. Mount the NFS share:

```bash
sudo mount -t nfs \
  -o vers=4.1,sec=krb5 \
  nfs-cbs.schulich.uwo.ca:/ifs/cbs/<pi_last_name> \
  /mnt/<pi_last_name>
```

5. Access files at `/mnt/<pi_last_name>`.



> [!Note] 
> For linux access you'll need to repeat mouting step after rebooting, or config your `/etc/fstab` accordingly.

## Off-campus access

- Access through the CBS VDI servers.
- Local mounting requires Western VPN (Pulse Secure).

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)