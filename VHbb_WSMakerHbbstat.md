# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## "VHbb WSMaker and Hbb Stat" ##
===============================================================================
## Last Edited: 15-06-2019

Once a basic familiarity with WSMaker has been established, (see VHbb_2Lep_Reader_Inputs.md ) a task, which gets more and more priority is to test reducing the number of bins in the inputs.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
~~~
The first task is to run on official Inputs and run the MVA Fit which you can follow the instructions on 
```
https://gitlab.cern.ch/atlas-physics/higgs/hbb/WSMaker_VHbb/blob/master/scripts/ObservationResults_HowTo.md
```
To pull a branch that already exists
~~~
git checkout -b dev-tcalvet origin/dev-tcalvet
~~~
The first step is to split the inputs 
~~~
vim setup.sh
~~~
>  CHANGE blinding to be off. 
>   >     IS_BLINDED = 0 (L6)
~~~
source setup.sh
cd build && cmake ..
make -j10
cd ..
SplitInputs -r Run2 -v SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2
~~~
One the inputs have been split, open the job launcher and ensure that  that these settings are there
~~~
vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline Summer "ICHEP" (~L13)
>   >     version = "v07_fixed2"
>   >     GlobalRun = False 
>   >     doPostFit = False

>   >     doCutBase = False
>   >     doCutBasePlots = False 
>   >     doDiboson = False
>   >     Add1LMedium = False
>   >     doTTbarDataDriven2L = False
>   >     do_mbb_plot = False

>   >     # Options for STXS
>   >     doSTXS = False
>   >     FitSTXS_Scheme = 3 #3 corresponds to 5 POI
>   >     doSTXSQCD = True
>   >     doSTXSPDF = True
>   >     doSTXSDropTheoryAcc= True
>   >     doXSWS = True

>   >     # Channels                                                          
>   >     channels = ["012"]                                                 
>   >     MCTypes = ["mc16ad"]   

>   >     # What to run
>   >     createSimpleWorkspace = True
>   >     runPulls = True
>   >     runBreakdown = False
>   >     runRanks = False 
>   >     runLimits = False
>   >     runP0 = True                                                       
>   >     runToyStudy = False

>   >     # Turn on additional debug plots
>   >     doplots = True  

Once a configuration is chosen, one just has to type
~~~
python scripts/launch_default_jobs.py ICHEP1000Poubelle
~~~

So now the standard is that there is 1000 bins in each of the plots that form the input. They can be found in the inputs/SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2 directory.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/inputs
mkdir SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2_500
cp -r SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2 SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2_500
mkdir SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2_250
cp -r SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2 SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2_250

cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb

root -b -l -q '../RebinHistos.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/inputs/","SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2_500",500)' >> bins.log

root -b -l -q '../RebinHistos.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/inputs/","SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2_250",250)' >> bins250.log

vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline Summer "ICHEP" (~L13)
>   >     version = "v07_fixed2_500"
~~~
source setup.sh
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py ICHEP500Poubelle


vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline Summer "ICHEP" (~L13)
>   >     version = "v07_fixed2_250"
~~~
source setup.sh
python scripts/launch_default_jobs.py ICHEP250Poubelle
~~~

# Fit Debugging
Works in a very similar way to the above to start. This time we are comparing ICHEP to a new version. And we are only doing one channel, this time it is the 0L channel. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
source setup.sh
cd build && cmake ..
make -j10
cd ..
SplitInputs -r Run2 -v SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_v01
~~~
Then edit the launch_default_jobs.py with the correct settings for each prepared input (for ease I will only go over the newest one)
~~~
vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline FullRun2 (~L13)
>  >     version = "v01"

>   >     channels = ["0"]
>   >     MCTypes = ["mc16ade"]

>   >     createSimpleWorkspace = True
>   >     runPulls = True
>   >     runBreakdown = False
>   >     runRanks = False
>   >     runLimits = False
>   >     runP0 = False
>   >     runToyStudy = False

>   >     doplots = True

Everything else should be false
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
source setup.sh
lsetup git
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade
~~~
Once you have the multiple sets of plots, you can compare thigns like pulls by running 

