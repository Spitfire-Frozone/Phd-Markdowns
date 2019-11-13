# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 12-11-2019
-------------------------------------------------------------------------------
# Investigating B-tagging
Last time we played with the fit, some of the b-tagging systematics were being pulled in wierd ways. We will now perform many variations of the 0L fit in an attempt to better understand it, and to see what the issue is with the b-tagging parts of the fit. This also helps understand the fit problems with psuedo-continuous b-tagging (PCBT) as they are expected to be related.
To do this we will perform decorrelations in the b-tagging systematic variables in the:
- number of jets, 
- ptV region 
- in both at the same time
- processes (but this may come after some of the general fits)


In addition to this we will also do some general fits
- Merged ptV Fit
- CR+SR merged Fit
- 1-bin-in-all-SR Fit 
- Only 2jet catagory Fit 
- ad vs e

## Investigating B-tagging Fixed Cut Systematics
First we will want to get a fresh copy of the fit.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git
mv WSMaker_VHbb WSMaker_VHbb_Btagging
cd WSMaker_VHbb_Btagging
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..
~~~
The inputs have already been split, and these are special ones that we'll require
>    /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-10-03/ms1toms2/
~~~
cd inputs
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-10-03/ms1toms2/ .
mv ms1toms2/ SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS
cd ..

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13-L15)
>   >  version = "milestone1_v02_STXS"                                                                                    
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

>    ADD additional debug plots for shape plots(~L72)                                                                
>   >  doplots = True                                                                                                        

Then once you are ready you can run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA
~~~
After this is ready, we will want to run many fits for all of the b-tagging systematics that have been pulled but we need to edit them one at a time. I will show how I do this for one set, though this will need to be repeated for each systematic to be investigated. 
- B_0                                                                                                  
- B_1                                       
- C_0                                  
- Light_0

### Light_0
~~~
vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
>    ADD selective decorrelation for systematic of your choice (~L544)
>   >  if (sysname == "Eigen_Light_0") m_histoSysts.insert( { "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::noSym}.decorr(P::nJet) });
>   >  m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});  -> else m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});                                                                                        

Here T::shape defines the type of the uncertainty. T::shape="shape+norm" T::shapeOnly="shape"

The other two options you need to run for this are then
>    .decorr(P::binMin)                                                                                               
>    .decorr({P::nJet, P::binMin})   

Then you need to rebuild and re-run as normal
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py Light_0_nJetPtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.Light_0_nJetPtVDeco_fullRes_VHbb_Light_0_nJetPtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/Light_0_nJetPtVDeco

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA Light_0_nJetPtVDeco -n -a 5 -l Nominal L0_nJPtVDecorr 
mv output/pullComparisons output/Nominal_vs_Light_0ptVnJ

~~~
>    python scripts/launch_default_jobs.py Light_0_nJetDeco
>    mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.Light_0_nJetDeco_fullRes_VHbb_Light_0_nJetDeco_0_mc16ade_Systs_mva_STXS_Fit>    Scheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/Light_0_nJetDeco

>    python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA Light_0_nJetDeco -n -a 5 -l Nominal L0_nJDecorr 
>    mv output/pullComparisons output/Nominal_vs_Light_0nJ


>    python scripts/launch_default_jobs.py Light_0_PtVDeco
>    mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.Light_0_PtVDeco_fullRes_VHbb_Light_0_PtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/Light_0_PtVDeco

>    python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA Light_0_PtVDeco -n -a 5 -l Nominal L0_PtVDecorr 
>    mv output/pullComparisons output/Nominal_vs_Light_0PtV

### C_0
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py C_0_nJetDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.C_0_nJetDeco_fullRes_VHbb_C_0_nJetDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/C_0_nJetDeco

python scripts/launch_default_jobs.py C_0_PtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.C_0_PtVDeco_fullRes_VHbb_C_0_PtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/C_0_PtVDeco

python scripts/launch_default_jobs.py C_0_nJetPtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.C_0_nJetPtVDeco_fullRes_VHbb_C_0_nJetPtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/C_0_nJetPtVDeco


python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA C_0_nJetPtVDeco -n -a 5 -l Nominal C0_nJPtVDecorr 
mv output/pullComparisons output/Nominal_vs_C_0ptVnJ

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA C_0_PtVDeco -n -a 5 -l Nominal C0_PtVDecorr 
mv output/pullComparisons output/Nominal_vs_C_0ptV
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA C_0_nJetDeco -n -a 5 -l Nominal C0_nJDecorr 
mv output/pullComparisons output/Nominal_vs_C_0nJ
~~~
### B_0
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py B_0_nJetDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.B_0_nJetDeco_fullRes_VHbb_B_0_nJetDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/B_0_nJetDeco

