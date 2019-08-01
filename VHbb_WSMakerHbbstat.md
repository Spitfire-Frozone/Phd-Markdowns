# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## "VHbb WSMaker and Hbb Stat" ##
===============================================================================
## Last Edited: 23-07-2019

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

>  >     channels = ["0"]
>  >     MCTypes = ["mc16ad"]
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
>   >     version = "v01"

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

## Testing input sample completeness
If there are inconsistencies with you rimputs when you compare them in different regions, the first thing to test is that all the samples are present and fully correct. Since the inputs are generated centrally, if there is a problem with them they will nee dot be regenerated. Luckily LUca Ambroz has some code that will check just that.  
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
git clone ssh://git@gitlab.cern.ch:7999/luambroz/analysis_plotting_macro.git
vim analysis_plotting_macro/check_signal_0L_a_d_e.py
~~~
> CHANGE the binnings of the high ptv region (~L17)
>   >      BDT_bins["2"] = [0, 27, 106, 214, 361, 475, 548, 600, 641, 679, 712, 742, 771, 802, 839, 1002]  -> BDT_bins["2"] = [0, 23, 95, 182, 300, 406, 472, 516, 555, 593, 628, 659, 689, 719, 754, 1002]
>   >      BDT_bins["3"] = [0, 99, 184, 277, 370, 450, 524, 584, 635, 679, 718, 753, 787, 821, 861, 1002] -> BDT_bins["3"] = [0, 81, 157, 248, 338, 412, 481, 539, 586, 627, 665, 699, 732, 767, 808, 1002]

> ADD binnings of the extreme ptv region (~L20)
>   >      BDT_bins["4"] = [1, 97, 268, 524, 646, 715, 759, 792, 1002]
>   >      BDT_bins["6"] = [0, 192, 326, 545, 697, 774, 823, 867, 1002]
 
> CHANGE the names of the variables names in the list and the target files (~L122 - L125) but COMMENT OUT
>   >     var_names = ["njets", "NFwdJets", "nCentralJetsNoBtag", "mva", "MET", "mBB", "dRBB", "pTB1", "pTB2", "ActualMu"] ->   var_names = ["Njets", "dEtaBB", "mva", "MET", "METSig", "mBB" ,"mBBJ", "dRBB", "pTB1", "pTB2", "ActualMu"] 
>   >     file_a = ROOT.TFile("../hist_a.root") -> file_a = ROOT.TFile("/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16a.Oxford.r32-15varInputs.root")
>   >     file_d = ROOT.TFile("../hist_d.root") -> file_d = ROOT.TFile("/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16d.Oxford.r32-15varInputs.root")  
>   >     file_e = ROOT.TFile("../hist_e.root") -> file_e = ROOT.TFile("/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Oxford.r32-15varInputs.root")

> ADD region variable to accesss more binning regimes (~L178)
>   >     if "2jet" in full_name: #This section will identify the analysis region such that the mva binning could be obtained
>   >         if "150_" in full_name:
>   >             region = "2"
>   >         else:  
>   >             region = "4"
>   >     else:
>   >         if "150_" in full_name:
>   >             region = "3"
>   >         else:
>   >             region = "6"   

> CHANGE instances of 'jet' in BDT remapping functino to 'region' (~L190)
>   >     remapBDTHisto(h_a, BDT_bins[jet]) -> h_a = remapBDTHisto(h_a, BDT_bins[region])
>   >     remapBDTHisto(h_d, BDT_bins[jet]) -> h_d = remapBDTHisto(h_d, BDT_bins[region])
>   >     remapBDTHisto(h_e, BDT_bins[jet]) -> h_e = remapBDTHisto(h_e, BDT_bins[region])
>   >     remapBDTHisto(hICHEP_a, BDT_bins[jet]) -> hICHEP_a = remapBDTHisto(hICHEP_a, BDT_bins[region])
>   >     remapBDTHisto(hICHEP_d, BDT_bins[jet])-> hICHEP_d = remapBDTHisto(hICHEP_d, BDT_bins[region])

