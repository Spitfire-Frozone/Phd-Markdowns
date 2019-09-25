# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat Part 2 ##

## Last Edited: 25-09-2019
--------------------------------------------------------------------------
- Useful Links

https://nmorange.web.cern.ch/nmorange/WSMaker/html/HowTo.html

https://gitlab.cern.ch/atlas-physics/higgs/hbb/WSMaker/blob/master/HowTo.md

# Initial Setup and Running
Once a basic familiarity with WSMaker has been established, and a newer set of inputs has been created. These will need to be tested.  

1) MC16ad new + new regions Global Conditional
2) MC16ad new + new regions Asimov
3) MC16ad new + new regions Asimov fitting only CRs
4) MC16ad new + new regions Global Conditional fitting only CRs

- Pull plots 1 vs 2
- Pull plots 1 vs MC16ad old
- Pull plots 1 vs 3
- Pull Plots 1+2 vs 4

The first thing though, is to check out a new version of the WSMaker. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git 
(mv WSMaker_VHbb WSMaker_VHbb_WZjets) #Temprorarily move this such that the git clone works.
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git


mv WSMaker_VHbb WSMaker_VHbb_New0Linputs (&& mv WSMaker_VHbb_WZjets WSMaker_VHbb)
cd WSMaker_VHbb_New0Linputs
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..
~~~

Now we have to switch to a specific branch
~~~
git branch --all | grep kalkhour
git checkout -b master-kalkhour-Adapt_WS-to_NewRegions origin/master-kalkhour-Adapt_WS-to_NewRegions --track
~~~

The next step is to split the new inputs. To do this we need to create a new file. Referencing files from /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_customCDI_20190820. To perform the first 4 tasks, we need all So ade 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_New0Linputs
setupATLAS && lsetup git 
cd inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_v02_0L.txt
~~~
> ADD path ot 0L file (~L1)
>  >    CoreRegions
>  >    ZeroLepton ZeroLep/r32-15_customCDI_20190820/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.32-15NewRegionsOldExtensions.root mBB,MET,mva,mvadiboson 
~~~
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_v02_0L
~~~
With the inputs now split, it's time to start running the fit. We will need to run this 4 times. 

### 1) MC16ad new + new regions Global Conditional
~~~
vim scripts/launch_default_jobs.py
~~~
>  CHANGE Variables to run the new 0L baseline FullRun2. All other boolean variables should be false
>   >  version = "v02_0L"       (~L13)
>   >  GlobalRun = True         (~L14)
>   >  channels = ["0"]         (~L48)
>   >  MCTypes = ["mc16ade"]    (~L50)
>   >  syst_type = ["Systs"]    (~L53)
>   >  runPulls = True          (~L64)
>   >  doExp = "0"              (~L57)
>   >  runP0 = False            (~L68)
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-SRCR

mv output/SMVHVZ_2019_MVA_mc16ade_v02_0L.140ifb-0L-ade-SRCR_fullRes_VHbb_140ifb-0L-ade-SRCR_0_mc16ade_Systs_mva output/140ifb-0L-ade-SRCR
~~~
### 4) MC16ad new + new regions Global Conditional fitting only CRs
~~~
vim scripts/AnalysisMgr_VHbbRun2.py
~~~
>  COMMENT OUT SR region from new regions to only run over the CR's (~L57)
>   >  #self["Regions"].append(energy+"_ZeroLepton_{0}tag{1}jet_{2}ptv_SR_{3}".format(t, j, p, var2tag)) 
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-CR

mv output/SMVHVZ_2019_MVA_mc16ade_v02_0L.140ifb-0L-ade-CR_fullRes_VHbb_140ifb-0L-ade-CR_0_mc16ade_Systs_mva output/140ifb-0L-ade-CR
~~~
### 3) MC16ad new + new regions Asimov fitting only CRs
~~~
vim scripts/launch_default_jobs.py
~~~
>  CHANGE Variables from Global conditional run to Asimov run.
>   >  GlobalRun = False        (~L14)
>   >  runPulls = True          (~L64)
>   >  doExp = "1"              (~L57)
>   >  runP0 = True             (~L68)
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-CR-Asimov

