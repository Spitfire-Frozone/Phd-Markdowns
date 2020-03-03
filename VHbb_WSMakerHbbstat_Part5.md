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

# Running MC16 year comparison for Diboson Analysis
~~~
vim launch_default_[...].py
commands.extend(["-u", doExp+",{MassPoint},15"])
+breakdowns
doExp=1

work
cd WSMaker_VHbb_Mar2020
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ad-STXS-baseline-MVA

python scripts/launch_default_jobs.py 140ifb-2L-e-STXS-baseline-MVA
~~~
To get the ade, you only have to copy from what's already done. 
~~~
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-28_VHunblinded/VZ-MVA/2lep
cp -r SMVHVZ_2019_MVA_mc16ade_v06_STXS.2lep_1POI_VZ_fullRes_VHbb_2lep_1POI_VZ_2_mc16ade_Systs_mvadiboson_STXS_FitScheme_1 /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output


work
cd WSMaker_VHbb_Mar2020/output

mv SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS_postprocessed.140ifb-2L-ad-STXS-baseline-MVA_fullRes_VHbb_140ifb-2L-ad-STXS-baseline-MVA_2_mc16ad_Systs_mvadiboson_STXS_FitScheme_1 140ifb-2L-ad-STXS-baseline-VV

mv SMVHVZ_2019_MVA_mc16e_2L_v03_STXS_postprocessed.140ifb-2L-e-STXS-baseline-MVA_fullRes_VHbb_140ifb-2L-e-STXS-baseline-MVA_2_mc16e_Systs_mvadiboson_STXS_FitScheme_1 140ifb-2L-e-STXS-baseline-VV

mv SMVHVZ_2019_MVA_mc16ade_v06_STXS.2lep_1POI_VZ_fullRes_VHbb_2lep_1POI_VZ_2_mc16ade_Systs_mvadiboson_STXS_FitScheme_1 140ifb-2L-ade-STXS-baseline-VV

cd ..

python WSMakerCore/scripts/comparePulls.py -w 140ifb-2L-ade-STXS-baseline-VV 140ifb-2L-ad-STXS-baseline-VV 140ifb-2L-e-STXS-baseline-VV -n -a 5 -l ade ad e
mv output/pullComparisons output/pullComp_2L_VV_ade_vs_ad_vs_e


To get the fitted mu's for ade/ad/e you need to search for SigXsecOverSM in the three fccs/output_0.log
~~~~
# Running MC16 year comparison for VH-MVA Analysis
~~~
work
cd WSMaker_VHbb_Mar2020
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-e-STXS-baseline-MVA

python scripts/launch_default_jobs.py 140ifb-2L-ad-STXS-baseline-MVA

~~~
To get the ade
~~~
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-28_VHunblinded/VH-MVA/L2 . /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output
mv L2 140ifb-2L-ade-STXS-baseline-MVA

work
cd WSMaker_VHbb_Mar2020/output

mv SMVHVZ_2019_MVA_mc16ad_2L_v03_STXS_postprocessed.140ifb-2L-ad-STXS-baseline-MVA_fullRes_VHbb_140ifb-2L-ad-STXS-baseline-MVA_2_mc16ad_Systs_mva_STXS_FitScheme_1 140ifb-2L-ad-STXS-baseline-MVA
mv SMVHVZ_2019_MVA_mc16e_2L_v03_STXS_postprocessed.140ifb-2L-e-STXS-baseline-MVA_fullRes_VHbb_140ifb-2L-e-STXS-baseline-MVA_2_mc16e_Systs_mva_STXS_FitScheme_1 140ifb-2L-e-STXS-baseline-MVA

cd ..

python WSMakerCore/scripts/comparePulls.py -w 140ifb-2L-ade-STXS-baseline-MVA 140ifb-2L-ad-STXS-baseline-MVA 140ifb-2L-e-STXS-baseline-MVA -n -a 5 -l ade ad e
mv output/pullComparisons output/pullComp_2L_MVA_ade_vs_ad_vs_e
~~~
# Extraction of mu for MC16 year comparison for VH-MVA/VV-MVA Analyses
~~~
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-ad-STXS-baseline-MVA
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-e-STXS-baseline-MVA
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-ad-STXS-baseline-VV
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-2L-e-STXS-baseline-VV
~~~
vim scripts/NicePlot_SimpleMu.py

L177
    if mode=="7":
        value=[
            ["Comb."         ,0, 1.0, 0.0, 0.0, 0.0, 0.0, "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ade-STXS-baseline-VV/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt",1],
            ["2017  "        ,1, 1.0, 0.0, 0.0, 0.0, 0.0, "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-e-STXS-baseline-VV/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt",1],
            ["2015-2016  "   ,2, 1.0, 0.0, 0.0, 0.0, 0.0, "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ad-STXS-baseline-VV/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt",1],
        ] 


change path to breakdown folders for mc6ade vs mc16ad vs mc16e plot (mode 7) L316
> "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ade-STXS-baseline-MVA/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt"
> /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-e-STXS-baseline-MVA/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt
> /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Mar2020/output/140ifb-2L-ad-STXS-baseline-MVA/plots/breakdown/muHatTable_mode15_Asimov0_SigXsecOverSM.txt
~~~
python scripts/NicePlot_SimpleMu.py 7 1
~~~
Create a webpage of all of the nice plots. 
~~~
cd output
python "../WSMakerCore/macros/webpage/createHtmlOverview.py"
~~~
