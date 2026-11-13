# CBS software guide

The CBS VDI and SLURM servers share the same software stack. Software is installed centrally and made available through **environment modules** (Lmod), so the same tools and versions are available on both systems.

The stack is based on the containerized neuroimaging software provided by [Neurodesk](https://neurodesk.org/). Any application in the [Neurodesk application list](https://neurodesk.org/overview/applications/) can easily be added to the CBS servers: if you need one that is not installed yet, [ask us](#Requesting-software).

> [!Important]
> Do not download or install software in your home directory. Check the available modules first, and [contact us](#Requesting-software) if something is missing. See [Home directories](https://hackmd.io/@CompCore/cbs_storage#Home-directories) for why.

## Using modules

A module sets up your environment (paths, variables) so that a specific version of a program is ready to use. A few modules are loaded by default when you log in (see [below](#VDI-and-SLURM-differences)); everything else is loaded only when you ask for it.

> [!Warning] Do not unload the `apptainer` module
> The `apptainer` module is loaded by default on both the VDI and SLURM, because the software modules (Neurodesk containers) need it to run. Unloading it, for example with `module purge`, will break other modules.
>
> To remove a module, unload it by name (`module unload <name>`) instead of using `module purge`. If you do run `module purge`, load Apptainer again right after:
> ```bash
> module purge
> module load apptainer
> ```

| Task | Command |
|---|---|
| List all available modules | `module avail` |
| Search for a program and see its versions | `module spider <name>` |
| Show what a module sets up (paths, variables, container) | `module show <name>/<version>` |
| Load a module | `module load <name>/<version>` |
| Show loaded modules | `module list` |
| Unload a module | `module unload <name>` |
| Unload all modules (then reload Apptainer, see warning above) | `module purge` |

`ml` is a shortcut for `module`: for example, `ml freesurfer/8.2.0` loads a module and `ml` alone lists the loaded modules.

Example:

```bash
module spider freesurfer         # which versions are installed?
module load freesurfer/8.2.0     # load a specific version
module list                      # check what is loaded
```

> [!Tip] Always load a specific version
> `module load freesurfer` loads the default version, which can change when new versions are installed. For reproducible results, always include the version, e.g. `module load freesurfer/8.2.0`.

## VDI and SLURM differences

- **VDI:** the [commonly used software](#Commonly-used-software) below is loaded automatically when you log in, together with `apptainer` and the disk usage tools `dua` and `gdu`. You can load or unload modules on top of it.
- **SLURM:** only `apptainer` is loaded by default. Load the other modules you need **inside your job script**, rather than relying on what was loaded when you submitted the job:

  ```bash
  #!/bin/bash
  #SBATCH --time=02:00:00
  #SBATCH --cpus-per-task=4
  #SBATCH --mem=16G

  module load freesurfer/8.2.0

  recon-all ...
  ```

  See the [CBS SLURM server guide](https://hackmd.io/@CompCore/cbs_slurm) for complete job script examples.

## Commonly used software

These tools are loaded by default on the VDI. On SLURM, load the ones you need in your job script. Run `module spider <module>` to see the installed versions.

| Software | Module | Used for |
|---|---|---|
| [AFNI](https://afni.nimh.nih.gov/) | `afni` | fMRI and MRI analysis |
| [ANTs](https://github.com/ANTsX/ANTs) | `ants` | Image registration and segmentation |
| [BrainSuite](https://brainsuite.org/) | `brainsuite` | Cortical surface extraction and MRI analysis |
| [Connectome Workbench](https://www.humanconnectome.org/software/connectome-workbench) | `connectomeworkbench` | Visualization and analysis of surface and CIFTI data (HCP) |
| [dcm2niix](https://github.com/rordenlab/dcm2niix) | `dcm2niix` | DICOM to NIfTI conversion |
| [FreeSurfer](https://surfer.nmr.mgh.harvard.edu/) | `freesurfer` | Cortical reconstruction and surface-based analysis |
| [FSL](https://fsl.fmrib.ox.ac.uk/fsl/) | `fsl` | MRI, fMRI, and diffusion analysis |
| [ITK-SNAP](http://www.itksnap.org/) | `itksnap` | Image segmentation and visualization |
| [MATLAB](https://www.mathworks.com/products/matlab.html) | `matlab` | Numerical computing and scripting |
| [MRIcroGL](https://www.nitrc.org/projects/mricrogl/) | `mricrogl` | Image viewing and 3D rendering |
| [MRtrix3](https://www.mrtrix.org/) | `mrtrix` | Diffusion MRI analysis and tractography |
| [RStudio](https://posit.co/products/open-source/rstudio/) | `rstudio` | R programming environment |
| [3D Slicer](https://www.slicer.org/) | `slicer` | Medical image visualization and analysis |

Many more tools are available: run `module avail` to see them all.

## Containers

Both the VDI and SLURM support **Apptainer** (formerly Singularity) for running containerized software. The `apptainer` command is ready to use, as its module is loaded by default (see the [warning above](#Using-modules)).

### Shared container images

Commonly used container images are stored in a shared folder, available on both the VDI and SLURM:

```
/srv/containers
```

Reuse these images when possible instead of downloading your own copies: it saves time and space in your home directory and your lab's datashare. If an image is outdated or a commonly used tool is missing, [contact us](#Requesting-software) so it can be added for everyone.

### Running a container

```bash
apptainer exec /srv/containers/<image>.sif <command>
```

Your home directory and your datashares (`/nfs`, and `/cifs` on the VDI) are automatically available inside containers, so you do not need to bind them yourself.

### Container cache

When you pull or build images, Apptainer stores downloaded layers in `~/.apptainer/cache`, which counts against your 20 GB home quota. Clean it regularly:

```bash
apptainer cache clean
```

If you build or pull large images, store the resulting `.sif` files in your lab's datashare, not in your home directory.

## Requesting software

If the software you need is not available as a module or in `/srv/containers`, contact [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca). Please include:

- The name and version of the software
- A link to its website or repository
- A brief description of what you will use it for

Applications from the [Neurodesk application list](https://neurodesk.org/overview/applications/) are usually quick to add.

## Contact
Need help? Contact: [support-cbs-server@uwo.ca](mailto:support-cbs-server@uwo.ca)

Back to the [CBS servers guide](https://hackmd.io/@CompCore/cbs_servers), the main page of the CBS wiki.