mv output/SMVHVZ_2019_MVA_mc16ade_v02_0L.140ifb-0L-ade-CR-Asimov_fullRes_VHbb_140ifb-0L-ade-CR-Asimov_0_mc16ade_Systs_mva output/140ifb-0L-ade-CR-Asimov
~~~
### 2) MC16ad new + new regions Asimov
~~~
vim scripts/AnalysisMgr_VHbbRun2.py
~~~
>  COMMENT IN SR region from new regions to only run over the CR's (~L57)
>   >  self["Regions"].append(energy+"_ZeroLepton_{0}tag{1}jet_{2}ptv_SR_{3}".format(t, j, p, var2tag)) 
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-SRCR-Asimov

mv output/SMVHVZ_2019_MVA_mc16ade_v02_0L.140ifb-0L-ade-SRCR-Asimov_fullRes_VHbb_140ifb-0L-ade-SRCR-Asimov_0_mc16ade_Systs_mva output/140ifb-0L-ade-SRCR-Asimov
~~~
## Pull plots

There are several kinds of fits you want to compare and they all can't be run with the same flag. See lines 414-421 of comparePulls.py. However for the data vs Asimov one we only run the one folder and we use the flag 5.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_New0Linputs
setupATLAS && lsetup git 
source setup.sh
cd build && cmake ..
make -j10
cd ..

vim WSMakerCore/scripts/comparePulls.py
~~~  
>   2: unconditional ( start with mu=1 )                                                                             
>   4: conditional mu = 0                                                                                 
>   5: conditional mu = 1                                                                                              
>   6: run Asimov mu = 1 toys: randomize Poisson term                                                                      
>   7: unconditional fit to asimov where asimov is built with mu=1                                                   
>   8: unconditional fit to asimov where asimov is built with mu=0                                                    
>   9: conditional fit to asimov where asimov is built with mu=1                                                      
>   10: conditional fit to asimov where asimov is built with mu=0                                                          

~~~
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-SRCR -n -a 5
mv output/pullComparisons output/pullComparisons_SRCR


python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-CR-Asimov 140ifb-0L-ade-SRCR-Asimov -n -a 7 -l CR-Asv SR-CR-Asv
mv output/pullComparisons output/pullComparisons_Asimov
~~~

## Late Updates
Turns out someone wants a quick check of pulls for ad vs e for the new nominal input. After creating two new files called vim SMVHVZ_2019_MVA_mc16e_v02_0L.txt and vim SMVHVZ_2019_MVA_mc16ad_v02_0L.txt in inputConfigs.

LimitHistograms.VHbb.0Lep.13TeV.mc16ad.Oxford.32-15.v02.root
LimitHistograms.VHbb.0Lep.13TeV.mc16e.Oxford.32-15NewRegionsOldExtensions.root
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_New0Linputs
setupATLAS && lsetup git 
source setup.sh
cd build && cmake ..
make -j10
cd ..

SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ad_v02_0L
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16e_v02_0L


vim scripts/launch_default_jobs.py
~~~
>  CHANGE Variables to run the new 0L baseline FullRun2, and the MCTypes to "mc16ad"
>   >  version = "v02_0L"       (~L13)
>   >  GlobalRun = True         (~L14)
>   >  channels = ["0"]         (~L48)
>   >  MCTypes = ["mc16ad"]     (~L50)
>   >  syst_type = ["Systs"]    (~L53)
>   >  runPulls = True          (~L64)
>   >  doExp = "0"              (~L57)
>   >  runP0 = False            (~L68)
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ad-SRCR

mv output/SMVHVZ_2019_MVA_mc16ad_v02_0L.140ifb-0L-ad-SRCR_fullRes_VHbb_140ifb-0L-ad-SRCR_0_mc16ad_Systs_mva output/140ifb-0L-ad-SRCR

vim scripts/launch_default_jobs.py
~~~
>  CHANGE the MC period
>   >  MCTypes = ["mc16e"]     (~L50)
~~~
python scripts/launch_default_jobs.py 140ifb-0L-e-SRCR

mv output/SMVHVZ_2019_MVA_mc16e_v02_0L.140ifb-0L-e-SRCR_fullRes_VHbb_140ifb-0L-e-SRCR_0_mc16e_Systs_mva output/140ifb-0L-e-SRCR

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-SRCR 140ifb-0L-ad-SRCR 140ifb-0L-e-SRCR -n -a 5 -l ade ad e
mv output/pullComparisons output/pullComparisons_adeade
~~~
# New Inputs
Plots for the new baseline. This time the splitInputs have been provided for 0, 1 and 2 Lepton. These inputs have additional control regions in dRBB and operate with different binnin regimes.