> ADD the names of the variables names in the list and the target files for non-STXS samples (L127 - L130)
>   >     var_names = ["mva", "mvadiboson", "mBBMVA"]
>   >     file_a = ROOT.TFile("/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/v5/Non-STXS/LimitHistograms.VHbb.0Lep.13TeV.mc16a.Oxford.r32-15v5.root")
>   >     file_d = ROOT.TFile("/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/v5/Non-STXS/LimitHistograms.VHbb.0Lep.13TeV.mc16d.Oxford.r32-15v5.root"
>   >     file_e = ROOT.TFile("/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/v5/Non-STXS/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Oxford.r32-15v5.root"

> COMMENT OUT the names of the target ICHEP files
>   >     fileICHEP_a = ROOT.TFile("/home/ambroz/VHbb/CxAODICHEP_Tag/CxAODFramework_tag_r31-16_1/run/hist_a.root") (~L132)
>   >     fileICHEP_d = ROOT.TFile("/home/ambroz/VHbb/CxAODICHEP_Tag/CxAODFramework_tag_r31-16_1/run/hist_d.root") (~L133)
>   >     hICHEP_a = fileICHEP_a.Get(full_name) (~L143)
>   >     hICHEP_d = fileICHEP_d.Get(full_name) (~L144)
>   >     integralICHEP_a = hICHEP_a.Integral(0,-1) (~L154)
>   >     integralICHEP_d = hICHEP_d.Integral(0,-1) (~L155)
>   >     hICHEP_a.Scale(1./hICHEP_a.Integral(0,-1)) (~L174)
>   >     hICHEP_a.Scale(1./hICHEP_a.Integral(0,-1)) (~L175)
>   >     hICHEP_a = remapBDTHisto(hICHEP_a, BDT_bins[region]) (~L193)
>   >     hICHEP_a = remapBDTHisto(hICHEP_a, BDT_bins[region]) (~L194)
>   >       
>   >     resultsICHEP_a = rebinHisto(hICHEP_a, bins_a, x_min, x_max, True) (~L233 - L238)
>   >     hICHEP_a = resultsICHEP_a[0]
>   >     binsICHEP_a = resultsICHEP_a[1]
>   >     resultsICHEP_d = rebinHisto(hICHEP_d, bins_a, x_min, x_max, True)
>   >     hICHEP_d = resultsICHEP_d[0]
>   >     binsICHEP_d = resultsICHEP_d[1]
>   >       
>   >     hICHEP_a.SetLineWidth(2) (~L244)
>   >     hICHEP_d.SetLineWidth(2) (~L245)
>   >     hICHEP_a.SetLineColor(kOrange-3) (~L250)
>   >     hICHEP_d.SetLineColor(kAzure+9) (~L251)
>   >     if integralICHEP_a/integral_a  - 1 > 0: (~L273 - L280)
>   >        leg.AddEntry(hICHEP_a, "mc16a ICHEP (+" + str(round(integralICHEP_a/integral_a - 1,3)*100) + "% mc16a)", "lp")
>   >     else:
>   >        leg.AddEntry(hICHEP_a, "mc16a ICHEP (" + str(round(integralICHEP_a/integral_a - 1,3)*100) + "% mc16a)", "lp")
>   >     if integralICHEP_d/integral_a  - 1 > 0:
>   >        leg.AddEntry(hICHEP_d, "mc16d ICHEP (+" + str(round(integralICHEP_d/integral_a - 1,3)*100) + "% mc16a)", "lp")
>   >     else:
>   >        leg.AddEntry(hICHEP_d, "mc16d ICHEP (" + str(round(integralICHEP_d/integral_a - 1,3)*100) + "% mc16a)", "lp")

>   >     hICHEP_a.Draw("SAME") (~L307)
>   >     hICHEP_d.Draw("SAME") (~L308)

>   >     ratioICHEP_a = hICHEP_a.Clone("ratio") (~L328 - L334)
>   >     ratioICHEP_a.Divide(h_a)
>   >     ratioICHEP_a.Draw("HIST SAME")
>   >     ratioICHEP_d = hICHEP_d.Clone("ratio")
>   >     ratioICHEP_d.Divide(h_a)
>   >     ratioICHEP_d.Draw("HIST SAME")

>   >     del hICHEP_a (~L361)
>   >     del hICHEP_d (~L362)
>   >     del ratioICHEP_a (~L365)
>   >     del ratioICHEP_d (~L366)
>   >     fileICHEP_a.Close() (~L371)
>   >     fileICHEP_d.Close() (~L372)

> CHANGE output path so it runs on a single CPU (~L393)
>   >     p = Pool(8) -> p = Pool(1)

> CHANGE output directory for plot folders 
>   >     if not os.path.exists(sample): -> if not os.path.exists("140ifbade/" + sample): (L345)
>   >     os.makedirs(sample) -> os.makedirs("140ifbade/" + sample) (L346)
>   >     c.SaveAs(sample + "/" + name + ".png") -> c.SaveAs("140ifbade/" + sample + "/" + name + ".png") (L348)
>   >     c.SaveAs(sample + "/" + name + ".pdf") -> c.SaveAs("140ifbade/" + sample + "/" + name + ".pdf") (L349)

> COMMENT OUT variables core running variables (~L394)
>   >     p.map(plotSample, sample_names)

> ADD looping of sample in main function (L397)
>   >     for background_or_signal in sample_names :
>   >        if background_or_signal == " ": #You can insert things you don't want to run over in here.
>   >          continue
>   >        else :
>   >          plotSample(background_or_signal)

> ADD ptv split details
>   >     ptvregs = ["250ptv","150_250ptv"] (L120)

> ADD another nested layer in the loop for different ptv regions. Ensure everything afterwards is indented correctly.      
>   >     for ptv in ptvregs: (L137)

Now you can save (as a different file to not annoy Luca and such that he could use it) and run the macro
~~~
mv analysis_plotting_macro/check_signal_0L_a_d_e.py analysis_plotting_macro/check_samples_140ifb_0L_a_d_e.py 
python analysis_plotting_macro/check_samples_140ifb_0L_a_d_e.py 
~~~

Now for ttbar the analysis has MET filtering so if we want to check that this has been combined properly or is missing, we can run the same code, but change the source root files. 
~~~
mv 140ifbade 140ifbade_full
vim analysis_plotting_macro/check_samples_140ifb_0L_a_d_e.py
~~~
>  COMMENT IN variable-split inputs (~L123)
>  COMMENT OUT normal inputs (~L127)

>  COMMENT IN single sample geneation (~L389)
>   >    plotSample("qqZvvH125") -> plotSample("ttbar")
>  COMMENT OUT looping over all samples for generation of plots (~L390)
~~~
python analysis_plotting_macro/check_samples_140ifb_0L_a_d_e.py 
mv 140ifbade 140ifbade_ttbar
~~~

## Looking at input variables
It would also be good to look at the distribution of input variables that go into the BDT, but as a default only the BDT plots are output. Firstly one needs to add the kinematic variables you want to the related inputConfig file and Split the histograms. Check the main int note (p35) for details
The list of a set of kinematic variables for 
0L| MET,pTB1,pTB2,mBB,dRBB,dPhiVBB,pTJ3,mBBJ,mBBJ3,dEtaBB,METSig,MEff,MEff3                                                   
1L| pTV,MET,pTB1,pTB2,mBB,dRBB,dPhiVBB,dPhiLBmin,pTJ3,mBBJ,mTW,Mtop,dYWH                                                       
2L| pTV,MET,pTB1,pTB2,mBB,dRBB,dPhiVBB,pTJ3,dEtaVBB,mLL,mBBJ3,dEtaBB                                                           

Then one needs to change the location of the files themselves to point to the samples that have all the kinematic variables. Remember the base can be found here. 

/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/

ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15varInputs.root
TwoLep/r32-15_allMVAVariables/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.IOWAUSTC.r32-15_AllMVAVariables.root

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_v01.txt
~~~
> CHANGE all instances of mva,mvadiboson with the relevant set of kinematic variables (for 0L)
>   >      mva,mvadiboson,mBBMVA -> MET,pTB1,pTB2,mBB,dRBB,dPhiVBB,dPhiLBmin,pTJ3,mBBJ,mLL,mBBJ3,dEtaBB,METSig
> CHANGE Absolute path files
>   >      ZeroLep/v5/InputVar/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15varInputs.root
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
>  >     version = "v01_Vars"       (L13)
>  >     doPostFit = True           (L15)

>  >     channels = ["0"]           (L46)
>  >     MCTypes = ["mc16ade"]      (L48)

>  >     runPulls = False           (L62)
>  >     runP0 = False              (L66)
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-Inputs

cd output/
mv *MET 140ifb-0L-ade-Inputs_MET
mv *pTB1 140ifb-0L-ade-Inputs_pTB1
mv *pTB2 140ifb-0L-ade-Inputs_pTB2
mv *mBB 140ifb-0L-ade-Inputs_mBB
mv *dRBB 140ifb-0L-ade-Inputs_dRBB
mv *dPhiVBB 140ifb-0L-ade-Inputs_dPhiVBB
mv *pTJ3 140ifb-0L-ade-Inputs_pTJ3
mv *mBBJ 140ifb-0L-ade-Inputs_mBBJ
mv *dEtaBB 140ifb-0L-ade-Inputs_dEtaBB
mv *METSig 140ifb-0L-ade-Inputs_METSig
mv *MEff 140ifb-0L-ade-Inputs_MEff
mv *MEff3 140ifb-0L-ade-Inputs_MEff3
rm -rf *dPhiLBmin *dYWH *Mtop *mBBJ3 *mTW *dEtaVBB *pTV *mLL

~~~
Once this is done you now need to compare the split inputs against the standard. You are going to have around 18 outputs. To do this we will need to create a new script.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
vim inputVarchecker.sh
~~~
> ADD Workspace comparisons to file (L1)
>	 >     #!/bin/bash
>	 >     WS="140ifb-0L-ade"
>	 >     WS2="140ifb-0L-ade-Inputs_MET"
>	 >     WS3="140ifb-0L-ade-Inputs_pTB1"
>	 >     WS4="140ifb-0L-ade-Inputs_pTB2"
>	 >     WS5="140ifb-0L-ade-Inputs_mBB"
>	 >     WS6="140ifb-0L-ade-Inputs_dRBB"
>	 >     WS7="140ifb-0L-ade-Inputs_dPhiVBB"
>	 >     WS8="140ifb-0L-ade-Inputs_pTJ3"
>	 >     WS9="140ifb-0L-ade-Inputs_mBBJ"
>	 >     WS10="140ifb-0L-ade-Inputs_dEtaBB"
>	 >     WS11="140ifb-0L-ade-Inputs_METSig"
>	 >	  
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS2}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS3}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS4}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS5}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS6}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS7}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS8}
>	 >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS9}
>  >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS10}
>  >     python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f ${WS} ${WS11}
~~~
source ../inputVarchecker.sh
~~~
Here upon running and examining the METSig plots we actually see that we don't like the binning. This can be changed. The re-binning function can be seen in WSMaker/scripts/analysisPlottingConfig.py 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
vim WSMakerCore/scripts/plotMaker.py
~~~
> ADD re-binning of METSig plots by a factor 5 #L1172
>  >     rebin_factor = 5 # This part will add automatic re-binning for all METSig plots. 
>  >     if isinstance(hist, ROOT.TH1) and props and self.cfg.do_rebinning(props):
>  >         if props["L"] == "0" and props["dist"] == "METSig":
>  >             new_hist = hist.Rebin(rebin_factor,"newHist")

