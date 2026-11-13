# CBS server infrastructure

The Computational Brain Science (CBS) servers have multiple components physically located at the Robarts Research Institute, designed for researchers at Western University with computational neuroscience and biomedical imaging needs.

The CBS infrastructure has three components:

- **[CBS VDI servers](https://hackmd.io/@CompCore/cbs_vdi)**: Provides a Virtual desktop infrastructure (VDI) to create virtual clones of Linux machines for user sessions. 
*Uses: Data access, visualization and light to medium computations.*
- **[CBS SLURM server](https://hackmd.io/@CompCore/cbs_slurm)**: A High-Performance Computing cluster with SLURM job scheduling capabilities. 
*Uses: GPU and heavy computational workflows.*
- **[CBS data share storages](https://hackmd.io/@CompCore/cbs_storage)**: Datashares for each lab or collaborative project. Currenlty we have two systems (See [data storage](#📦-Data-storage)).

Access to these systems depends of the account type and data storage acquired by each lab. Continue reading for further details.


## 👤 User account types
### Account types
CBS offers two account types for servers usage, each designed to meet different computational needs:

- **Standard user account:** Available for users when a lab acquire a storage location. Standard accounts have access to the **VDI CBS-Basic machines**.
- **Power User Account :** Additional charges apply (see [CBS servers fees](#💵-CBS-Servers-fees) below). Power users account have access to **VDI CBS-Heavy machines** and the **CBS SLURM** cluster.

Choose the account type that best suits your workload. You can request to change your account type  (see [account changes](#Account-changes) below) if your computational needs change over time. 

> [!Note] 
> Don't hesitate to reach out to [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca) if you need assistance determining which account type is right for your research needs.


## 📦 Data storage
CBS servers have two file server system:

- **OneFS 2025 storage (/nfs)**: Since September 2025, we have a new file server which offers improved performance and compatibility to be used with the new SLURM cluster.
- **Legacy 2018 Storage (/cifs)**: The file server before September 2025. It will continue to be available and backed up, but because of the hardaware age and limited compatibility with the SLURM cluster, storage will be charged at a lower rate (See [CBS servers Fees](#💵-CBS-servers-fees)).

See [CBS storage info](https://hackmd.io/@CompCore/cbs_storage) for additional details.

> [!IMPORTANT] 
> - CBS VDI servers are capable of mounting both Legacy and OneFS.
> - CBS SLURM server only supports OneFS.


## ❓ Requesting account / storage
### If you are user/student/postdoc  (<u>not</u> a Principal Investigator (PI))
1. All user accounts must be associated with a pre-existing PI account
2. You must use your valid uwo.ca email address when requesting an account
3. Check with your PI regarding power user accounts, as they incur additional charges
4. Fill out this form for non-PI user accounts: https://forms.office.com/r/mdmWQ6qpep

### If you are a Principal Investigator (PI)
1. You must use your uwo.ca email address when requesting an account
2. Fill out this form for PI accounts: https://forms.office.com/r/dztbCVfNxk
3. Consider requesting only a nominal amount of storage initially, as it's easier to increase storage allocations than to decrease them.
4. Review the [fee schedule](#💵-CBS-servers-fees) below before requesting resources.


### Account processing
After submitting your form, the information is forwarded to [IT Helpdesk](https://servicedesk.schulich.uwo.ca/) for account setup. You will receive an email confirmation when your account is ready. New account requests are typically fulfilled within 1 business day, though delays may occur. You will use your existing Western University credentials to log in.

### Account changes
Do you need to make changes to your existing account? Use whichever form is appropriate for your needs.
- **PI account changes:** PIs needing to increase storage, create new project spaces, or update speedcode, please use this form: https://forms.office.com/r/Syb4tKN717
- **User account changes:** Users (both PI and non-PI) wishing to request access to additional datashares, change account type (Basic or Power), update contract end date, or request access to project spaces, please use this form: https://forms.office.com/r/sQepNZDNqS

>[!Note]
>Account change requests are processed by Robarts IT and may take 1-2 business days to complete.



## 💵 CBS servers fees

The following are the **reduced rates** supported through CFI funding. Full cost recovery rates (e.g., for grant or departmental budgeting) are **2× the reduced rate**, and are shown below for reference. Billing is done on a fixed quarterly schedule.

For billing questions or custom quotes, please contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

### Basic user account fees

With the purchase of any storage, any number of standard user accounts can be created at no additional cost. Basic users have access to **CBS-basic VDI** machines.


### Power user account fees

Power users have access to **CBS-Heavy VDI** machines and the **CBS-SLURM** cluster.

| Account Type                      | Reduced Rate      | Full Cost Recovery Rate |
|----------------------------------|-------------------|--------------------------|
| First account per lab            | $1000 / year      | $2000 / year             |
| Each additional account per lab  | $500 / year       | $1000 / year             |


### Storage fees

| Storage Type         | Reduced Rate         | Full Cost Recovery Rate |
|----------------------|----------------------|--------------------------|
| Legacy 2018 Storage (/cifs)  | $40 / TB / year      | $80 / TB / year          |
| OneFS 2025 Storage (/nfs)   | $65 / TB / year      | $130 / TB / year         |

> [!Note]
> Legacy 2018 storage pricing was reduced from $50 to $40 per TB per year.

## Contact
Questions? [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)