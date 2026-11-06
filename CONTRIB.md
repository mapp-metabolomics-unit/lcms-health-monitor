# LC–MS health monitor: roadmap

## Overall goal

Write a software to monitor the health state of LCMS systems. In this precise case: give the MAPP platform a quick view of the health of its HF and HFX LC–MS systems. Use long-term QCs spiked with standards and blanks to track performance across injections; show instrument warnings and errors separately from measured performance. This is a project guide, not a specification for alert thresholds or a chosen software stack.

## To be resolved 

@Colin

- [ ] return taxon name associated to each QC
- [ ] return sample_id associated to each QC
- [ ] return identity of each spiked standard (Pubchem id, link) and concentration
- [ ] create a batch_id and associate it to QC and blanks


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
- [QC4metabolomics](https://github.com/stanstrup/QC4Metabolomics): tracks m/z + RT pairs for user-defined compounds per instrument, extracts ppm/RT/intensity/peak-shape, timeline dashboard with Shiny + ggplot2/plotly, MariaDB. However, its Docker-based deployment is resource-heavy (~19 GB disk, 7-8 GB RAM), so it is more of a reference than a starting point.
- [RforMassSpectrometry](https://github.com/rformassspectrometry/RforMassSpectrometry): a broader suite of packages covering raw-data handling (Spectra), feature detection (xcms), quantification, and annotation; could supply the data-processing layer that feeds metrics into the monitoring dashboard?

### Python
- [Rapid QC-MS](https://github.com/czbiohub-sf/Rapid-QC-MS): local Python app (CZ Biohub San Francisco, *Anal. Chem.* 2024); tracks retention time, m/z, and intensity per injection; Plotly Dash dashboard with real-time updates; Slack/email alerts on QC failure; designed for Thermo Orbitrap workflows. Gaps: no MS/MS fragment metrics, no column pressure/ID extraction, no annotated feature count. Note: **is Windows-first** (requires MSConvert + MS-DIAL as external dependencies); dashboard is browser-based (Plotly Dash), not a native GUI. Possibly unmaintained (last update July 2024). **Most promising as a starting point**.
- [π-MetaboQC](https://github.com/PHOENIXcenter/pi-metaboqc): feature-intensity matrix QC and preprocessing pipeline (missingness triage, blank/QC filtering, batch correction, normalization, report generation). More of a data-curation tool than an instrument dashboard; relevant for the "annotated feature count" side?
- [MaSpeQC](https://github.com/MonashProteomics/MaSpeQC): **closest in scope**; interactive LC-MS/MS instrumentation QC (*J. Proteome Res.* 2026); tracks m/z (ppm + mDa), RT, FWHM, and normalised area for up to 15 user-defined components per QC; extracts detailed pressure profiles from Thermo RAW files (requires vendor-provided non-redistributable RawFileReader libraries); email alerts on threshold breach; MS/MS metrics limited to total MS2 spectra count, peptide spectrum matches, and unique peptides (proteomics-oriented). Gaps: no fragment-level MS/MS metrics (fragment count, max fragment intensity), no column ID tracking, no annotated feature count. Heavier multi-service web stack (Node.js + MySQL + Python + mzMine2 + Morpheus).
- [QCScreen](https://sourceforge.net/projects/qcscreen/): conceptually close (m/z, RT, mass accuracy, feature area across files; Qt GUI + SQLite) but very dated (Python 2.6, Windows XP/7 only, unmaintained since 2015).
- [AutoQ4MS](https://github.com/Landeswasserversorgung/AutoQ4MS) - checked and rejected: requires a MATLAB license.

## Resources

- https://github.com/wkumler/mzsql
- https://github.com/Rappsilber-Laboratory/MetaXtract
- https://github.com/oolonek/mzmlwatcher
- https://github.com/eralpdogu/MSstatsQC
- https://github.com/stanstrup/QC4Metabolomics
- https://github.com/rformassspectrometry/RforMassSpectrometry
- https://github.com/czbiohub-sf/Rapid-QC-MS
- https://github.com/PHOENIXcenter/pi-metaboqc
- https://github.com/MonashProteomics/MaSpeQC
- https://github.com/Landeswasserversorgung/AutoQ4MS

## Prioritized task list

### P0 — 