>   /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-08-20/milestone1
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_New0Linputs/inputs 
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-08-20/milestone1 .
mv milestone1 SMVHVZ_2019_MVA_mc16ade_v03_baseline

vim scripts/launch_default_jobs.py
~~~
>    CHANGE Variables to run the new 0L baseline FullRun2, and the MCTypes to "mc16ade"
>   >  version = "v03_baseline" (~L13)                                                                                   
>   >  GlobalRun = True         (~L14)                                                                                       
>   >  channels = ["0"]         (~L48)                                                                                        
>   >  MCTypes = ["mc16ade"]    (~L50)                                                                                     
>   >  syst_type = ["Systs"]    (~L53)                                                                                    
>   >  runPulls = True          (~L64)                                                                                     
>   >  doExp = "0"              (~L57)                                                                                      
>   >  runP0 = True             (~L68)                                                                                   

~~~
setupATLAS && lsetup git 
source setup.sh
cd build && rm -rf *
cmake .. && make -j10
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-baseline
mv output/SMVHVZ_2019_MVA_mc16ade_v03_baseline.140ifb-0L-ade-baseline_fullRes_VHbb_140ifb-0L-ade-baseline_0_mc16ade_Systs_mva output/140ifb-0L-ade-baseline

~~~
# Investigation of different binning regimes with Significances
We also want to change the binning for some of the control regions. This is done in binning_vhbbrun2.cpp .We have too many bins. Taking the convention CRLow high-ptv/CRLow Extreme-ptv/CRHigh high-ptv/CRHigh Extreme-ptv we currently have <6/<6/<6/<6 bins which ends up with 5/4/5/4. This ends up with too few events in the bins of CRLow Extreme-ptv. Also, since 1L have got good significance with fewer bins we want to try two regimes.

- 1/1/1/1 and 
- 2/1/2/2
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_New0Linputs/

vim src/binning_vhbbrun2.cpp
~~~
- For option 1
>    COMMENT OUT  extreme-ptv exception
>   >     //if (c[Property::binMin] == 250) return {6}; 
>    CHANGE the default number of bins to be run to be 1 (L114)
>   >     else return {2};  -> return oneBin(c);

- For option 2
>   ADD TH1* object for fixing of bins. (L103)
>   >       TH1* sig = c.getSigHist();

>    COMMENT OUT previous lines in doNewRegions if statement (L113).
>    ADD differing bins between CRHigh and CRLow (L114)
>   >       if (doNewRegions){                                                                      
>   >           if (c(Property::descr).Contains("CRLow") && (c(Property::dist) == "pTV" || c(Property::dist) == "MET")) {
>   >             if (c[Property::binMin] == 250) return oneBin(c);
>   >             else return {2};                                                                      
>   >           }                                                  
>   >           if (c(Property::descr).Contains("CRHigh") && (c(Property::dist) == "pTV" || c(Property::dist) == "MET")) {
>   >             return {2};
>   >           }                                                                                         
>   >        }                                                                                    
>    ADD forcing of bin changing in CRHigh and CRLow in 0L (L132)
>   >       if(c[Property::nLep] == 0) res = {2}; ->  if(c[Property::nLep] == 0) {
>   >           if (doNewRegions) {                                                              
>   >               if (c(Property::descr).Contains("CRLow") && (c(Property::dist) == "pTV" || c(Property::dist) == "MET")){
>   >                   if (c[Property::binMin] == 150) res = getBinsFromEdges(sig,{250,200,150});
>   >               }                                                                      
>   >               if (c(Property::descr).Contains("CRHigh") && (c(Property::dist) == "pTV" || c(Property::dist) == "MET")) {
>   >                   if (c[Property::binMin] == 150) res = getBinsFromEdges(sig,{250,200,150});
>   >                   if (c[Property::binMin] == 250) res = getBinsFromEdges(sig,{480,370,260});
>   >               }                                                                                            
>   >           }                                                                                       
>   >       }                                                                                                  
~~~
vim scripts/launch_default_jobs.py
~~~
>    CHANGE runP0 so that one can compare significances.
>   >  runP0 = True            (~L68)
~~~
setupATLAS && lsetup git 
source setup.sh
cd build && cmake ..
make -j10
cd ..
~~~
For option 1
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-baseline1111
mv output/SMVHVZ_2019_MVA_mc16ade_v03_baseline.140ifb-0L-ade-baseline1111_fullRes_VHbb_140ifb-0L-ade-baseline1111_0_mc16ade_Systs_mva output/140ifb-0L-ade-baseline1111
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-baseline 140ifb-0L-ade-baseline1111 -n -a 5 -l CR-qFree CR-1111 
mv output/pullComparisons output/pullComparisons_1111
~~~
For option 2
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-baseline2122
mv output/SMVHVZ_2019_MVA_mc16ade_v03_baseline.140ifb-0L-ade-baseline2122_fullRes_VHbb_140ifb-0L-ade-baseline2122_0_mc16ade_Systs_mva output/140ifb-0L-ade-baseline2122
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-baseline 140ifb-0L-ade-baseline2122 -n -a 5 -l CR-qFree CR-2122 
mv output/pullComparisons output/pullComparisons_2122

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-baseline 140ifb-0L-ade-baseline1111 140ifb-0L-ade-baseline2122 -n -a 5 -l CR-qFree CR-1111  CR-2122 
mv output/pullComparisons output/pullComparisons_11112122
~~~
To obtain the significances of the different fits you can print the last 10 lines of the log to the screen
~~~
tail -n 10 140ifb-0L-ade-baseline/logs/output_getSig_125.log
tail -n 10 140ifb-0L-ade-baseline1111/logs/output_getSig_125.log
tail -n 10 140ifb-0L-ade-baseline2122/logs/output_getSig_125.log
~~~

