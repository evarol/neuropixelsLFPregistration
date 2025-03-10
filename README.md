<image src="https://github.com/evarol/dredge/blob/main/assets/logo.png" width="200px"></image>

Published in *Nature Methods*: https://www.nature.com/articles/s41592-025-02614-5

Free PDF link: https://rdcu.be/ecqZf

# DREDge: Decentralized Registration of Electrophysiology Data

DREDge is an algorithm for estimating the relative motion of a high-density microelectode array
(e.g., Neuropixels) relative to the brain tissue. DREDge can estimate this motion both from local
field potentials and from spike data, and the motion estimate can be rigid (i.e., the motion does
not change along the probe) or nonrigid (different channels on the probe may undergo different
motion).

> :point_right: We now recommend using [SpikeInterface][spikeinterface] to run DREDge (both the AP and LFP methods) and integrate it with downstream interpolation and spike sorting pipelines.
> SpikeInterface's [`correct_motion()`](https://spikeinterface.readthedocs.io/en/latest/api.html#spikeinterface.preprocessing.correct_motion) function implements DREDge under the presets
> `"dredge"`, `"dredge_fast"`, as well as older versions of DREDge under the name `"decentralized"` (the presets `"nonrigid_accurate"`, `"nonrigid_fast_and_accurate"`, and `"rigid_fast"`).
> For information, see the tutorial pages:
>  - Handle motion/drift with spikeinterface: https://spikeinterface.readthedocs.io/en/latest/how_to/handle_drift.html
>  - Estimate drift using the LFP traces: https://spikeinterface.readthedocs.io/en/latest/how_to/drift_with_lfp.html
> 
> and the module documentation:
>    - Motion/drift correction module summary https://spikeinterface.readthedocs.io/en/latest/modules/motion_correction.html
>    - API reference: https://spikeinterface.readthedocs.io/en/latest/api.html#module-spikeinterface.sortingcomponents.motion

This repo contains our original implementation of DREDge as well as some extra utility functions. For most users, SpikeInterface would make more sense, but this code is still maintained
and you can feel free to open issues here with questions.

Check out our preprint for more details and experiments! https://www.biorxiv.org/content/10.1101/2023.10.24.563768v1

### Questions? Issues?

We want to make this tool as easy to use as it can be. Feel free to open an issue here or email <a href="mailto:ciw2107@columbia.edu">ciw2107@columbia.edu</a> if you have questions or get stuck.

## Python instructions

### Introduction

This library contains Python code for motion estimation and correction for high density LFP and spiking data. Demo notebooks are available to see how to use this code on your data:
 - [Spike registration](notebook/ap_registration.ipynb)
 - [LFP-based registration and interpolation](notebook/lfp_registration_and_interpolation_demo.ipynb)
 - [Experimental demo of chronic multi-session registration](notebook/ap_registration.ipynb)

### Requirements

Since DREDge is written in Python and PyTorch, it runs on Python versions after 3.8 and any system supported by PyTorch, including most recent Linux, Mac, and Windows versions. PyTorch supports both CPU and GPU-based workflows, and we suggest using the GPU version if you have a compatible graphics card (e.g., NVIDIA) since this can be much faster. See https://pytorch.org for supported operating systems. DREDge has been tested on Linux (Ubuntu 20.04) and recent MacOS; collaborators have tested it on Windows as well.

The suggested setup below should take 5-10 minutes on a typical computer and internet connection when installing everything including Python from scratch in a new `mamba` environment.


### Installation with Conda (or Mamba) (recommended)

**Setting up your conda environment**

If you are new to conda, we recommend using [mamba][mamba]'s Mambaforge distribution to install Python and DREDge. These tools manage GPU runtime dependencies in addition to python dependencies in an easy way, so that you won't have to deal with installing the CUDA runtime yourself. Install instructions are at that link -- look for an installation script under the heading Mambaforge for your platform.

