# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document concerns the addition of plots into the group internal note. # 

## Adding Plots to the Internal Note ##

## Last Edited: 11-10-2019

-------------------------------------------------------------------------------
There will come a time where you will have produced plots and want to put them in a paper. The best way to make that paper
is in a Late$\chi$ environment, but if several people will want to add and edit things then the best thing to do is to create
a git repo such that changes and edits can be stored. You download the repo, add your stuff, build to make sure your changes don't break things and upload. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git

git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics-office/HIGG/ANA-HIGG-2018-51/ANA-HIGG-2018-51-INT1.git
cd ANA-HIGG-2018-51-INT1

~~~
You will want to add your plots to the figures directory, and then you can just include your images into the plots like you would in a normal lateX format.

For adding the fit plots into the note, you will want a directory structure that looks like this. 
~~~
mkdir 0lep-stxs-cba-fit 0lep-stxs-mva-fit
cd 0lep-stxs-mva-fit
mkdir data-vs-asimov  fcc  postfit  ranking
mkdir fcc/AsimovFit_conditionnal_mu1  fcc/GlobalFit_conditionnal_mu1
mkdir 2jet  3jet
mkdir 2jet/CRLow 2jet/CRHigh 2jet/SR 3jet/CRLow 3jet/CRHigh 3jet/SR
cd ../0lep-stxs-cba-fit
mkdir data-vs-asimov  fcc  postfit  ranking
mkdir fcc/AsimovFit_conditionnal_mu1  fcc/GlobalFit_conditionnal_mu1
mkdir 2jet  3jet
mkdir 2jet/CRLow 2jet/CRHigh 2jet/SR 3jet/CRLow 3jet/CRHigh 3jet/SR
~~~
For the MVA fit. These copies represent most of the plots you would want, minus the mBB MVA plots which are not present at the moment. The breakdowns are copied over by hand from it's text format.
~~~
cd 0lep-stxs-mva-fit

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Norm.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_FloatNorm.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Zjets.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Wjets.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Top.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Diboson.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_LUMI.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_MET.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_BTag.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Jet.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_MVA/NP_Lepton.pdf data-vs-asimov/

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull/plots/fcc/AsimovFit_conditionnal_mu1/NP_VH.pdf fcc/AsimovFit_conditionnal_mu1/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull/plots/fcc/GlobalFit_conditionnal_mu1/corr_HighCorr.pdf fcc/GlobalFit_conditionnal_mu1/

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA/pdf-files/pulls_SigXsecOverSM_125.pdf ranking/

cd postfit
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMax250_BMin150_Y6051_DSR_T2_L0_distmva_J2_GlobalFit_conditionnal_mu1log.pdf 2jet/SR
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMin250_Y6051_DSR_T2_L0_distmva_J2_GlobalFit_conditionnal_mu1log.pdf 2jet/SR
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMax250_BMin150_Y6051_DSR_T2_L0_distmva_J3_GlobalFit_conditionnal_mu1log.pdf 3jet/SR
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMin250_Y6051_DSR_T2_L0_distmva_J3_GlobalFit_conditionnal_mu1log.pdf 3jet/SR

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMax250_BMin150_Y6051_DCRLow_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRLow
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMin250_Y6051_DCRLow_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRLow
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMax250_BMin150_Y6051_DCRLow_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRLow
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMin250_Y6051_DCRLow_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRLow

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMax250_BMin150_Y6051_DCRHigh_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRHigh
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMin250_Y6051_DCRHigh_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRHigh
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMax250_BMin150_Y6051_DCRHigh_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRHigh
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-MVA-pull_2/plots/postfit/Region_BMin250_Y6051_DCRHigh_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRHigh
~~~
The CBA plots to be targetted will therefore be largely the same, though the names may differ slightly.
~~~
cd ../../0lep-stxs-cba-fit

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Norm.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_FloatNorm.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Zjets.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Wjets.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Top.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Diboson.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_LUMI.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_MET.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_BTag.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_NJet.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Jet.pdf data-vs-asimov/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/pullComparisons_CBA/NP_Lepton.pdf data-vs-asimov/

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/fcc/AsimovFit_conditionnal_mu1/NP_VH.pdf fcc/AsimovFit_conditionnal_mu1/
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/fcc/GlobalFit_conditionnal_mu1/corr_HighCorr.pdf fcc/GlobalFit_conditionnal_mu1/

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA/pdf-files/pulls_SigXsecOverSM_125.pdf ranking/

cd postfit
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMax250_BMin150_Y6051_DSR_T2_L0_distmBB_J2_GlobalFit_conditionnal_mu1.pdf 2jet/SR
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMax250_BMin150_Y6051_DSR_T2_L0_distmBB_J3_GlobalFit_conditionnal_mu1.pdf 3jet/SR
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMin250_Y6051_DSR_T2_L0_distmBB_J2_GlobalFit_conditionnal_mu1.pdf 2jet/SR
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMin250_Y6051_DSR_T2_L0_distmBB_J3_GlobalFit_conditionnal_mu1.pdf 3jet/SR

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMax250_BMin150_Y6051_DCRLow_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRLow
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMin250_Y6051_DCRLow_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRLow
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMax250_BMin150_Y6051_DCRLow_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRLow
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMin250_Y6051_DCRLow_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRLow

cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMax250_BMin150_Y6051_DCRHigh_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRHigh
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMin250_Y6051_DCRHigh_T2_L0_distMET_J2_GlobalFit_conditionnal_mu1.pdf 2jet/CRHigh
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMax250_BMin150_Y6051_DCRHigh_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRHigh
cp /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb/output/140ifb-0L-ade-STXS-baseline-CBA-mBBpull/plots/postfit/Region_BMin250_Y6051_DCRHigh_T2_L0_distMET_J3_GlobalFit_conditionnal_mu1.pdf 3jet/CRHigh