python scripts/launch_default_jobs.py B_0_PtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.B_0_PtVDeco_fullRes_VHbb_B_0_PtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/B_0_PtVDeco

python scripts/launch_default_jobs.py B_0_nJetPtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.B_0_nJetPtVDeco_fullRes_VHbb_B_0_nJetPtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/B_0_nJetPtVDeco



python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA B_0_nJetPtVDeco -n -a 5 -l Nominal B0_nJPtVDecorr 
mv output/pullComparisons output/Nominal_vs_B_0ptVnJ

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA B_0_PtVDeco -n -a 5 -l Nominal B0_PtVDecorr 
mv output/pullComparisons output/Nominal_vs_B_0ptV
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA B_0_nJetDeco -n -a 5 -l Nominal B0_nJDecorr 
mv output/pullComparisons output/Nominal_vs_B_0nJ
~~~
### B_1
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py B_1_nJetDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.B_1_nJetDeco_fullRes_VHbb_B_1_nJetDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/B_1_nJetDeco

python scripts/launch_default_jobs.py B_1_PtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.B_1_PtVDeco_fullRes_VHbb_B_1_PtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/B_1_PtVDeco

python scripts/launch_default_jobs.py B_1_nJetPtVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.B_1_nJetPtVDeco_fullRes_VHbb_B_1_nJetPtVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/B_1_nJetPtVDeco



python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA B_1_nJetPtVDeco -n -a 5 -l Nominal B1_nJPtVDecorr 
mv output/pullComparisons output/Nominal_vs_B_1ptVnJ

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA B_1_PtVDeco -n -a 5 -l Nominal B1_PtVDecorr 
mv output/pullComparisons output/Nominal_vs_B_1ptV
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA B_1_nJetDeco -n -a 5 -l Nominal B1_nJDecorr 
mv output/pullComparisons output/Nominal_vs_B_1nJ
~~~
## Generic Fit Variations
Another tool we can use to understand the 'erronious' pulls is to warp the fit in different ways and see how the pulls/shapes change. 

### Merged PtV Fit
This one is rather simple because there is a flag in the job launcher steering file, but you do have to change the inputs as well.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging

vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
>    REMOVE selective decorrelation for systematic of your choice (~L544)
>   >  if (sysname == "Eigen_Light_0") m_histoSysts.insert( { "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::noSym}.decorr(P::nJet) });
>   > else m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig}); -> m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig}); 

~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flag to run over merged ptV inputs (~L13)
>   >  version = "milestone1_v02_0L_STXS_MergedPtV"

>    CHANGE flag to switch on merged PtV run (~L21)
>   >  mrgPtV = True 

>    CHANGE flag to switch on merged PtV run (~L68)
>   >  runP0 = True
~~~
cd inputs
mkdir SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedPtV
cp -r SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton* SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedPtV/
cd SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedPtV/

