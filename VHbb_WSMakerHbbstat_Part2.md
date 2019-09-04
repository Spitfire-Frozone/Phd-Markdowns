# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat Part 2 ##

## Last Edited: 30-08-2019
--------------------------------------------------------------------------

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
# Pull plots

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

# Late Updates
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
