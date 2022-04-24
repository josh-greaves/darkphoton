# Dark Photon Plots

## Limit Plots

Dark Photon limit scaling and min coupling/mixing Relic calculation

See "limit_plot_files.zip" for all files required to plot:
Relic data from DMsimp: DMS_relic.root

Relic data from HAHM dark photon model: HAHM_relic.root

Archive relic data (not plotted): Relic_v5_V_res_v2.root

CMS expected/observed data from monojet invisible decay with full detector simulation from DMsimp vector model: HEPData.root. source: cms_ex_20_004



HAHM model github: https://github.com/davidrcurtin/HAHM/edit/main/README.md

HAHM papers: arXiv:1412.0018, arXiv:1312.4992. 

HAHM DP modification github: https://github.com/violatingcp/darkphoton_highmass

Thesis: 

Other relevant papers: arXiv:2203.12035, arXiv:

 
## Relic Plots

0. Setup for DMsimp coupling scan (HAHM mixing scan) includes configuration files: Launch.txt (launch_eps.txt), generate.txt (generate_eps.txt), root_parser.py (root_parser_EPS.py), relic_compute.C (relic_compute_eps.C). These all need to be tar'd up in /MG5_aMC_v2_9_4/ in relic.tgz (included here). This is untar'd on each condor local machine during process. 
1. Run Begin.py. Loops through dark matter masses mXd and submits one job to condor for each mass. Each job will sweep through mediator masses and coupling/mixing parameter according to 'launch.txt' (called by run.sh) or 'launch_eps.txt' (called by run_eps.sh), whether you want to plot coupling for DMsimp (vector) or epsilon for HAHM (dark photon), respectively.
2. This calls run.sh (run_eps.sh), which calls generate.txt to import the appropriate model (DMsimp or HAHM), and generate the relic density fro MadDM using the parameters defined in launch.txt. 
3. Launch.txt scans the defined range of mediator masses my1 or mZDinput (DMsimp and HAHM respectively). Also scans 10^-5 to 10^5 in coupling for DMsimp, and 10^-7 to 10 in mixing parameter for HAHM.
4. Run.sh (run_eps.sh) then calls root_parser.py (root_parser_eps.py). This parses the output of MadDM (scan_run_01.txt) as a root file. The column numbers may need adjusting in this python file, depending on scan_run.txt.
5. Run.sh (run_eps.sh) then calls relic_compute.C (relic_compute_eps.C). This takes the current root file, and creates a new root file with the max and min coupling (mixing) computed in separate TTrees. (See Thesis section 'Relic Plots' for details).
6. All local outputs of parameter inputs, scan output text, and both stage root files are copied to /eos/ storage.
7. All ...proc.root output files need to be merged with 'hadd' command to be plotted with output.C script.
8. Output_dms.C (output_eps.C) generate output plot from output_proc.root (output_eps_proc.root) to show minimum heat map plot for entire scan. Previous merge step is required. Bin sizes need to be set manually in the code based on how the scan was done (resolution, how many masses scanned, etc).


