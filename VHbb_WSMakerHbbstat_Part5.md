# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 03-03-2020
-------------------------------------------------------------------------------
# Unblinded Fits on latest inputs

First you will need to obtain a fresh copy of the WSmaker directory, and get the right development branch/tag
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git

mv WSMaker_VHbb WSMaker_VHbb_Mar2020
cd WSMaker_VHbb_Mar2020
git checkout tags/00-03-04-fix 
source setup.sh
cd build
cmake ..
make -j8
cd ..
~~~

# Post-processing New MC split inputs

The raw inputs for MC16ade/ad/a/d/e can be found here:
> /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/TwoLep/r32-15_postMS2_20191124/v9.Y/FitVariables/

The first port of call then is to create some inputConfigs for these and split some inputs.
~~~
cd inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_2L_v03_STXS.txt
~~~
> ADD Location of MC16ade inputs and list of present variables
>   >  CoreRegions                                                                                                             
>   >  TwoLepton TwoLep/r32-15_postMS2_20191124/v9.Y/FitVariables/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.Kyoto.r32-15.CSFix.v9.X.imcomplete_ttbar.hacked.root mva,pTV,mBB,mvadiboson                                                             

It is important to note that since this is the default, post-processed split inputs are very likely to already exist. 
~~~
vim SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS.txt
~~~
> ADD Location of MC16ad inputs and list of present variables
>   >  CoreRegions                                                                                                             
>   >  TwoLepton TwoLep/r32-15_postMS2_20191124/v9.Y/FitVariables/LimitHistograms.VHbb.2Lep.13TeV.mc16ad.Kyoto.r32-15.CSFix.v9.X.imcomplete_ttbar.hacked.root mva,pTV,mBB,mvadiboson                                                             
~~~
vim SMVHVZ_2019_MVA_mc16e_2L_v03_STXS.txt
~~~
> ADD Location of MC16e inputs and list of present variables
>   >  CoreRegions                                                                                                             
>   >  TwoLepton TwoLep/r32-15_postMS2_20191124/v9.Y/FitVariables/LimitHistograms.VHbb.2Lep.13TeV.mc16e.Kyoto.r32-15.CSFix.v9.X.imcomplete_ttbar.hacked.root mva,pTV,mBB,mvadiboson                                                             

To set up the post-processing, a new version of ROOT is required. 
~~~
lsetup "root 6.18.04-x86_64-centos7-gcc8-opt"

SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_2L_v03_STXS
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16e_2L_v03_STXS
~~~
Then to post-process the inputs you run the following commands. I recommend running these in parallel as they will take several hours.
~~~
python scripts/RemoveNormImpact.py --indir inputs/SMVHVZ_2019_MVA_mc16ade_2L_v03_STXS/ --outdir inputs/SMVHVZ_2019_MVA_mc16ade_2L_v03_STXS_postprocessed/
python scripts/RemoveNormImpact.py --indir inputs/SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS/ --outdir inputs/SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS_postprocessed/
python scripts/RemoveNormImpact.py --indir inputs/SMVHVZ_2019_MVA_mc16e_2L_v03_STXS/ --outdir inputs/SMVHVZ_2019_MVA_mc16e_2L_v03_STXS_postprocessed/
~~~
Then you will want to move the post-processed inputs to somewhere where people can get to them.
~~~
cp SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS_postprocessed/* /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2020Feb28_VHunblinding_SplitByYear/ad/

cp SMVHVZ_2019_MVA_mc16e_2L_v03_STXS_postprocessed/* /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2020Feb28_VHunblinding_SplitByYear/e/
~~~
Once you have the inputs, you are ready to start running jobs.

# Running MC16 year comparison for Diboson Analysis
Firstly ensure that you are unblinded.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020
vim setup.sh
~~~
> CHANGE blinding flag from 1 to 0 (~L6)                                                                                       
>   >  export IS_BLINDED=0                                                                                                      
~~~
vim analysisPlottingConfig.py
~~~
> CHANGE Analysis from VH-MVA to VV-MVA (~L10)                                                                                 
>   >  vh_fit = false                                                                                                         
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020
vim launch_default_jobs.py
~~~
> CHANGE input version and analysis from VV-MVA                                                                             
>   >  version = "2L_v03_STXS_postprocessed"  (~L13)                                                                           
>   >  doDiboson = True                       (~L19)                                                                          

> CHANGE the setup to run on 2L pulls and breakdowns (~L51-68)                                                                 
>   >  channels = ["2"]                                                                                                       
>   >  MCTypes = ["mc16ad"] then ["mc16e"]                                                                                     
>   >  syst_type = [ "Systs" ]                                                                                                 
>   >  doExp = "0"                                                                                                           

>   >  createSimpleWorkspace = True                                                                                           
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = True                                                                                                   

> CHANGE the mass point to make running on the breakdowns faster (~L467 if you run locally: run_on_batch = False) 
>   >  commands.extend(["-u", doExp+",{MassPoint},11"]) -> commands.extend(["-u", doExp+",{MassPoint},15"])                   

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ad-STXS-baseline-VV

python scripts/launch_default_jobs.py 140ifb-2L-e-STXS-baseline-VV
~~~
To get the ade, you only have to copy from what's already done. 
~~~
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-28_VHunblinded/VZ-MVA/2lep
cp -r SMVHVZ_2019_MVA_mc16ade_v06_STXS.2lep_1POI_VZ_fullRes_VHbb_2lep_1POI_VZ_2_mc16ade_Systs_mvadiboson_STXS_FitScheme_1 /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output

~~~
Now you just need to rename the output directories and create the pull plots.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output

mv SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS_postprocessed.140ifb-2L-ad-STXS-baseline-140ifb-2L-ad-STXS-baseline-VV_fullRes_VHbb_140ifb-2L-ad-STXS-baseline-140ifb-2L-ad-STXS-baseline-VV_2_mc16ad_Systs_mvadiboson_STXS_FitScheme_1 140ifb-2L-ad-STXS-baseline-VV
mv SMVHVZ_2019_MVA_mc16e_2L_v03_STXS_postprocessed.140ifb-2L-e-STXS-baseline-140ifb-2L-ad-STXS-baseline-VV_fullRes_VHbb_140ifb-2L-e-STXS-baseline-140ifb-2L-ad-STXS-baseline-VV_2_mc16e_Systs_mvadiboson_STXS_FitScheme_1 140ifb-2L-e-STXS-baseline-VV
mv SMVHVZ_2019_MVA_mc16ade_v06_STXS.2lep_1POI_VZ_fullRes_VHbb_2lep_1POI_VZ_2_mc16ade_Systs_mvadiboson_STXS_FitScheme_1 140ifb-2L-ade-STXS-baseline-VV
cd ..

python WSMakerCore/scripts/comparePulls.py -w 140ifb-2L-ade-STXS-baseline-VV 140ifb-2L-ad-STXS-baseline-VV 140ifb-2L-e-STXS-baseline-VV -n -a 5 -l ade ad e
mv output/pullComparisons output/pullComp_2L_VV_ade_vs_ad_vs_e
~~~~
# Running MC16 year comparison for VH-MVA Analysis
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/
vim analysisPlottingConfig.py
~~~
> CHANGE Analysis from VV-MVA to VH-MVA (~L10)                                                                                 
>   >  vh_fit = true                                                                                                         
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020
vim launch_default_jobs.py
~~~
> CHANGE input version and analysis from VV-MVA                                                                             
>   >  version = "2L_v03_STXS_postprocessed"  (~L13)                                                                           
>   >  doDiboson = False                      (~L19)                                                                          
~~~
cd WSMaker_VHbb_Mar2020
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-2L-ad-STXS-baseline-MVA

python scripts/launch_default_jobs.py 140ifb-2L-e-STXS-baseline-MVA
~~~
To get the ade
~~~
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-28_VHunblinded/VH-MVA/L2 . /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output
~~~
Once again rename the output directories and create the pull plots.
~~~
mv L2 140ifb-2L-ade-STXS-baseline-MVA
mv SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS_postprocessed.140ifb-2L-ad-STXS-baseline-MVA_fullRes_VHbb_140ifb-2L-ad-STXS-baseline-MVA_2_mc16ad_Systs_mva_STXS_FitScheme_1 140ifb-2L-ad-STXS-baseline-MVA
mv SMVHVZ_2019_MVA_mc16e_2L_v03_STXS_postprocessed.140ifb-2L-e-STXS-baseline-MVA_fullRes_VHbb_140ifb-2L-e-STXS-baseline-MVA_2_mc16e_Systs_mva_STXS_FitScheme_1 140ifb-2L-e-STXS-baseline-MVA
cd ..

python WSMakerCore/scripts/comparePulls.py -w 140ifb-2L-ade-STXS-baseline-MVA 140ifb-2L-ad-STXS-baseline-MVA 140ifb-2L-e-STXS-baseline-MVA -n -a 5 -l ade ad e
mv output/pullComparisons output/pullComp_2L_MVA_ade_vs_ad_vs_e
~~~
# Extraction of mu for MC16 year comparison for VH-MVA/VV-MVA Analyses
Not so difficult this one, all you need to do is to run the breakdown tables and run a script over a series of text files.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/
source setup.sh
cd build
cmake ..
make -j8
cd ..

python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-ad-STXS-baseline-MVA
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-e-STXS-baseline-MVA
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-ad-STXS-baseline-VV
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-e-STXS-baseline-VV

vim scripts/NicePlot_SimpleMu.py
~~~
> CHANGE input path for breakdowns for VV-MVA and VH-MVA mode 7 ade/ad/e
> ~L177                                                                                                                      
>   >  if mode=="7":
>   >          value=[
>   >              ["Comb."         ,0, 1.0, 0.0, 0.0, 0.0, 0.0, "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ade-STXS-baseline-VV/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt",1],
>   >              ["2017  "        ,1, 1.0, 0.0, 0.0, 0.0, 0.0, "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-e-STXS-baseline-VV/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt",1],
>   >              ["2015-2016  "   ,2, 1.0, 0.0, 0.0, 0.0, 0.0, "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ad-STXS-baseline-VV/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt",1],
>   >          ] 

> ~L316                                                                                                                      
>   >  "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ade-STXS-baseline-MVA/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt"
>   >  /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-e-STXS-baseline-MVA/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt
>   >  /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ad-STXS-baseline-MVA/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt
~~~
python scripts/NicePlot_SimpleMu.py 7 1
python scripts/NicePlot_SimpleMu.py 7 0
cp Mu_Results/*.pdf output/
~~~
Alternatively you can extract the final mus from the breakdown tables yourself created in plots/breakdowns or by searching for SigXsecOverSM in the fccs/output_0.log that comes as a result of the pulls.

# Create a webpage of all of the nice plots. 
~~~
cd output
python "../WSMakerCore/macros/webpage/createHtmlOverview.py"

cd /afs/cern.ch/user/d/dspiteri/www/
mkdir VHUnblinding && cd VHUnblinding
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/www/* .
~~~
Link available at https://dspiteri.web.cern.ch/dspiteri/VHUnblinding/
