# Dark Photon Plots

## Limit Plots
Dark Photon limit scaling and min coupling/mixing Relic calculation

See "limit_plot_files.zip" for all files required to plot:
Relic data from DMsimp: DMS_relic.root

Relic data from HAHM dark photon model: HAHM_relic.root
Archive relic data (not plotted): Relic_v5_V_res_v2.root
CMS expected/observed data from monojet invisible decay with full detector simulation from DMsimp vector model: HEPData.root. source: cms_ex_20_004
#### Files
- begin.py: either runs launch script, or submits to condor
- run_mg5.sh / run_mg5_HAHM.sh: launch script. Writes an input.txt file with the simulation parameters (process, beam energy, cuts, etc.). One file for DMSimp (vector) one file for DPhoton (HAHM). Calls MadGraph5 with this input file.
- submit_condor_mg5.sh: condor submission. (sets max runtime/cutoff, parameter called 'JobFlavour')

#### MadGraph simulation
For limits calculated from MadGraph simulation, run MadGraph (I used v2_9_4) to produce .hepmc output files
1. Run begin.py. This either runs the launch script, or submits to condor.
2. .hepmc's will be output in the directory designated in the launch script (e.g. run_mg5.sh)
#### MadAnalysis5
To calculate the limits from .hepmc using madanalyis on lxplus8 (madanalysis can be a bit -very- temperamental). Configure MA5:
1. Ensure you are running lxplus8, create a local directory where you want to conduct your Mad analyzing...
2. source /cvmfs/sft.cern.ch/lcg/views/LCG_99/x86_64-centos8-gcc10-opt/setup.sh   -- this is a configuration setup for the environment, to a specific LCG view (LHC Computing Grid)
3. git clone https://github.com/MadAnalysis/madanalysis5.git
4. cd madanalysis5
5. ./bin/ma5 -b -f [-b force sample analyzer static library rebuild; -f means force answer 'y' to any prompt]  -- Maybe overkill - but ensures a complete build.
6. exit
7. source ./tools/SampleAnalyzer/setup.sh   -- ensures setup has completed fully
8. Add lines TO .../madanalysis5/tools/PAD/Input/Cards/delphes_card_cms_exo_20_004.tcl: (OR just replace this input card file with the delphes_card file in the zip)  --  this is the input card corresponding to the cms_exo_20_004 analysis, which is the 13TeV Run 2 monojet analysis relevant to this work.
9. ./bin/ma5 -R   --    (R is 'recast' mode, necessary for this analysis)
10. install delphes
11. install PAD
12. set main.recast = on

Run MA5 to obtain limits:
1. import ..../Events/run_01/tag_1_pythia8_events.hepmc.gz  as  name_your_output
2. submit name_your_output   (you will see Do you want to edit the recasting_card? dont answer yet.)
3. (in another window to avoid this manual step):  cp recasting_card.dat .../name_your_output/Input/       -- this  switches off all analyses except cms_exo_20_004
4. edit card: type Y to check that all are off except the relevant analysis, or type N just to keep going.
5. The relevant output is stored: ..../madanalysis5/name_your_output/Output/SAF/CLs_output_summary.dat, and the expected/observed limits are at the very bottom.



#### Dark Photon (Hidden Abelian Higgs Model, HAHM) model info:
HAHM model github: https://github.com/davidrcurtin/HAHM/edit/main/README.md
HAHM papers: arXiv:1412.0018, arXiv:1312.4992. 
HAHM DP modification github: https://github.com/violatingcp/darkphoton_highmass

Other relevant papers: arXiv:2203.12035, arXiv:

 
## Relic Plots

MG5 version: MG5_aMC_v2_9_4.

0. Setup for DMsimp coupling scan (HAHM mixing scan) includes configuration files: Launch.txt (launch_eps.txt), generate.txt (generate_eps.txt), root_parser.py (root_parser_EPS.py), relic_compute.C (relic_compute_eps.C). These all need to be tar'd up in a directory called /MG5_aMC_v2_9_4/ in file named 'relic.tgz'. This is untar'd on each condor local machine during the process (see run.sh and run_eps.sh). 
1. Run Begin.py. Loops through dark matter masses mXd and submits one job to condor for each mass. Each job will sweep through mediator masses and coupling/mixing parameter according to 'launch.txt' (called by run.sh) or 'launch_eps.txt' (called by run_eps.sh), whether you want to plot coupling for DMsimp (vector) or epsilon for HAHM (dark photon), respectively.
2. This calls run.sh (run_eps.sh), which calls generate.txt to import the appropriate model (DMsimp or HAHM), and generate the relic density fro MadDM using the parameters defined in launch.txt. 
3. Launch.txt scans the defined range of mediator masses my1 or mZDinput (DMsimp and HAHM respectively). Also scans 10^-5 to 10^5 in coupling for DMsimp, and 10^-7 to 10 in mixing parameter for HAHM.
4. Run.sh (run_eps.sh) then calls root_parser.py (root_parser_eps.py). This parses the output of MadDM (scan_run_01.txt) as a root file. The column numbers may need adjusting in this python file, depending on scan_run.txt.
5. Run.sh (run_eps.sh) then calls relic_compute.C (relic_compute_eps.C). This takes the current root file, and creates a new root file with the max and min coupling (mixing) computed in separate TTrees. (See Thesis section 'Relic Plots' for details).
6. All local outputs of parameter inputs, scan output text, and both stage root files are copied to /eos/ storage.
7. All ...proc.root output files need to be merged with 'hadd' command to be plotted with output.C script.
8. Output_dms.C (output_eps.C) generate output plot from output_proc.root (output_eps_proc.root) to show minimum heat map plot for entire scan. Previous merge step is required. Bin sizes need to be set manually in the code based on how the scan was done (resolution, how many masses scanned, etc).


