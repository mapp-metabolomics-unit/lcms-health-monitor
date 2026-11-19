# LC–MS health monitor: roadmap

## Overall goal

Write a software to monitor the health state of LCMS systems. In this precise case: give the MAPP platform a quick view of the health of its HF and HFX LC–MS systems. Use long-term QCs spiked with standards and blanks to track performance across injections; show instrument warnings and errors separately from measured performance. This is a project guide, not a specification for alert thresholds or a chosen software stack.

## To be resolved 

@Colin

- [x] return taxon name associated to each QC
- [x] return sample_id associated to each QC
- [ ] return identity of each spiked standard (Pubchem id, link) and concentration
- [ ] create a batch_id and associate it to QC and blanks

- **dbgi_001031** is _Vachellia karroo_ https://emi-collection.unifr.ch/directus/admin/content/Field_Data/66810
- **dbgi_001032** is _Sequoiadendron giganteum_ https://emi-collection.unifr.ch/directus/admin/content/Field_Data/66816

## Functions

List of functions to be implemented. Draft of first discussion (PMA, Marco, Colin)

- Allows for rapid assesment of the health of the various LC-MS systems of the MAPP platform.
    - LC
        - LC status: pings the user in case an error or warning appears on the LC system
        - LC performance:
            - for each standards + 1 feature of each QC (used to improve coverage of rt range)
                - RT
                - peak shape
                - number of data point
                - tailing factor
                - FWHM
            - extract pressure from the thermo RAW files
                - if so, monitor pressure of QC + blanks
            - track id of columns. How do we do for Thermo Systems ?

            

    - MS
        - MS status: pings the user in case an error or warning appears on the MS system
        - MS performance
            - for each standards + 1 feature of each QC (used to improve coverage of mass range)
                - m/z 
                - ppm
                - intensity
                - area
                - msms
                    - intensity of max peak
                    - number of fragments
            - for each QC
                - number of annotated features

    - Dashboard: a GUI centralizing the plots and monitoring functions
        - Select a LCMS system (here HF or HFX)
        - View dynamic plots for all dimensions mentionned above (in y axis) versus injection date (x axis)
        - Viewer for individual chromatograms + associated spectra

## Start here

1. Read the current document. Meet the team and discuss the goal and functions with them. Reformulate in your own words the proposal of the project in the section below.
2. Torough bibliography overview of existing solution.
3. Manually treat a file from A-Z (spotting it in the network drive, open and treat in mzmine)
4. Draft an overall schema

## Proposal
The goal is to build a software tool that gives the MAPP platform a way to quickly assess the operational health of its two LC-MS instruments (HF and HFX).

The approach rests on **longitudinal quality-control (QC) injections** - standards and blanks run on a recurring schedule - so that every injection produces a set of performance metrics that can be tracked over time.

**Measured performance (QC-derived metrics):** for each standard and a representative feature per QC (chosen to span the full retention-time and mass range), the following are extracted:
  - *LC side:* retention time, tailing factor, FWHM, number of data points, and - where available from Thermo RAW files - column pressure and column identity tracking.
  - *MS side:* m/z accuracy (ppm), signal intensity/area, MS/MS quality (intensity of the most abundant fragment, number of fragments), and the count of annotated features per QC.

**Instrument status:** errors and warnings from the LC or MS hardware are flagged and (ideally) push a notification to designated users, independently of analytical performance. Currently unclear if these signals are accessible directly from the instruments without proprietary vendor software, so this is treated as a stretch goal pending feasibility evaluation.

Ideally, all of this would be packed into a **dashboard** where the user selects an instrument and sees dynamic plots of metrics plotted against injection date, plus a viewer for individual chromatograms and their associated spectra. The goal is a single interface that answers "*is this instrument performing normally?*" at a glance.

**Implementation notes**:
- **Database:** SQL database to store metadata and mzML files.
- **Front-end / GUI:** Still open. The tool is a locally installed application, not a hosted web app. Options include a native desktop framework (Rust with Tauri, Python with PyQt) or a web-based UI (React + D3.js inside Tauri, or Shiny bound to localhost).
- **Existing ecosystem:** Existing toolkits could be adapted or expanded rather than building from scratch.

