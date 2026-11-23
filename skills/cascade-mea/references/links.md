# CASCADE links and references

All links checked on 2026-11-12 (manufacturer links 2026-11-16) (journal and Zenodo pages block automated checks, so their
DOIs were checked through the doi.org resolver).

## CASCADE itself

| What | Link |
|---|---|
| GitHub repository (code, README, CHANGELOG) | https://github.com/FA387/cascade-mea |
| Releases (download a specific version) | https://github.com/FA387/cascade-mea/releases |
| Report a bug / ask a question | https://github.com/FA387/cascade-mea/issues |
| PyPI package (`pip install cascade-mea`) | https://pypi.org/project/cascade-mea/ — v0.9.6 since 2026-11-20 |
| conda package (`conda install -c fa387 -c conda-forge cascade-mea`) | https://anaconda.org/fa387/cascade-mea — v0.9.6 |
| Published article | https://doi.org/10.1093/bioinformatics/btag570 (journal page: https://academic.oup.com/bioinformatics/article/42/8/btag570/8746882) |
| Sample recordings + v0.8.0 expected output (Zenodo) | https://doi.org/10.5281/zenodo.19161615 |
| Source code archive cited in the article (Zenodo, v0.8.0) | https://doi.org/10.5281/zenodo.20840795 |
| Source code, all versions (Zenodo concept DOI; new versions are made automatically from GitHub releases) | https://doi.org/10.5281/zenodo.20840794 |
| Print the citation | `cascade --citation` (text + BibTeX) |

**Cite as:** Avila F, Kim J, Park J-C, Kang R, Lee H, Park S-H. CASCADE: criticality avalanche
spike cross-platform analysis detection engine, a multi-manufacturer MEA bash analysis
pipeline. *Bioinformatics*. 2026;42(8):btag570. https://doi.org/10.1093/bioinformatics/btag570
— and state the CASCADE version used (the article describes v0.8.0).

## Manufacturer documentation (file formats and APIs)

| Device | Link |
|---|---|
| 3Brain (BXR/BRW format PDFs; bwpy reader) | https://bwpy.readthedocs.io/en/latest/ |
| Axion (official file loader) | https://github.com/axionbio/AxionFileLoader |
| Cortical Labs (docs, SDK) | https://docs.corticallabs.com/ , https://github.com/Cortical-Labs/cl-sdk |
| FinalSpark (Neuroplatform docs, utilities) | https://finalspark-np.github.io/np-docs/welcome.html , https://github.com/FinalSpark-np/np-utils |
| MaxWell (MaxLab Python API; no public HDF5 layout spec) | https://api-docs.mxwbio.com/ |
| MCS (HDF5 definition; McsPyDataTools) | https://www.multichannelsystems.com/sites/multichannelsystems.com/files/documents/manuals/HDF5%20MCS%20Raw%20Data%20Definition.pdf , https://mcspydatatools.readthedocs.io/en/latest/ |

## Installing the tools CASCADE needs

| What | Link |
|---|---|
| Miniforge (conda) — recommended | https://github.com/conda-forge/miniforge |
| conda getting started | https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html |
| micromamba (lighter alternative) | https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html |
| Python (CASCADE needs ≥ 3.10; conda installs it for you) | https://www.python.org/downloads/ |
| Git (optional, for `git clone` / updates) | https://git-scm.com/downloads |
| WSL — Linux inside Windows (optional) | https://learn.microsoft.com/en-us/windows/wsl/install |
| VS Code — editor for README/scripts (optional) | https://code.visualstudio.com/ |

## Reference implementations CASCADE was validated against

| What | Link |
|---|---|
| `mrestimator` — Wilting & Priesemann MR branching-ratio toolbox (CASCADE BR matches it to 6 decimals) | https://github.com/Priesemann-Group/mrestimator |
| PySpike — Kreuz SPIKE-distance (optional extra: `pip install ".[synchrony]"`) | https://github.com/mariomulansky/PySpike |

## Method references (cite the ones whose metrics you report)

| Metric | Reference |
|---|---|
| Neuronal avalanches | Beggs & Plenz (2003) *J Neurosci* 23:11167. https://doi.org/10.1523/JNEUROSCI.23-35-11167.2003 |
| Avalanche pipeline, DCC, `--perc`, `--pl_exclude` | Habibollahi et al. (2023) *Nat Commun* 14:5287. https://doi.org/10.1038/s41467-023-41020-3 |
| Power-law fitting (normalisation, xmin by KS) | Clauset, Shalizi & Newman (2009) *SIAM Rev* 51:661. https://doi.org/10.1137/070710111 |
| NCC toolbox (power laws, shape collapse) | Marshall et al. (2016) *Front Physiol* 7:250. https://doi.org/10.3389/fphys.2016.00250 |
| Branching ratio (MR estimator) | Wilting & Priesemann (2018) *Nat Commun* 9:2325. https://doi.org/10.1038/s41467-018-04725-4 |
| Shape collapse | Friedman et al. (2012) *Phys Rev Lett* 108:208102. https://doi.org/10.1103/PhysRevLett.108.208102 |
| SPIKE-distance | Kreuz et al. (2013) *J Neurophysiol* 109:1457. https://doi.org/10.1152/jn.00873.2012 |
| Synchrony Index | Paiva, Park & Príncipe (2010) *Neural Comput Appl* 19:405. https://doi.org/10.1007/s00521-009-0307-6 |
| Avalanche size with `--perc` (threshold contribution subtracted) | Poil et al. (2012) *J Neurosci* 32:9817. https://doi.org/10.1523/JNEUROSCI.5990-11.2012 |

The README "References" section is the source of these; if they disagree, the README wins.
