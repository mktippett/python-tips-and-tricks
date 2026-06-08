# Setting up a Pangeo notebook environment on a Mac

This guide installs the tools you need to run Pangeo-style Jupyter notebooks on a Mac. You do **not** need any prior experience with the Terminal — for each step you can copy a command, paste it, and press Enter.

---

## Before you start: three basics

**Opening the Terminal.**
Press `Command (⌘) + Spacebar` to open Spotlight search, type `Terminal`, and press `Enter`. A window with a text prompt opens. This is where every command in this guide goes.
(You can also find it at Applications → Utilities → Terminal.)

**How to run a command.**
For each gray code block below: copy the *whole* block, click in the Terminal window, paste with `Command (⌘) + V`, and press `Enter`. Then wait — when the prompt (a line ending in `$` or `%`) comes back, the command is done and you can move on.

**Which kind of Mac do you have?**
Click the Apple menu  (top-left corner) → **About This Mac**.
- If you see **Chip: Apple M1 / M2 / M3 / M4**, you have an **Apple Silicon** Mac.
- If you see **Processor: Intel**, you have an **Intel** Mac.

You'll need this in Step 1.

---

## 1. Install Miniforge

> **Already have a working `conda` or `mamba`?** Skip this entire step and go to Step 2. Re-running the installer would create a *second* conda installation and change which `conda`/`mamba` your Terminal uses by default. It would not delete your existing environments, but it causes confusion. Your existing setup can create the new environment just fine.

Miniforge is a small installer for `conda`, pre-configured to use the `conda-forge` package collection.

### If you have an Apple Silicon Mac (M1/M2/M3/M4)

```bash
cd ~/Downloads
curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
bash Miniforge3-MacOSX-arm64.sh
```

### If you have an Intel Mac

```bash
cd ~/Downloads
curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-x86_64.sh
bash Miniforge3-MacOSX-x86_64.sh
```

**The installer will ask you a few questions.** Answer them like this:

1. *"Please press ENTER to continue"* → press `Enter`.
2. The license text appears. Press the `Spacebar` a few times to scroll to the bottom, until it asks *"Do you accept the license terms?"* → type `yes` and press `Enter`.
3. *"Miniforge3 will now be installed into this location…"* → press `Enter` to accept the default location.
4. *"Do you wish to update your shell profile…?"* → type `yes` and press `Enter`.

**Now quit Terminal completely and open a new Terminal window** (this is required — the new settings only apply to a fresh window).

Check that it worked by running each line:

```bash
conda --version
mamba --version
```

Each should print a version number. If you see "command not found," make sure you opened a brand-new Terminal window after the install.

---

## 2. Create a project folder and the environment file

Copy and paste this **entire block** at once, then press `Enter`. It makes a folder called `pangeo-setup` in your home directory, moves into it, and creates the `environment.yml` file for you — so you don't need to open any text editor.

```bash
mkdir -p ~/pangeo-setup
cd ~/pangeo-setup
cat > environment.yml << 'EOF'
name: pangeo-local
channels:
  - conda-forge
  - nodefaults
dependencies:
  - python=3.12
  - pangeo-notebook=2026.01.21
  # core geoscience stack — pangeo-notebook itself only brings JupyterLab + dask;
  # these are the packages that actually do the geoscience work (see Step 8)
  - xarray
  - zarr
  - cdsapi
  - arraylake
  - netcdf4
  - h5netcdf
  - matplotlib
  - cartopy
  - cf_xarray
  - fsspec
  - gcsfs
  - s3fs
  - intake-xarray
  - nc-time-axis
  - bottleneck
  - pip
EOF
```

Confirm the file was written correctly:

```bash
cat environment.yml
```

You should see the same lines printed back.

---

## 3. Create the environment

Make sure you're still in the project folder (this is safe to run even if you already are):

```bash
cd ~/pangeo-setup
```

Then build the environment:

```bash
mamba env create -f environment.yml
```

This downloads and installs a lot of packages, so **it can take several minutes.** Lots of text will scroll by — that's normal. Wait until the prompt returns.

> **If it fails** with a message like *"nothing provides pangeo-notebook=2026.01.21"*, that exact version may not be available. Check what versions exist with `mamba search pangeo-notebook`, then either edit `environment.yml` to a version that's listed, or remove `=2026.01.21` so it just reads `- pangeo-notebook`.

