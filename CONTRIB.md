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
Write software to track health performance of HF and HFX LC–MS systems of the MAPP platform. Monitoring should include to allow tracking long-term performance across injections with the help of several associated metadata variables and graphs for quality control. Besides metadata directly extracted from chromatograms and spectra, include additional information like pressure of QC + blanks from thermo RAW files. Ideally, this is packed into a GUI app (either Rust or Python as back end + SQL database to store metadata and mzml files). 

If possible, try to implement a system that ideally pings designated user(s) in case of errors or warnings in the LC-MS systems or at the very least keeps track of such.

## Ressources

- https://github.com/wkumler/mzsql
- https://github.com/Rappsilber-Laboratory/MetaXtract
- https://github.com/oolonek/mzmlwatcher

## Prioritized task list

### P0 — 