python WSMakerCore/scripts/comparePulls.py -w FolderName1 FolderName2 -n -a 5 -l name1 name2

Here I will rename the output folder such that the commands are easier to read. 
~~~
mv output/SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade_fullRes_VHbb_140ifb-0L-ade_0_mc16ade_Systs_mva output/140ifb-0L-ade
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade ICHEP-0L-ad -n -a 5 -l 140ifb ICHEP

cd output
mv pullComparisons pullComparisons_0L_ICHEP-140ifb
~~~

If you see that in the pull plots that there are things wrong there are 2 tests that you want to do which are at two different levels:
- Understand overall picture or group of pulls: here you can change your model, in our case we could be interested in running a fit without the split at 250 GeV to see the impact of this boundary on all pulls (may be a 3J only one also, but less clear)
- Understand the origin of single pulls and validate them: here you usually run a decorrelation of that NP across the physics observables used to build the categories. In our case, there are 2 directions for the decorrelation: N( jet ) or PtV, and there are 2 uncertainties which decorrelation we are interested in at first:
- ZPtV: because it has a large change, and with new PtV regions we might expect changes in this one. For this one only the N( jet ) decorr is interesting, because it is a PtV unc, so if you decorrelate across PtV it makes no sense
- ttbarMbb: because it has a large change again, for this one we can do both decorrelations


Lets decorrelate Njets in ttbarMbb for example
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
> CHANGE SysConfig to decorrelate systematic in nJets (L506)
>   >     m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}) }); -> m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}).decorr(P::nJet) });
~~~
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade

cd output
mv SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade_fullRes_VHbb_140ifb-0L-ade_0_mc16ade_Systs_mva 140ifb-0L-ade_decorr_njets_ttbarmbb
mv pullComparisons pullComparisons_standard
cd ..
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade_decorr_njets_ttbarmbb 140ifb-0L-ade -n -a 5 -l 140ifb ttbardecnjets
mv output/pullComparisons output/pullComparisons_decorr_njets_ttbarmbb

~~~
If you can see variables like `_SysTTbarPTV_J2` in the ranking plot, it indicates that 2-jet and 3-jet region are now de-correlated.

Lets decorrelate ptV in ttbarMbb for example
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
> CHANGE SysConfig to decorrelate systematic in PtV (L506)
>   >     m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}) }); -> m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75),(P::nLep==0)&&(P::binMin==250)}) });
~~~
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade

cd output
mv SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade_fullRes_VHbb_140ifb-0L-ade_0_mc16ade_Systs_mva 140ifb-0L-ade_decorr_ptv_ttbarmbb

cd ..
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade_decorr_ptv_ttbarmbb 140ifb-0L-ade -n -a 5 -l 140ifb ttbardecptv
cd output
mv pullComparisons pullComparisons_decorr_ptv_ttbarmbb
~~~
# Fit Debugging Part 2.
So we have some results and we want to investigate them further. The first thing that we want to do is to check the the errors we get are consistent across MC periods. The first port of call then is to split the MC16ade into MC16a+d and MC16e, and compare these to MC16ade. To do this we need to create a new inputConfigs file with these split MC regions. Where do we find these?

>   /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/


0L| ZeroLep/v5/Non-STXS/LimitHistograms.VHbb.0Lep.13TeV.mc16ad.Oxford.r32-15v5.root 
0L| ZeroLep/v5/Non-STXS/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Oxford.r32-15v5.root 
1L| OneLep/r32-15_customCDI/no_STXS/LimitHistograms.VHbb.1Lep.13TeV.mc16ad.LAL.v01.NoSTXS.r32-15_customCDI.root
1L| OneLep/r32-15_customCDI/no_STXS/LimitHistograms.VHbb.1Lep.13TeV.mc16e.LAL.v01.NoSTXS.r32-15_customCDI.root
2L| TwoLep/r32-15_customCDI/v2/LimitHistograms.VHbb.2Lep.13TeV.mc16ad.Kyoto.r32-15_customCDI_v2.root
2L| TwoLep/r32-15_customCDI/v2/LimitHistograms.VHbb.2Lep.13TeV.mc16e.Kyoto.r32-15_customCDI_v2.root