Once it finishes, turn the environment on:

```bash
conda activate pangeo-local
```

Your prompt should now begin with `(pangeo-local)`. That tells you the environment is active.

---

## 4. Register the Jupyter kernel

With the environment active (prompt starts with `(pangeo-local)`), run:

```bash
python -m ipykernel install --user --name pangeo-local --display-name "Python (pangeo-local)"
```

This makes your environment selectable inside JupyterLab.

---

## 5. Start JupyterLab

```bash
jupyter lab
```

JupyterLab usually opens in your web browser automatically. If it doesn't, look in the Terminal for a line starting with `http://localhost:` or `http://127.0.0.1:` that includes a long token, and copy that full line into your browser's address bar.

Inside JupyterLab, when you open or create a notebook, choose the kernel named:

```text
Python (pangeo-local)
```

To stop JupyterLab later, go back to the Terminal window it's running in and press `Control + C`.

---

## 6. Coming back to this later

Each time you want to work again, open a new Terminal and run:

```bash
conda activate pangeo-local
jupyter lab
```

(You don't need to repeat Steps 1–4 — those are one-time setup.)

---

## 7. Installing additional packages

First make sure the environment is active:

```bash
conda activate pangeo-local
```

Then install with `mamba`. For example:

```bash
mamba install xskillscore
```

For several at once:

```bash
mamba install xesmf cf_xarray cmocean
```

`mamba` will show what it plans to install and ask you to confirm — press `Enter` (or type `y` and `Enter`) to proceed.

Use `mamba install` for packages from `conda-forge`. Use `pip install` only when a package isn't available through conda-forge.

After installing, you can check that it imported correctly:

```bash
python -c "import xskillscore; print('ok')"
```

If it prints `ok`, you're set.

---

## 8. What the environment file does (optional reading)

`pangeo-notebook` is named for the *Pangeo* project, but despite the name it is
**not** a geoscience package — it's a thin metapackage that brings in only the
JupyterLab/JupyterHub/dask-gateway layer (`jupyterlab`, `jupyterhub-singleuser`,
`dask-labextension`, `pangeo-dask` → `dask`/`distributed`, etc.). It pulls in
*zero* of `xarray`, `zarr`, `netcdf4`, `matplotlib`, or any other geoscience
library — confirmed by checking its declared dependencies directly, and true even
of the official Pangeo Docker images' own
[`base-notebook` environment](https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-notebook/environment.yml)
and [lock file](https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-notebook/conda-linux-64.lock)
(neither contains `xarray` or `zarr` either — that "official" environment is the
Jupyter/dask layer alone, same as this one). If you only installed
`pangeo-notebook`, you'd get a working JupyterLab with no science stack —
exactly the (surprising, easy-to-hit) trap this guide's `environment.yml` now
avoids by listing the geoscience packages explicitly.

- `pangeo-notebook=2026.01.21` — the JupyterLab/Hub/dask layer described above:
  what makes the environment show up and run as a notebook server, with
  multi-core/cluster compute (`dask`/`distributed`) wired in. It does *not*
  supply the geoscience stack — that's what the next group of packages is for.
- `xarray`, `zarr`, `netcdf4`, `h5netcdf` — the core data model and file formats:
  `xarray` for labeled N-D arrays, `zarr`/`netcdf4`/`h5netcdf` so it can read and
  write the array/file formats geoscience data actually comes in.
- `matplotlib`, `cartopy` — plotting, including map projections for geographic data.
- `cf_xarray`, `nc-time-axis`, `bottleneck` — small utilities `xarray` leans on:
  CF-convention-aware `.cf` accessors, plotting non-standard (e.g. `noleap`)
  calendars, and faster rolling/groupby reductions.
- `fsspec`, `gcsfs`, `s3fs`, `intake-xarray` — cloud-native data access: reading
  Zarr/NetCDF straight out of Google Cloud Storage / S3 buckets and from data
  catalogs, the way most public geoscience datasets (ERA5, CMIP6, etc.) are
  actually distributed today.
- `python=3.12` — sets the Python version.
- `nodefaults` — keeps the environment on `conda-forge` and avoids mixing in packages from the default Anaconda channel.

This list is a reasonable, broadly useful starting point — not exhaustive. Add
more with Step 7 as your own work needs them (e.g. `xesmf` for regridding,
`xskillscore` for verification metrics, `dask-jobqueue` for HPC clusters).