# Running Official 0L Plots 
Once we understand the fit we can produce the official plots for public consumption. Essentially we want to update the 0L standalone plots in here Appendix B of https://cds.cern.ch/record/2317111/files/ATL-COM-PHYS-2018-512.pdf . Hence will need to run the WSMaker four times. twice on MV analysis and twice on the CB analysis.

1) Run the MV analysis with Rankings, Breakdowns and Significances       
2) Run the MV analysis over the post-fit variables and with the EXPECTED pulls  

3) Run the CB analysis with Rankings, Breakdowns and Significances          
4) Run the CB analysis over the post-fit variables and with the EXPECTED pulls             

Here is a summary of the plots that we require

| Designation |      Plots                           |  Runs  |
|:-----------:|:-------------------------------------|:------:|
| a           | MVA Asimov VS Data                   |  (2)   |
| b           | MVA ranking and breakdown            | (1,3)  |
| c           | MVA post-fit plots for mBB (and PtV?)|  (2)   |
| d           | CBA VS mva Or/And cba Asimov VS Data | (1,4)  |
| e           | CBA ranking and breakdown            |  (3)   |
| f           | CBA post-fit plots for mBB (and PtV?)|  (4)   |



~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git 
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git
mv WSMaker_VHbb WSMaker_VHbb_Official
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..
~~~
The inputs have already been split, and these are special ones that will require 

>  /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-08-20/milestone1_STXS
~~~
cd inputs
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-08-20/milestone1_STXS .
mv milestone1_STXS SMVHVZ_2019_MVA_mc16ade_milestone1_STXS
cd ..

vim setup.sh
~~~
>    CHANGE the analysis to be blinded
>   >  export IS_BLINDED = 0 -> export IS_BLINDED = 1
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
source setup.sh
cd build && cmake ..
make -j10
cd ..
~~~

### 1) Run the MV analysis with Rankings, Breakdowns and Significances       

Then you will want to run the mva analysis on launch_default_jobs.py with the following configuration. If it's not specified underneath, it should be run as 'false'            
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13-L15)
>   >  version = "milestone1_STXS"                                                                                    
>   >  GlobalRun = False            
>   >  doPostFit = False                                                                                                

>    CHANGE all do cutbase block to 'false' (~L17-L23)
>   >  doCutBase = False        (~L17)
>   >  do_mbb_plot = False      (~L23)                                                                                     

>    CHANGE the STXS block suck that we can run the 1 POI scheme (~L27-L32)
>   >  doSTXS = True                                                                                            
>   >  FitSTXS_Scheme = 1 #3 corresponds to 5 POI                                                                   
>   >  doSTXSQCD = True                                                                                         
>   >  doSTXSPDF = True                                                                               
>   >  doSTXSDropTheoryAcc= True                                                  
>   >  doXSWS = False # switch to true for 3/5 POIs                                                                        

>    CHANGE variable such that you run over the new CR's (~L34)
>   >  doNewRegions = True 

>    CHANGE variables so you are running the observed 0L standalone fit (~L46-L57)
>   >  channels = ["0"]         (~L48)                                                                                        
>   >  MCTypes = ["mc16ade"]    (~L50)                                                                                     
>   >  syst_type = ["Systs"]    (~L53)

