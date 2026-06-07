# iMODFIT v1.51 — Flexible Fitting into Cryo‑EM Maps

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ts387/imodfit/blob/claude/lucid-bell-M9Uxw/iMODFIT_Colab.ipynb)

**iMODFIT** (*internal coordinates normal MODe based FITting*) flexibly fits an
atomic structure (PDB) into a cryo‑electron‑microscopy (cryo‑EM) density map by
combining the map's gradient with **Normal Mode Analysis (NMA) in internal
coordinates** (dihedral angles). It morphs a starting model along its collective
low‑frequency vibrational modes until it matches the target density, producing a
fitted structure and an optional trajectory "movie" of the conformational change.

This repository packages the official **64‑bit Linux release**
(`iMODFIT_v1.51_Linux_20190228`) from
[Chacón Lab (IQFR‑CSIC)](http://chaconlab.org/methods/fitting/imodfit) together
with a self‑contained **Google Colab notebook** that installs the dependencies
and runs the tool end‑to‑end — no local setup required.

> The fastest way to try it: click the **Open in Colab** badge above.

---

## What's in this repo

```
.
├── iMODFIT_Colab.ipynb     # ▶ Step-by-step Colab notebook (install + run + visualize)
├── bin/                    # Pre-compiled 64-bit Linux executables
│   ├── imodfit_gcc         #   Flexible fitting  — GNU build  (recommended on Colab)
│   ├── imodfit_mkl         #   Flexible fitting  — Intel MKL build (fastest; needs MKL)
│   ├── pdb2vol_gcc         #   Simulate a density map from a PDB
│   ├── rmsd_gcc            #   Optimal structural alignment / RMSD
│   ├── pdbtool_gcc         #   Atomic-structure manipulation utility
│   └── *  (Intel "icpc" counterparts: imodfit_mkl, pdb2vol, pdbtool, rmsd)
├── imodfit_test/           # Tutorial data (GroEL conformational change)
│   ├── 1sx4A.pdb           #   Initial model  (open  GroEL conformation)
│   ├── 1oel.ccp4 / .pdb    #   Target 10 Å map + its reference structure (closed)
│   └── 1sx4A.ccp4          #   Reverse-direction target map
├── docs/
│   └── README_imodfit      # Original upstream documentation
└── README.md
```

### Which binary should I use?

| Binary        | Compiler   | Libraries | Linkage | Notes |
|---------------|------------|-----------|---------|-------|
| `*_gcc`       | GNU gcc    | LAPACK/BLAS/ARPACK/FFTW | dynamic | **Use these on Colab / generic Linux.** Only needs open‑source libs (one `apt-get` away). |
| `imodfit_mkl`, `pdb2vol`, `rmsd`, `pdbtool` | Intel icpc | Intel MKL | dynamic/static | Fastest, but require the Intel Math Kernel Library at runtime. |

The Colab notebook uses the **`_gcc`** build for zero‑configuration reproducibility.

---

## Quick start (Google Colab)

1. Open `iMODFIT_Colab.ipynb` in Colab (badge above).
2. Run the cells top to bottom. They will:
   - clone this repo (binaries + test data),
   - `apt-get install` the runtime libraries,
   - run the bundled **GroEL** fitting example,
   - report the C‑α RMSD, plot the convergence, and render the fit in 3D,
   - give you a ready‑to‑edit template to fit **your own** structure + map,
   - and a **batch mode** to fit one model into many maps at once (each result is
     named after its map, e.g. `stateA.ccp4` → `stateA_fitted.pdb`).

## Quick start (local Linux, 64‑bit)

```bash
# 1. Install runtime libraries (Debian/Ubuntu names)
sudo apt-get update
sudo apt-get install -y liblapack3 libblas3 libfftw3-single3 \
                        libarpack2t64 || sudo apt-get install -y libarpack2

# 2. Run the bundled example (open -> closed GroEL, 10 Å map)
cd imodfit_test
../bin/imodfit_gcc 1sx4A.pdb 1oel.ccp4 10 0 -t

# 3. Check accuracy against the known answer (1oel)
../bin/rmsd_gcc imodfit_fitted.pdb 1oel.pdb -c     # ~1.37 Å C-alpha RMSD
```

`imodfit_gcc <pdb> <map> <resolution_Å> <density_cutoff> [options]`

| Argument        | Meaning |
|-----------------|---------|
| `<pdb>`         | Initial atomic model to be deformed. |
| `<map>`         | Target cryo‑EM map (`.ccp4` / `.mrc` / Situs `.sit`). |
| `<resolution>`  | Map resolution in Å (must match the map). |
| `<cutoff>`      | Density threshold; `0` uses the whole map. Set to the map's recommended contour level to ignore background/noise. |
| `-t`            | Also write the trajectory `*_movie.pdb` (view in VMD/ChimeraX). |
| `-o NAME`       | Output basename (default `imodfit`). |
| `-n`, `-e`      | Number / ratio of normal modes and "excited" modes used. |
| `-m`            | Coarse‑grained model (`0`=Cα, `2`=heavy atoms, default). |

Run `bin/imodfit_gcc --help` for the full option list.

---

## How it works (in one paragraph)

iMODFIT iterates: (1) compute low‑frequency normal modes of the current model in
**dihedral (internal) coordinate** space — far fewer degrees of freedom than
Cartesian NMA, which keeps stereochemistry physical; (2) move the model a small
step along the linear combination of modes that best increases the
cross‑correlation with the target map; (3) periodically re‑diagonalize as the
structure changes. This continues until the correlation converges, yielding a
stereochemically sensible large‑scale conformational transition.

## Citing iMODFIT

> **iMODFIT: Efficient and robust flexible fitting based on vibrational analysis
> in internal coordinates** (2013). López‑Blanco JR & Chacón P.
> *Journal of Structural Biology* 184(2):261‑270.
> doi:[10.1016/j.jsb.2013.08.010](https://doi.org/10.1016/j.jsb.2013.08.010)

## License & attribution

iMODFIT is developed by the **Structural Bioinformatics Group, IQFR‑CSIC
(Madrid, Spain)** — José Ramón López‑Blanco & Pablo Chacón. The binaries here are
redistributed from the official Linux release for convenience. Please consult
[chaconlab.org](http://chaconlab.org/methods/fitting/imodfit) for terms of use,
tutorials, the Cartesian‑NMA tools, and the UCSF Chimera plug‑in.
