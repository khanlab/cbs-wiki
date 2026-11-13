# CBS servers guide

The Computational Brain Science (CBS) servers, located at the Robarts Research Institute, provide computing and storage for Western University researchers working in computational neuroscience and biomedical imaging. They have three components:

- **[CBS VDI servers](https://hackmd.io/@CompCore/cbs_vdi)**: Linux virtual desktops for data access, visualization, and light to medium computations.
- **[CBS SLURM server](https://hackmd.io/@CompCore/cbs_slurm)**: a high-performance computing cluster for GPU and heavy computational workflows.
- **[CBS storage](https://hackmd.io/@CompCore/cbs_storage)**: datashares for each lab and for collaborative projects (see [Data storage](#Data-storage)).

Which of these you can use depends on your account type and on the storage your lab has. 

Other guides:

- **[CBS software guide](https://hackmd.io/@CompCore/cbs_software)**: software modules and containers on the VDI and SLURM servers.
- **[CBS Python guide](https://hackmd.io/@CompCore/python_cbs)**: managing Python environments (Pixi, uv, Conda).


## User account types

| Account type | Access | Cost |
|---|---|---|
| **Basic user account** | CBS-Basic VDI machines | Free with any lab storage |
| **Power user account** | CBS-Basic and CBS-Heavy VDI machines, and the CBS SLURM cluster | See [fees](#CBS-servers-fees) |

You can change your account type later if your needs change (see [Account changes](#Account-changes)). If you are not sure which account type you need, contact [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca).


## Data storage
CBS has two file servers: **OneFS 2025** (`/nfs`), available on both the VDI and SLURM, and **Legacy 2018** (`/cifs`), available on the VDI only and charged at a lower rate. See the [CBS storage guide](https://hackmd.io/@CompCore/cbs_storage) for details.


## Requesting accounts and storage
Every user account belongs to a Principal Investigator (PI), so the PI's account must exist before lab members can request theirs. All requests must use a valid uwo.ca email address.

### New accounts

| Who | Form | Before you apply |
|---|---|---|
| **PI** (includes the lab's initial datashare) | [PI account form](https://forms.office.com/r/dztbCVfNxk) | Review the [fees](#CBS-servers-fees). Start with a modest amount of storage: it is easier to increase an allocation than to decrease it. |
| **Lab member** (student, postdoc, staff) | [User account form](https://forms.office.com/r/mdmWQ6qpep) | Your PI must already have an account. Check with your PI before requesting a Power user account, as it incurs additional charges. |

### Account changes

| I want to... | Who | Form |
|---|---|---|
| Increase storage, request a new datashare (e.g., on OneFS 2025), create a new project space, or update the speedcode | PI | [PI account changes form](https://forms.office.com/r/Syb4tKN717) |
| Get access to another datashare or project space, change account type (Basic or Power), or update my contract end date | Any user (PI or lab member) | [User account changes form](https://forms.office.com/r/sQepNZDNqS) |

Requests are forwarded to the [IT Helpdesk](https://servicedesk.schulich.uwo.ca/), who set up or update your account. Both new accounts and account changes are typically completed within 1–2 business days.

You will receive an email confirmation when your request is complete. Log in with your existing Western University credentials.


## CBS servers fees

The rates below are **reduced rates** supported through CFI funding. Full cost recovery rates (e.g., for grant or departmental budgeting) are **2× the reduced rate** and are shown for reference. Billing is done on a fixed quarterly schedule. For billing questions or custom quotes, contact [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca).

### Basic user account fees

With the purchase of any storage, any number of Basic user accounts can be created at no additional cost.

### Power user account fees

| Account | Reduced rate | Full cost recovery rate |
|---|---|---|
| First account per lab | $1000 / year | $2000 / year |
| Each additional account per lab | $500 / year | $1000 / year |

### Storage fees

| Storage | Reduced rate | Full cost recovery rate |
|---|---|---|
| Legacy 2018 (`/cifs`) | $40 / TB / year | $80 / TB / year |
| OneFS 2025 (`/nfs`) | $65 / TB / year | $130 / TB / year |

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)