Conda/mamba lets us have multiple Python environments on our system so that package versions don't conflict across your different projects. Here's how to set up an example environment that will support everything DREDge needs. If you used `conda` instead of `mamba`, replace `mamba` with `conda` in the instructions below.

```
# make the environment (feel free to use another name or install into your environment of choice)
# and, of course, feel free to add other packages that you like to use here!
# note, I'm adding some of the spikeinterface dependencies here so that mamba manages them
$ mamba create -n dredge python=3.11 numpy scipy h5py tqdm jupyterlab seaborn numba scikit-learn

# activate this environment!
$ mamba activate dredge
# now, you should see (dredge) in your terminal prompt
```

*Get our code*

```
(dredge) $ git clone git@github.com:evarol/dredge.git # (or https://github.com/evarol/dredge.git)
(dredge) $ cd dredge /
```

*Install PyTorch*

Go to pytorch.org and follow the installation instructions there under the Conda tab for your platform.
*[!!!]* if you are using `mamba` rather than `conda`, be sure to replace `conda` with `mamba` in the
commands given by pytorch.org's instructions!

**Install pip dependencies and DREDge**

Note, this step installs `spikeinterface`.

```
# make sure we're in the right folder:
(dredge) $ pwd
# => .../dredge/
(dredge) $ pip install -r requirements.txt
(dredge) $ pip install .
```

### Installation with `pip`

If you prefer to avoid conda/mamba and you already have Python installed, you can clone the repo and run the last two commands above to install DREDge. We'd recommend first following the install instructions at https://pytorch.org under the `pip` heading to install a GPU version of PyTorch. Since Jupyter is not listed in `requirements.txt`, you can also `pip install jupyterlab` if you'd like to view our demo notebooks.

## Trying out the demos

After installing as above, you can open Jupyter Lab and checkout the demo notebooks:

```
# make sure we're in the right folder:
(reglib) $ pwd
# => .../dredge/
(reglib) $ jupyter lab
```

This should open your browser. Use the file explorer on the left to navigate to `notebook/ap_registration.ipynb` 

## Related information

We're also porting an updated AP version and the LFP version of this code into [spikeinterface][spikeinterface]. We'll update this README accordingly, as SpikeInterface will eventually be the recommended way to use DREDge for most users.

## Code structure in `python/dredge/`

 - `dredgelib.py`
   - Library code (cross-correlations and linear algebra to solve batch and online centralization problems)
 - `motion_util.py`
   - `MotionEstimate` objects. The function `get_motion_estimate` takes the matrix of displacement vectors and the corresponding time and spatial bin edges or centers, and returns a `RigidMotionEstimate` or `NonrigidMotionEstimate`. These classes implement `disp_at_s()` and `correct_s` which take in times and depths and return the corresponding estimated displacements or motion-corrected positions
   - `spike_raster`, which takes spike times, depths, and amplitudes, and returns a 2d raster map
   - Binning and window helpers
   - Plotting helpers (e.g., `show_spike_raster`, `plot_me_traces`, ...)
 - `dredge_ap`: `register()` function registers spike data
 - `dredge_lfp`: `register_online_lfp()` function implements online algorithm for LFP data


[mamba]: https://github.com/conda-forge/miniforge#mambaforge
[spikeinterface]: https://github.com/SpikeInterface/spikeinterface

### Matlab instructions

<image src="https://github.com/evarol/dredge/blob/main/assets/image.png" width="400px"></image>

To run
1. Download demo files from dropbox link (too big for github) : 
    a. https://www.dropbox.com/s/jfr450rpjf6bew5/Pt03.h5?dl=0
    b. https://www.dropbox.com/s/uf5j75jqkyisesh/Pt02_2.h5?dl=0
2. Put file in same directory as all this code.
3. Run main.m

Example results from 2 human neuropixels recordings from Paulk et al. (https://www.biorxiv.org/content/10.1101/2021.06.20.449152v2)


![Dataset1](https://github.com/evarol/dredge/blob/main/assets/pt03_results.png)

![Dataset2](https://github.com/evarol/dredge/blob/main/assets/pt02_results.png)