hadd 13TeV_ZeroLepton_2tag2jet_150ptv_SR_mva.root 13TeV_ZeroLepton_2tag2jet_*_CRHigh_mva.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_SR_mvadiboson.root 13TeV_ZeroLepton_2tag2jet_*_SR_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRHigh_mBB.root 13TeV_ZeroLepton_2tag2jet_*_CRHigh_mBB.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRHigh_MET.root 13TeV_ZeroLepton_2tag2jet_*_CRHigh_MET.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRHigh_mvadiboson.root 13TeV_ZeroLepton_2tag2jet_*_CRHigh_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRHigh_mva.root 13TeV_ZeroLepton_2tag2jet_*_CRHigh_mva.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRLow_mBB.root 13TeV_ZeroLepton_2tag2jet_*_CRLow_mBB.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRLow_MET.root 13TeV_ZeroLepton_2tag2jet_*_CRLow_MET.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRLow_mvadiboson.root 13TeV_ZeroLepton_2tag2jet_*_CRLow_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_CRLow_mva.root 13TeV_ZeroLepton_2tag2jet_*_CRLow_mva.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_SR_mBB.root 13TeV_ZeroLepton_2tag2jet_*_SR_mBB.root
hadd 13TeV_ZeroLepton_2tag2jet_150ptv_SR_MET.root 13TeV_ZeroLepton_2tag2jet_*_SR_MET.root  
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_SR_mva.root 13TeV_ZeroLepton_2tag3jet_*_SR_mva.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_SR_mvadiboson.root 13TeV_ZeroLepton_2tag3jet_*_SR_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRHigh_mBB.root 13TeV_ZeroLepton_2tag3jet_*_CRHigh_mBB.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRHigh_MET.root 13TeV_ZeroLepton_2tag3jet_*_CRHigh_MET.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRHigh_mvadiboson.root 13TeV_ZeroLepton_2tag3jet_*_CRHigh_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRHigh_mva.root 13TeV_ZeroLepton_2tag3jet_*_CRHigh_mva.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRLow_mBB.root 13TeV_ZeroLepton_2tag3jet_*_CRLow_mBB.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRLow_MET.root 13TeV_ZeroLepton_2tag3jet_*_CRLow_MET.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRLow_mvadiboson.root 13TeV_ZeroLepton_2tag3jet_*_CRLow_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_CRLow_mva.root 13TeV_ZeroLepton_2tag3jet_*_CRLow_mva.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_SR_mBB.root 13TeV_ZeroLepton_2tag3jet_*_SR_mBB.root
hadd 13TeV_ZeroLepton_2tag3jet_150ptv_SR_MET.root 13TeV_ZeroLepton_2tag3jet_*_SR_MET.root

rm *_250ptv_*
cd ../..

source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ptVMerge

mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedPtV.140ifb-0L-ade-STXS-baseline-MVA-ptVMerge_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ptVMerge_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ptVMerge

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ptVMerge -n -a 5 -l Nominal Merged_PtV
mv output/pullComparisons output/Nominal_vs_MergedPtV
~~~
### Merged CR+SR Fit

This one is rather simple as well because there is a flag in the job launcher steering file, but the inputs have to change as well.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flag to run over merged ptV inputs (~L13)
>   >  version = "milestone1_v02_0L_STXS_MergedSR"

>    CHANGE flag to switch on merged PtV run (~L21)
>   >  mrgPtV = False 

>    CHANGE flag to switch off the new CR's (~L34)
>   >  doNewRegions = False

>    CHANGE flag to switch on merged PtV run (~L68)
>   >  runP0 = True

~~~
cd inputs
mkdir SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedSR
cd SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedSR/

hadd 13TeV_ZeroLepton_2tag2jet_150_250ptv_SR_mva.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_150_250ptv_*_mva.root  
hadd 13TeV_ZeroLepton_2tag2jet_150_250ptv_SR_mvadiboson.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_150_250ptv_*_mvadiboson.root 
hadd 13TeV_ZeroLepton_2tag2jet_150_250ptv_SR_mBB.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_150_250ptv_*_mBB.root 
hadd 13TeV_ZeroLepton_2tag2jet_150_250ptv_SR_MET.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_150_250ptv_*_MET.root 
hadd 13TeV_ZeroLepton_2tag2jet_250ptv_SR_mva.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_250ptv_*_mva.root
hadd 13TeV_ZeroLepton_2tag2jet_250ptv_SR_mvadiboson.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_250ptv_*_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag2jet_250ptv_SR_mBB.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_250ptv_*_mBB.root 
hadd 13TeV_ZeroLepton_2tag2jet_250ptv_SR_MET.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag2jet_250ptv_*_MET.root

hadd 13TeV_ZeroLepton_2tag3jet_150_250ptv_SR_mva.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_150_250ptv_*_mva.root  
hadd 13TeV_ZeroLepton_2tag3jet_150_250ptv_SR_mvadiboson.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_150_250ptv_*_mvadiboson.root 
hadd 13TeV_ZeroLepton_2tag3jet_150_250ptv_SR_mBB.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_150_250ptv_*_mBB.root 
hadd 13TeV_ZeroLepton_2tag3jet_150_250ptv_SR_MET.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_150_250ptv_*_MET.root 
hadd 13TeV_ZeroLepton_2tag3jet_250ptv_SR_mva.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_250ptv_*_mva.root
hadd 13TeV_ZeroLepton_2tag3jet_250ptv_SR_mvadiboson.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_250ptv_*_mvadiboson.root
hadd 13TeV_ZeroLepton_2tag3jet_250ptv_SR_mBB.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_250ptv_*_mBB.root 
hadd 13TeV_ZeroLepton_2tag3jet_250ptv_SR_MET.root ../SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS/13TeV_ZeroLepton_2tag3jet_250ptv_*_MET.root