>    CHANGE to run over the Asimov Dataset    
>   >  doExp = "1"              (~L57)                                                                                     

>    CHANGE variables to run on the batch as this will be hefty job (~L60)
>   >  run_on_batch = True                                                                                             

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)
>   >  createSimpleWorkspace = True                                                                                
>   >  runPulls = False                                                                                                       
>   >  runBreakdown = True                                                                                              
>   >  runRanks = True                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runP0 = True                                                                                                        
>   >  runToyStudy = False                                                                                                 

Then once you are ready you can run
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA
~~~

### 2)  Run the MV analysis over the post-fit variables and with the EXPECTED pulls  
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE postfit flag to get the mBB plots (~L17)
>   >  doPostFit = True    

>    CHANGE flags to get expected pulls (~L57-L70)                                      
>   >  doExp=0                                                                                                          
>   >  runPulls = True                                                                                                     
>   >  runBreakdown = False                                                                                              
>   >  runRanks = False                                                                                                
>   >  runP0 = False                                                                                                
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-mBBpull
~~~

### 4) Run the CB analysis over the post-fit variables and with the EXPECTED pulls     
Then all you need to do to run over the CB analysis is to change two lines in launch_default_jobs.py 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE cutbase block to 'true' (~L17)
>   >  doCutBase = True                                                                                                 
>   >  doPostFit = false                                                             
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-CBA-mBBpull
~~~

### 3) Run the CB analysis with Rankings, Breakdowns and Significances  
We do not need  doPostFit = True here as we are only interested in mBB and mBB comes for free in the CBA  
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flags to get Rankings, Breakdowns and Significances (~L57-L70) 
>   >  doExp=1                                                                                                             
>   >  runPulls = False                                                                                                   
>   >  runBreakdown = True                                                                                              
>   >  runRanks = True                                                                                                
>   >  runP0 = True                                                                                                       
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-CBA
~~~        
## Organising directories for plots.

If you ran your jobs on the grid then the output from lxbatch jobs is stored in:
/afs/cern.ch/work/${USER:0:1}/${USER}/analysis/statistics/batch/
> /afs/cern.ch/work/d/dspiteri/analysis/statistics/batch/ for example

and can be gathered using getResults.py
> python scripts/getResults.py list python WSMakerCore/scripts/getResults.py list --plots --tables --NPs --fccs --workspaces --ranking --breakdown --restrict-to fullRes $version
with more examples in gather_default_results.sh
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
source setup.sh

python WSMakerCore/scripts/getResults.py list --plots --tables --NPs --fccs --workspaces --ranking --breakdown --restrict-to fullRes /afs/cern.ch/work/d/dspiteri/analysis/statistics/batch/SMVHVZ_2019_MVA_mc16ade_milestone1_STXS.140ifb-0L-ade-STXS-baseline-MVA.140ifb-0L-ade-STXS-baseline-MVA

~~~
If not go to the usual output folder and move some of the ungainly folders around. 
~~~
cd output 

mv SMVHVZ_2019_MVA_mc16ade_milestone1_STXS.140ifb-0L-ade-STXS-baseline-CBA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-CBA_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated 140ifb-0L-ade-STXS-baseline-CBA
mv SMVHVZ_2019_MVA_mc16ade_milestone1_STXS.140ifb-0L-ade-STXS-baseline-CBA-mBBpull_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-CBA-mBBpull_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated 140ifb-0L-ade-STXS-baseline-CBA-mBBpull
mv SMVHVZ_2019_MVA_mc16ade_milestone1_STXS.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated 140ifb-0L-ade-STXS-baseline-MVA
mv SMVHVZ_2019_MVA_mc16ade_milestone1_STXS.140ifb-0L-ade-STXS-baseline-MVA-mBBpull_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-mBBpull_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated 140ifb-0L-ade-STXS-baseline-MVA-mBBpull
mv SMVHVZ_2019_MVA_mc16ade_milestone1_STXS.140ifb-0L-ade-STXS-baseline-MVA-mBBpull_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-mBBpull_0_mc16ade_Systs_MET_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated 140ifb-0L-ade-STXS-baseline-MVA-METpull
rm -rf SMVHVZ_2019_MVA_mc16ade_milestone1_STXS*
~~~
The last thing that needs to be prepared then are the two sets of pull plots.