## Splitting the inputs in MC regions 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_v01.txt
~~~
> CHANGE all instances of ade -> ad and save as SMVHVZ_2019_MVA_mc16ad_v01.txt
> CHANGE all instances of ade -> e and save as SMVHVZ_2019_MVA_mc16e_v01.txt

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
source setup.sh
cd build && cmake ..
make -j10
cd ..
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ad_v01
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16e_v01
~~~

~~~
vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline FullRun2 (~L13)
>  >     version = "v01"

>   >     channels = ["0"]
>   >     MCTypes = ["mc16ad"]
~~~

vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
> CHANGE SysConfig to decorrelate systematicto default (L506)
>   >     m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}) });
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
source setup.sh
lsetup git
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ad

cd output
mv SMVHVZ_2019_MVA_mc16ad_v01.140ifb-0L-ad_fullRes_VHbb_140ifb-0L-ad_0_mc16e_Systs_mva 140ifb-0L-ad
~~~
Repeat the above changing the MCTypes to mc16e
~~~
python scripts/launch_default_jobs.py 140ifb-0L-e

cd output
mv SMVHVZ_2019_MVA_mc16e_v01.140ifb-0L-e_fullRes_VHbb_140ifb-0L-e_0_mc16e_Systs_mva 140ifb-0L-e
~~~

Now we want to generate pulls comparing each of ad and e to the nominal ade
~~~
/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade 140ifb-0L-ad -n -a 5 -l 140ade 140ad
mv output/pullComparisons output/pullComparisons_ade_vs_ad
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade 140ifb-0L-e -n -a 5 -l 140ade 140e
mv output/pullComparisons output/pullComparisons_ade_vs_e

OR

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade 140ifb-0L-ad 140ifb-0L-e -n -a 5 -l 140ade 140ad 140e
mv output/pullComparisons output/pullComparisons_ade_vs_ad_vs_e
~~~
## Asimov fit with significance
We need to see if the decorrelation of the ttbarMBB variable in Njets and ptv is an issue that needs to be resolved. One of the ways to test if this is seomthing that we should worry about is to perform an Asimov fit as opposed to a Global conditional one, and then check the decorrelations impact on the significances. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
source setup.sh
lsetup git

vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline FullRun2 (~L13)
>  >     version = "v01"

>   >     channels = ["0"]
>   >     MCTypes = ["mc16ade"]
>   >     runP0 = True             (L66)

~~~
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-Asimov

cd output
mv SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade-Asimov_fullRes_VHbb_140ifb-0L-ade-Asimov_0_mc16ade_Systs_mva 140ifb-0L-ade-Asimov
cd ..

vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
> CHANGE SysConfig to decorrelate systematic in PtV (L506)
>   >     m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}) }); -> m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75),(P::nLep==0)&&(P::binMin==250)}) });
~~~
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-Asimov

cd output
mv SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade-Asimov_fullRes_VHbb_140ifb-0L-ade-Asimov_0_mc16ade_Systs_mva 140ifb-0L-ade-Asimov_decorr_ptv_ttbarmbb
cd ..

vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
> CHANGE SysConfig to decorrelate systematic in nJets (L506)
>   >     m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}) }); -> m_histoSysts.insert( {"SysTTbarMBB", SysConfig{T::shapeonly, S::noSmooth, Sym::noSym}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}).decorr(P::nJet) });
~~~
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-Asimov

cd output
mv SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade-Asimov_fullRes_VHbb_140ifb-0L-ade-Asimov_0_mc16ade_Systs_mva 140ifb-0L-ade-Asimov_decorr_njets_ttbarmbb
cd ..
~~~
To get significances you have to go into the logs folder. And you want the median significance which is the expected. If you have something in observed close it immediately and show it to no one. You have done fucked up. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/