cd ../..
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-MergeSR

mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedSR.140ifb-0L-ade-STXS-baseline-MVA-MergeSR_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-MergeSR_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-MergeSR

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-MergeSR -n -a 5 -l Nominal Merged_SR
mv output/pullComparisons output/Nominal_vs_MergedSR
~~~

### Splitting Fit by process. 
The next test we want to do is to split the B-tagging systematics by process (i.e Zbb, Wbb, Zcl, Wcl, Zl, Wl, Zbl e.t.c) and see how the pulls are affected. Splitting them totally into each process might reduce the statistics too much so a better thing to do might be to split them into similar processes. Vbb (Zbb+Wbb), Vc (Zxx,Wxx where at least one of the x's is a c) and Vl (Zxx,Wxx where at least one of the x's is a l, but and the other is NOT a c).
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13, L21, L34)
>   >  version = "milestone1_v02_STXS"  

>   >  mrgPtV = False    

>   >  doNewRegions = True
~~~
vim src/systematiclistsbuilder_vhbbrun2.cpp
~~~
>    ADD decorrelation for flavour tagging (~L534)
>   >    SysConfig FTag_Process_Decorr_Config = noSmoothConfig;
>   >      FTag_Process_Decorr_Config.decorrFun([](const PropertiesSet& pset, const Sample& s) {                 
>   >                                     if( s.hasKW("Diboson") || s.hasKW("Higgs") ) return "_signal_VV";             
>   >                                     else if( s.name() == "Zbb" || s.name() == "Wbb" ) return "_Vbb";                  
>   >                                     else if( s.name() == "Zbc" || s.name() == "Wbc" || s.name() == "Wcl" || s.name() == "Zcl" || s.name() == "Zcc" || s.name() == "Wcc" ) return "_Vc";                             
>   >                                     else if( s.name() == "Zbl" || s.name() == "Wbl" || s.name() == "Wl" || s.name() == "Zl" ) return "_Vl";                                                                     
>   >                                     else if( s.name() == "ttbar" ) return "_ttbar";                      
>   >                                     else return "";                                                        
>   >                                     });                                                                                 

>    ADD selective decorrelation for problematic flavour tagging pulls (~L544)
>   >    if (sysname == "Eigen_Light_0" || sysname == "Eigen_C_0" || sysname == "Eigen_B_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , FTag_Process_Decorr_Config});                                                                     
>   >     m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});  -> else m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});                                                                                                           

~~~
source setup.sh
cd build && rm -rf *
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ProcessDecorr
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.140ifb-0L-ade-STXS-baseline-MVA-ProcessDecorr_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ProcessDecorr_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ProcessDecorr

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ProcessDecorr -n -a 5 -l Nominal bcl_proc_decorr
mv output/pullComparisons output/Nominal_vs_BCLProcDecorr
~~~
### One Bin in all regions
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging


vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13, L21, L34)
>   >  version = "milestone1_v02_STXS"  

>   >  mrgPtV = False    

>   >  doNewRegions = True

>    ADD shapes plots just in case they are needed (~L72)
>   >  doplots = True
~~~

vim src/binning_vhbbrun2.cpp
~~~
>    CHANGE Binning to be 1 in SR and CRs
>   >      //Testing Single bin in all regions
>   >      if (doNewRegions && (c(Property::descr).Contains("CR")) ){
>   >        return oneBin(c);
>   >      }
>   >      if (doNewRegions && (c(Property::descr).Contains("SR")) ){
>   >        return oneBin(c);
>   >      }
~~~
source setup.sh
cd build && rm -rf *
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-OneBinAll
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_STXS.140ifb-0L-ade-STXS-baseline-MVA-OneBinAll_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-OneBinAll_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-OneBinAll

~~~
We also want to run this with the merged PtV. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flag to run over merged ptV inputs (~L13)
>   >  version = "milestone1_v02_0L_STXS_MergedPtV"

>    CHANGE Global run conditions (~L21)
>   >  mrgPtV = True    
~~~
source setup.sh
cd build && rm -rf *
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-OneBinAll-ptVMerge
mv output/SMVHVZ_2019_MVA_mc16ade_milestone1_v02_0L_STXS_MergedPtV.140ifb-0L-ade-STXS-baseline-MVA-OneBinAll-ptVMerge_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-OneBinAll-ptVMerge_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-OneBinAll-ptVMerge
