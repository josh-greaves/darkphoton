# Dark Photon Plots

## Limit Plots
Dark Photon limit scaling and min coupling/mixing Relic calculation

See "limit_plot_files.zip" for all files required to plot:

#### Files
- begin.py: either runs launch script, or submits to condor.
- run_mg5.sh / run_mg5_HAHM.sh: launch script. Writes an input.txt file with the simulation parameters (process, beam energy, cuts, etc.). One file for DMSimp (vector) one file for DPhoton (HAHM). Calls MadGraph5 with this input file.
- submit_condor_mg5.sh: condor submission. (sets max runtime/cutoff, parameter called 'JobFlavour')
- HEPdata.root: CMS expected/observed data from monojet invisible decay with full detector simulation from DMsimp vector model: cms_ex_20_004
- Relic_v5_V_res_v2.root: Archive relic data (not plotted)
- HAHM_relic.root: Relic data from HAHM dark photon model
- DMS_relic.root: Relic data from DMsimp
- delphes_card_cms_exo_20_004.tcl: input card replacement for MadAnalysis5(MA5), adding required PDG code for dark photon (18,-18). HAHM model will not run properly without this change. DMSimp vector model will.
- recasting_card.dat: replacement input selection card replacement, switching off all analyses except cms_exo_20_004.
- limit_plot.py: python code to produce the plot. Contains read-in's of relevant root files, and hardcoded values from the following spreadsheet (sorry)
- JGreaves_limits_calc.xlsx: a rudimentary but potentially helpful spreadsheet of results detailing calculations made. (Connection point between MA5 output and limit_plot.py)

#### MadGraph
For limits calculated from MadGraph simulation, run MadGraph (I used v2_9_4) to produce .hepmc output files. 
MadGraph can be temperamental. I presume here that the user already has a working version (I have not found a definitive procedure to get it working). The gist of it is: Download a clean, new version of MadGraph. At the MG5 prompt, Install hepmc, lhapdf, pythia8, madgraph_pythia_interface, madanalysis5; not necessarily in that order. When you 'launch' a simulation, the options available should include both madanalysis and pythia.
The local directory of a working version needs to be zipped up (into "mg5.tgz" in my example) in order to be un-tar'd in 'condor space', when submitting jobs to the cluster.

#### MadGraph simulation (on lxplus)
1. For looping/submitting to condor: run begin.py. This either runs the launch script, or submits to condor. NOTE: running begin.py is designed to launch Madgraph many times, with different input parameters. 
2. For a single launch: the launch scripts (e.g. run_mg5.sh) can be run individually, or the contained commands entered at the MG5 command prompt manually.
3. .hepmc's will be output in the directory designated in the launch script (e.g. run_mg5.sh)
#### MadAnalysis5 (on lxplus8)
To calculate the limits from .hepmc using madanalyis on lxplus8 (madanalysis can be a bit -very- temperamental). 

Configure MA5:
1. Ensure you are running lxplus8, create a local directory where you want to conduct your Mad analyzing...
2. source /cvmfs/sft.cern.ch/lcg/views/LCG_99/x86_64-centos8-gcc10-opt/setup.sh   -- this is a configuration setup for the environment, to a specific LCG view (LHC Computing Grid)
3. git clone https://github.com/MadAnalysis/madanalysis5.git
4. cd madanalysis5
5. ./bin/ma5 -b -f [-b force sample analyzer static library rebuild; -f means force answer 'y' to any prompt]  -- Maybe overkill - but ensures a complete build.
6. exit
7. source ./tools/SampleAnalyzer/setup.sh   -- ensures setup has completed fully
8. ./bin/ma5 -R   --    (R is 'recast' mode, necessary for this analysis)
9. install delphes
10. install PAD
11. set main.recast = on
12. REPLACE .../madanalysis5/tools/PAD/Input/Cards/delphes_card_cms_exo_20_004.tcl with the delphes_card file in the zip  --  this is the input card corresponding to the cms_exo_20_004 analysis, which is the 13TeV Run 2 monojet analysis relevant to this work. It needs to have the PDG code associated with the dark photon (18,-18) added in the appropriate places. (This step only necessary for Dark Photon HAHM model simulations). 


Run MA5 to obtain limits:
1. import ..../Events/run_01/tag_1_pythia8_events.hepmc.gz  as  name_your_output
2. submit name_your_output   (you will see Do you want to edit the recasting_card? dont answer yet.)
3. (in another window to avoid this manual step):  cp recasting_card.dat .../name_your_output/Input/       -- this switches off all analyses except cms_exo_20_004
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


