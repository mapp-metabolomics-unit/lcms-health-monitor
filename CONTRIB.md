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
        - MS status: pings the user in case an error or warning appears on the LC system
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

## Proposal

## Prioritized task list

### P0 — 