## Existing tools

### R

- [MSstatsQC](https://github.com/eralpdogu/MSstatsQC): already implements longitudinal QC monitoring for LC-MS (control charts, change-point detection, supervised ML classification of run quality); designed primarily for targeted proteomics but the statistical framework is general.
- [QC4metabolomics](https://github.com/stanstrup/QC4Metabolomics): tracks m/z + RT pairs for user-defined compounds per instrument, extracts ppm/RT/intensity/peak-shape, timeline dashboard with Shiny + ggplot2/plotly, MariaDB. However, its Docker-based deployment is resource-heavy (~19 GB disk, 7–8 GB RAM), so it is more of a reference than a starting point.
- [RforMassSpectrometry](https://github.com/rformassspectrometry/RforMassSpectrometry): a broader suite of packages covering raw-data handling (Spectra), feature detection (xcms), quantification, and annotation; could supply the data-processing layer that feeds metrics into the monitoring dashboard?

| Feature | MSstatsQC | QC4metabolomics | RforMassSpectrometry |
|---|:---:|:---:|:---:|
| **LC: Retention Time** | &check; (core metric) | &check; (RT deviation) | &check; (xcms extracts RT) |
| **LC: Peak shape (tailing/asymmetry)** | &check; (Peak Asymmetry) | &check; (Tailing + Asymmetry) | ~ (derivable from peak data, not a QC metric) |
| **LC: FWHM** | &check; (MaxFWHM) | ~ (peak width via xcms, not an explicit QC metric) | ~ (peak width in scans available in xcms) |
| **LC: Number of data points** | &cross; | &cross; | ~ (peak width in scans, no direct QC metric) |
| **LC: Column pressure (Thermo RAW)** | &cross; | &cross; | &cross; |
| **LC: Column ID tracking** | &cross; | &cross; | &cross; |
| **MS: m/z accuracy (ppm)** | &check; (MassAccuracy) | &check; (mz_dev_ppm) | ~ (m/z extracted, ppm requires reference mass) |
| **MS: Signal intensity / area** | &check; (Total Peak Area) | &check; (Apex intensity) | &check; (peak area / intensity via xcms) |
| **MS/MS: Intensity of max fragment** | &cross; | &cross; | ~ (MS2 spectra accessible via Spectra, no QC extraction) |
| **MS/MS: Number of fragments** | &cross; | &cross; | ~ (MS2 peak count via Spectra, no QC extraction) |
| **Annotated features per QC** | &cross; | &cross; | ~ (feature count via xcms, no QC-specific counting) |
| **Dashboard: Longitudinal plots (metric vs. date)** | &check; (Control Charts: XmR, CUSUM) | &check; (time-line plots, ggplot2/plotly) | &cross; |
| **Dashboard: Chromatogram viewer** | &cross; | ~ (EICs for contaminants) | ~ (Chromatograms package, msbrowser) |
| **Dashboard: Spectrum viewer** | &cross; | &cross; | ~ (Spectra base facilities) |
| **Dashboard: Instrument selection** | ~ (per peptide, not per instrument) | &check; (instrument-specific compound lists) | &cross; |
| **SQL database** | &cross; (CSV-based) | &check; (MariaDB) | &check; (MsBackendSql / MsBackendOfflineSql) |
| **Notifications / alerts** | &cross; | &check; (email warnings) | &cross; |
| **Hardware error / warning monitoring** | &cross; | &cross; | &cross; |
| **Local (non-hosted) installation** | &check; (R + Shiny) | ~ (Docker, local but resource-heavy) | &check; (R packages) |

**Legend:** &check; = implemented, ~ = partially available / derivable but not as a QC feature, &cross; = not present

--- 

**Key observations:**

- **MSstatsQC** best covers the statistical monitoring layer (control charts, change-point detection, ML classification) and the most critical LC/MS metrics (RT, FWHM, peak asymmetry, area, ppm), but provides **no** raw data extraction, no chromatogram/spectrum viewer, no MS/MS QC metrics, and no database.

- **QC4metabolomics** is the only tool with a **fully integrated pipeline** (raw data &rightarrow; xcms &rightarrow; metrics &rightarrow; MariaDB &rightarrow; Shiny dashboard &rightarrow; alerts) and instrument selection. However, it lacks MS/MS QC metrics, FWHM as an explicit metric, a chromatogram viewer, and hardware monitoring.

- **RforMassSpectrometry** is not directly a monitoring/analysis tool but the **processing layer**: Spectra (data import, including vendor formats via extensions), xcms (peak detection, alignment, feature extraction), Chromatograms (visualisation). It provides the building blocks from which metrics can be extracted, **including SQL-backed storage via `MsBackendSql`**, but has no dashboard and no QC logic of its own.

None of the three tools implements column pressure extraction from Thermo RAW files, column ID tracking, or hardware error/warning monitoring — these features would need to be developed from scratch in any case.


### Python

- [Rapid QC-MS](https://github.com/czbiohub-sf/Rapid-QC-MS): local Python app (CZ Biohub San Francisco, *Anal. Chem.* 2024); tracks retention time, m/z, and intensity per injection; Plotly Dash dashboard with real-time updates; Slack/email alerts on QC failure; designed for Thermo Orbitrap workflows. Gaps: no MS/MS fragment metrics, no column pressure/ID extraction, no annotated feature count. Note: **is Windows-first** (requires MSConvert + MS-DIAL as external dependencies); dashboard is browser-based (Plotly Dash), not a native GUI. Possibly unmaintained (last update July 2024). **Most promising as a starting point**.
- [π-MetaboQC](https://github.com/PHOENIXcenter/pi-metaboqc): feature-intensity matrix QC and preprocessing pipeline (missingness triage, blank/QC filtering, batch correction, normalization, report generation). More of a data-curation tool than an instrument dashboard; relevant for the "annotated feature count" side?
- [MaSpeQC](https://github.com/MonashProteomics/MaSpeQC): **closest in scope**; interactive LC-MS/MS instrumentation QC (*J. Proteome Res.* 2026); tracks m/z (ppm + mDa), RT, FWHM, and normalised area for up to 15 user-defined components per QC; extracts detailed pressure profiles from Thermo RAW files (requires vendor-provided non-redistributable RawFileReader libraries); email alerts on threshold breach; MS/MS metrics limited to total MS2 spectra count, peptide spectrum matches, and unique peptides (proteomics-oriented). Gaps: no fragment-level MS/MS metrics (fragment count, max fragment intensity), no column ID tracking, no annotated feature count. Heavier multi-service web stack (Node.js + MySQL + Python + mzMine2 + Morpheus).
- [QCScreen](https://sourceforge.net/projects/qcscreen/): conceptually close (m/z, RT, mass accuracy, feature area across files; Qt GUI + SQLite) but very dated (Python 2.6, Windows XP/7 only, unmaintained since 2015).
- [AutoQ4MS](https://github.com/Landeswasserversorgung/AutoQ4MS) - checked and rejected: requires a MATLAB license.

| Feature | Rapid QC-MS | π-MetaboQC | MaSpeQC | QCScreen | PeakQC |
|---|:---:|:---:|:---:|:---:|:---:|
| **LC: Retention Time** | ✓ (RT shift from library + in-run avg) | ✗ | ✓ (per component) | ✓ (RT + deviation) | ✓ (RT error per tracked ion) |
| **LC: Peak shape (tailing/asymmetry)** | ✗ | ✗ | ✓ | ✗ | ✗ |
| **LC: FWHM** | ✗ | ✗ | ✓ | ✗ | ✗ |
| **LC: Number of data points** | ✗ | ✗ | ✗ | ✗ | ✗ |
| **LC: Column pressure (Thermo RAW)** | ✗ | ✗ | ✓ (max, start, end; requires RawFileReader DLLs) | ✗ | ✗ |
| **LC: Column ID tracking** | ✗ | ✗ | ✗ | ✗ | ✗ |
| **MS: m/z accuracy (ppm)** | ✓ (Δm/z vs. library) | ✗ | ✓ (ppm + mDa) | ✓ (m/z + mass accuracy ± ppm) | ✓ (mass error ppm) |
| **MS: Signal intensity / area** | ✓ (intensity across specimens) | ~ (intensity matrix QC, not per-injection monitoring) | ✓ (normalised area) | ✓ (feature area + relative bias) | ~ (XIC intensity + TIC per tracked ion) |
| **MS/MS: Intensity of max fragment** | ✗ | ✗ | ✗ | ✗ | ✗ |
| **MS/MS: Number of fragments** | ✗ | ✗ | ~ (total MS2 count, PSMs, unique peptides — proteomics only) | ✗ | ~ (MS2 spectrum count + TIC only) |
| **Annotated features per QC** | ✗ | ~ (retained feature count per stage) | ✗ | ✗ | ✗ |
| **Dashboard: Longitudinal plots (metric vs. date)** | ✓ (Plotly Dash, real-time) | ✗ (batch reports, not time-series monitoring) | ✓ (interactive web charts) | ✓ (per-feature across files) | ~ (per-batch diagnostic figures/heatmaps; static output, no persistent time-series GUI) |
| **Dashboard: Chromatogram viewer** | ~ (targeted feature m/z vs. RT) | ✗ | ✗ | ✓ (EIC per feature) | ~ (via Mirador, separate tool in same suite: XIC, XIM heatmaps) |
| **Dashboard: Spectrum viewer** | ✗ | ✗ | ✗ | ✗ | ~ (via Mirador: MS/MS mirror plots) |
| **Dashboard: Instrument selection** | ~ (single-instrument setup) | ✗ | ✓ (multi-instrument config) | ✗ | ✗ (processes a file batch, no instrument selector) |
| **SQL database** | ✓ (SQLite) | ✗ | ✓ (MySQL) | ✓ (SQLite) | ✗ (CSV + static figures) |
| **Notifications / alerts** | ✓ (Slack + email) | ✗ | ✓ (email on threshold breach) | ✗ | ✗ (mentioned as future work in paper) |
| **Hardware error / warning monitoring** | ✗ | ✗ | ✗ | ✗ | ✗ |
| **Local (non-hosted) installation** | ✓ (local app; needs MSConvert + MS-DIAL) | ✓ (Python package, CLI/notebook) | ~ (local but heavy: Node.js + MySQL + Python + mzMine2 + Morpheus) | ~ (Windows XP/7 only, Python 2.6, unmaintained) | ✓ (self-contained .exe, no installation, no external dependencies) |

**Legend:** ✓ = implemented, ~ = partially available / derivable but not as a QC feature, ✗ = not present

---

**Key observations:**

- **Rapid QC-MS** is the most complete "ready-to-use" starting point: real-time Plotly Dash dashboard, SQLite, Slack/email alerts, and tracks RT, Δm/z, and intensity per internal standard. However, it is **targeted-internal-standard focused** (no untargeted feature counting, no peak-shape metrics, no FWHM, no MS/MS fragment metrics, no pressure). It also depends on MS-DIAL as an external processing engine and is Windows-first. Last update ~July 2024; open issues and pull requests remain unresolved — maintenance status uncertain.

- **MaSpeQC** is the **closest in overall scope**: multi-instrument, and covers all three peak-shape metrics (FWHM, Tailing, Asymmetry), plus pressure profiles from Thermo RAW (max, start, end; main/loading/gradient pump), mass accuracy (ppm + mDa), normalised area and height, and RT per component. It also provides MySQL storage, email alerts on threshold breach, and an interactive web dashboard. Its main gaps relative to the proposal are: no fragment-level MS/MS metrics (only proteomics-oriented MS2 counts), no column ID tracking, no annotated feature count, no chromatogram/spectrum viewer, and a heavy multi-service stack (Node.js + MySQL + mzMine2 + Morpheus).

- **π-MetaboQC** operates at a **different level**: it is a post-acquisition data-curation pipeline (feature-intensity matrix &rightarrow; filtering &rightarrow; batch correction &rightarrow; normalization &rightarrow; report). It does not extract per-injection metrics from raw files, has no dashboard for longitudinal monitoring, and no alerting. Its only overlap with the proposal is tracking retained feature counts, which could inform the "annotated features per QC" metric.

- **QCScreen** is conceptually the simplest (Qt GUI + SQLite + per-feature RT/m/z/area plots) but is **unmaintained since 2015**, requires Python 2.6, and targets Windows XP/7. Not viable as a starting point, but its architecture (SQLite + per-feature QC charts) could be a useful design reference.
- **PeakQC** (PNNL, *JASMS* 2024) is a **self-contained desktop executable** (tkinter GUI, no installation, no external dependencies beyond bundled MZA, Python and Python packages) that reads Thermo `.raw`, Agilent `.d`, Bruker `.d`, and `mzML` directly. It performs omics-agnostic automated QC: PCA across runs, auto-tracking of the most abundant/frequent ions (or user-specified targets), per-ion metric extraction (RT error, mass error ppm, XIC intensity, TIC, spectrum count), Isolation Forest outlier detection, and error heatmaps. Its main strengths are **zero-dependency raw-data access** (MZA-based, no MS-DIAL/MSConvert) and **batch-level outlier/defect detection**. Its main gaps relative to the proposal are: no peak-shape metrics, no pressure, no persistent longitudinal dashboard (output is static CSV + figures per batch), no database, no alerting (explicitly listed as future work in the paper [PMID: 39013167](https://pmc.ncbi.nlm.nih.gov/articles/PMC11932329/)), no instrument selection, and no annotated feature count. The companion tool **Mirador** (same suite) provides an interactive XIC/XIM/MS/MS mirror-plot viewer, but it is a separate application, not integrated into the QC workflow. Same for TandemMatch (MS/MS spectral library matching), PeakQuant (targeted MS1 abundance extraction). The GitHub repo (`pnnl/IonToolPack`) shows activity as of February 2026.

- **AutoQ4MS** excluded: requires a MATLAB license (Functionally it would be relevant — PostgreSQL, Grafana dashboard, MSPC; PCA/Hotelling's T2, multi-instrument, email alerts — but the license requirement makes it impractical).

**No tool implements column ID tracking, fragment-level MS/MS QC metrics (fragment count, max fragment intensity), a general chromatogram + spectrum viewer integrated into the QC workflow, or hardware error/warning monitoring.** These remain new features regardless of which codebase is chosen.

**Practical implication:** No single tool covers the full proposal. The most viable strategy appears to be a **hybrid**: adopt PeakQC's MZA-based raw-data ingestion (zero external converter dependency) and outlier-detection logic as the processing backbone, combine with Rapid QC-MS's dashboard/alerting architecture (Plotly Dash + SQLite + notifications) as the presentation layer, and build the missing pieces (peak-shape metrics, pressure extraction, column ID tracking, fragment-level MS/MS metrics, integrated viewer) as new modules.

## Resources

### SQL & MS

- https://github.com/idsm-src/pgms
- https://github.com/wkumler/mzsql
- https://github.com/rformassspectrometry/MsBackendSql
- https://github.com/oolonek/mzmlwatcher

### Varia 

- https://github.com/Rappsilber-Laboratory/MetaXtract
- https://github.com/eralpdogu/MSstatsQC
- https://github.com/stanstrup/QC4Metabolomics
- https://github.com/rformassspectrometry/RforMassSpectrometry
- https://github.com/czbiohub-sf/Rapid-QC-MS
- https://github.com/PHOENIXcenter/pi-metaboqc
- https://github.com/MonashProteomics/MaSpeQC
- https://github.com/Landeswasserversorgung/AutoQ4MS
- https://github.com/pnnl/IonToolPack

## Prioritized task list

### P0 — 