tail -n 10 140ifb-0L-ade-Asimov/logs/output_getSig_125.log
tail -n 10 140ifb-0L-ade-Asimov_decorr_njets_ttbarmbb/logs/output_getSig_125.log
tail -n 10 140ifb-0L-ade-Asimov_decorr_ptv_ttbarmbb/logs/output_getSig_125.log
~~~
Now we want to generate pulls comparing each of the ttbarMBB  to the nominal Asimov. Lines 410-420 in comparePulls.py tells you what number to change. 
~~~
/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
source setup.sh
lsetup git
cd build && cmake ..
make -j10
cd ..

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-Asimov 140ifb-0L-ade-Asimov_decorr_njets_ttbarmbb -n -a 7 -l Asimov ttbarDecNjets
mv output/pullComparisons output/pullComparisons_Asimov_decorr_njets_ttbarmbb

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-Asimov 140ifb-0L-ade-Asimov_decorr_ptv_ttbarmbb -n -a 7 -l Asimov ttbarDecPtv
mv output/pullComparisons output/pullComparisons_Asimov_decorr_ptv_ttbarmbb

cd output/140ifb-0L-ade-Asimov/plots/shapes
imgcat Region_BMax250_BMin150_Y6051_DSR_T2_L0_distmva_J2_ttbar_SysTTbarMBB.png
imgcat Region_BMax250_BMin150_Y6051_DSR_T2_L0_distmva_J3_ttbar_SysTTbarMBB.png
imgcat Region_BMin250_Y6051_DSR_T2_L0_distmva_J2_ttbar_SysTTbarMBB.png
imgcat Region_BMin250_Y6051_DSR_T2_L0_distmva_J3_ttbar_SysTTbarMBB.png

cd ../../..
cd 140ifb-0L-ade-Asimov_decorr_njets_ttbarmbb/plots/shapes
imgcat *_ttbar_SysTTbarMBB_*

cd ../../..
cd 140ifb-0L-ade-Asimov_decorr_ptv_ttbarmbb/plots/shapes
imgcat *_ttbar_SysTTbarMBB_*

cd ../../..
cd 140ifb-0L-ade/plots/shapes
imgcat *ttbar_SysTTbarMBB*

cd ../../..
cd 140ifb-0L-ad/plots/shapes
imgcat *ttbar_SysTTbarMBB*

cd ../../..
cd 140ifb-0L-e/plots/shapes
imgcat *ttbar_SysTTbarMBB*
~~~
Making sure to repeat the above steps for the following variable strings Zhf_SysZMbb, ttbar_SysTTbarPTV, Zhf_SysZPtV


## Looking at input variables
It would also be good to look at the distribution of input variables that go into the BDT, but as a default only the BDT plots are output. Firstly one needs to add the kinematic variables you want to the related inputConfig file and Split the histograms.
The list of a set of kinematic variables for 
0L| MET,pTB1,pTB2,mBB,dRBB,dEtaBB,dPhiVBB,MEff,MEff3,pTJ3,mBBJ
1L| pTV,MET,pTB1,pTB2,mBB,dRBB,dPhiVBB,dPhiLBmin,mTW,dYWH,Mtop,pTJ3,mBBJ,MET
2L| pTV,MET,pTB1,pTB2,dRBB,dPhiVBB,dEtaVBB,mLL,pTJ3,mBBJ3,mBB,dEtaBB

Then one needs to change the location of the files themselves to point to the samples that have all the kinematic variables. Remember the base can be found here. 

/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/

ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15varInputs.root
TwoLep/r32-15_allMVAVariables/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.IOWAUSTC.r32-15_AllMVAVariables.root

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_v01.txt
~~~
> CHANGE all instances of mva,mvadiboson with the relevant set of kinematic variables
> CHANGE Absolute path files
>   >     ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15varInputs.root
>   >      TwoLep/r32-15_allMVAVariables/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.IOWAUSTC.r32-15_AllMVAVariables.root
> Call it SMVHVZ_2019_MVA_mc16ade_v01_Vars.txt
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
source setup.sh
cd build && cmake ..
make -j10
cd ..
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_v01_Vars
~~~
Once this is done you now need to run launch_default_jobs.py but with a few changes. 
~~~
vim scripts/launch_default_jobs.py
~~~
> CHANGE Variables to run the baseline FullRun2 (~L13)
>  >     version = "v01_Vars"        (L13)
>   >     doPostFit = True           (L15)

>   >     channels = ["0"]           (L46)
>   >     MCTypes = ["mc16ade"]      (L48)

>   >     runPulls = False           (L62)
>   >     runP0 = False              (L66)
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-Inputs
~~~