Then assuming no changes have been made to launch_default_jobs.py you can just re-run this command. Otherwise you will have to edit the file to the state required to run the individual input variables. 
~~~
python scripts/launch_default_jobs.py 140ifb-0L-ade-Inputs
~~~


# Fit Studies
Now we start getting into the real meat and bones of the fit. Here we shall try to combine some of the parameters in the fit. The main reason is that it is clear 0-lep should not be able to fix Wbb (W+hf) but we do have some information about Z+hf. We have four floating normalisation (NF) systematics: 
- 1 NF for Z+hf 2-jet: norm_Zbb_J2
- 1 NF for Z+hf 3-jet: norm_Zbb_J3
- 1 NP for W+hf in 2-jet: WbbNorm_J2
- 1 NP for W+hf in 3-jet: WbbNorm_J3

and we have no real handle on two of them. The solution to this is to try and get information about Wbb somewhere else and this comes in one of two forms
1) V+hf NFs (Combining W+hf and Z+hf and decorrelate in terms of njets)
2) W+hf fixed and Z+hf floating (Keep the Z+hf as they are bu tget the W+hf stuff from the 1L fit).

## V+hf Normilations Factors
Option 1 consists of removing the individual Z+hf and W+hf markers for the fit and replacing them with a combined thing.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"

vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
> REMOVE 0L from W+hf items (~L144)
>  >    if(hasZeroLep || hasTwoLep) -> if(hasTwoLep)

> CHANGE Structure 0L in Z+hf items (~L163)
>  >    if(hasTwoLep || hasZeroLep) { -> if(hasTwoLep) {
>  >      normFact("Zbb", SysConfig{"Zhf"}.decorr(P::nJet));
> MOVE normalisation for systematics to 2L only if case (L173 -> 166)
>  >    normSys("SysZbbNorm", 0.07, SysConfig{"Zhf"}.applyIn(P::nLep==0).decorr(P::nLep));
>  >    else { -> if(hasOneLep){ (~L168)

> ADD merging of Z and W systematics (~L171)
>  >    //Trialling combinationf Z+hf and W+hf in 0L
>  >    if(hasZeroLep){ 
>  >      normFact("Vbb", SysConfig{{"Zhf", "Whf"}}.decorr(P::nJet));
>  >    }

Once these changes have been made you need to run the fit 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
source setup.sh
cd build && cmake ..
make -j10
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-WZmerge

mv SMVHVZ_2019_MVA_mc16ade_v01.140ifb-0L-ade-WZmerge_fullRes_VHbb_140ifb-0L-ade-WZmerge_0_mc16ade_Systs_mva 140ifb-0L-ade-WZmerge
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade 140ifb-0L-ade-WZmerge -n -a 5 -l WbbZbb Vbb
mv output/pullComparisons output/pullComparisons_WbbandZbb_vs_Vbb
~~~
