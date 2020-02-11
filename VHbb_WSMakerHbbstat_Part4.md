# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 11-02-2020
-------------------------------------------------------------------------------
# Milestone 2 Fits on latest inputs

Now we shall test the 0L standalone fit for new and shiny inputs. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git

mv WSMaker_VHbb WSMaker_VHbb_Milestone2
cd WSMaker_VHbb_Milestone2
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..
~~~
The inputs can be found here.
>    0-lepton :
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.                                                     
>    1-lepton : 
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/OneLep/r32-15_postMS2_20191128/LimitHistograms.VHbb.1Lep.13TeV.mc16ade.UCL.32-15.root                                                     
>    2-lepton : 
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/TwoLep/r32-15_postMS2_20191124/v1/fullSysts/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.Kyoto.r32-15_postMS2_20191124.v1.root                 

The first thing to do is to check the yields against the previous inputs used. 
~~~
vim scripts/InputsCheck_VHbb.py
~~~
>    CHANGE Inputs to compare and the common area where they are kept (~L28-30)                                               
>   >  eos_dir ='/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/'                           
>   >  input_ref  = eos_dir + 'ZeroLep/r32-15_PCBT_20191113/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford-newExt-PCBT.32-15.root'                                                                                                                      
>   >  input_test = eos_dir + 'ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root'

>    CHANGE the Lepton channel to 0 lepton (~L27)                                                                             
>   >  lep = 'L0'                                                                                                             

>    CHANGE the Lepton variable to run over (~L54)                                                                      
>   >  'L0' = 'mva'                                                                                                             
~~~
python scripts/InputsCheck_VHbb.py > Comp32-15_PCBT-20191113_postMS2_20191127.txt
tail -n 7 Comp32-15_PCBT-20191113_postMS2_20191127.txt
~~~
The next step is to split these new inputs
~~~
cd inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_v03_STXS.txt 
~~~
>    ADD new samples
>   >  ZeroLepton ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root mva,MET,mBB,mvadiboson                                                                                 
>   >  OneLepton OneLep/r32-15_postMS2_20191128/LimitHistograms.VHbb.1Lep.13TeV.mc16ade.UCL.32-15.root mva,pTV,mBB,mvadiboson 
>   >  TwoLepton TwoLep/r32-15_postMS2_20191124/v1/fullSysts/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.Kyoto.r32-15_postMS2_20191124.v1.root mva,pTV,mBB,mvadiboson                                                                              
~~~
cd ..
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_v03_STXS
~~~

### Nominal 0L
Now we have to run the nominal standalone 0L fit.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13-L15)                                                                                  
>   >  version = "v03_STXS"                                                                                                  
>   >  GlobalRun = False                                                                                                 
>   >  doPostFit = False                                                                                                

>    CHANGE all do cutbase block to 'false' (~L17-L23)
>   >  doCutBase = False        (~L17)                                                                                        
>   >  do_mbb_plot = False      (~L23)                                                                                        

>    CHANGE setting of new PCBT inputs to be true (~L24)                                                                      
>   >  PCBTInputs = True                                                                                                       

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

>    CHANGE variables to run locally (~L60)                                                                                   
>   >  run_on_batch = False                                                                                             

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)                                                         
>   >  createSimpleWorkspace = True                                                                                
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = False                                                                                              
>   >  runRanks = False                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runP0 = False                                                                                                        
>   >  runToyStudy = False                                                                                                 

>    ADD additional debug plots for shape plots(~L72)                                                                
>   >  doplots = True                                                                                                        

Then once you are ready you can run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-FullVarsPostFit
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA
~~~
Create a website so that everyone can see your plots
~~~
cd output
python "../WSMakerCore/macros/webpage/createHtmlOverview.py"
~~~
Before continuing it is important to check that your results are consistent with other peoples. Here '0LepFit29112019/0LMVAfit/' and 'ThomasFit/' are 0L standalone fit generated by other analysers. Here I run a pull comparison and the pulls should be an exact match.
~~~
source setup.sh
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 0LepFit29112019/0LMVAfit/ ThomasFit/ -n -a 5 -l Dwayne Luca Thomas
mv output/pullComparisons output/DwaynevsLucavsTom0LFit
~~~
### Asimov 0L Fit
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE to construct Asimov dataset using prefit NP's (~L62)                                                               
>   >  doExp=0 -> doExp=1                                                                                                    
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-Asimov
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-Asimov_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-Asimov_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-Asimov
~~~
### PostFit 0L
To break down the output mva into the main variables that go into it.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L15)                                                                                      
>   >  doPostFit = True                                                                                                       

>    CHANGE to construct Asimov dataset using central values from fit to data (~L62)                                           
>   >  doExp=1 -> doExp=0                                                                                                    

>    CHANGE what you want to run in the 0L standalone fit (~L69)                                                          
>   >  runPulls = False                                                                                                 

>    CHANGE postfit variables of interest (~L92)
>   >  vs2tag = ['pTV','MET','pTB1','pTB2','mBB','dRBB','dEtaBB','dPhiVBB','dEtaVBB','MEff','MEff3','dPhiLBmin','mTW','mLL','dYWH','Mtop','pTJ3','mBBJ','mBBJ3','METSig'] -> vs2tag =  ['pTV','MET']

Then once you are ready you can run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-postfitVars_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitVars_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-mBB
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-postfitVars_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitVars_0_mc16ade_Systs_MET_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MET
~~~

### Full 0L Fit
Now we have to run the full nominal standalone 0L fit. This may take some time. 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L15)                                                                                      
>   >  doPostFit = False                                                                                                       

>    CHANGE to construct Asimov dataset using central values from fit to data (~L62)                                           
>   >  doExp=0 -> doExp=1                                                                                                   

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)                                                         
>   >  createSimpleWorkspace = True                                                                                
>   >  runPulls = False                                                                                                       
>   >  runBreakdown = True                                                                                              
>   >  runRanks = True                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runP0 = True                                                                                                        
>   >  runToyStudy = False                                                                                                    

>    ADD additional debug plots for shape plots (~L72)                                                                
>   >  doplots = True                                                                                                        
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-Full
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-Full_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-Full_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-Full
~~~
## Getting the Plots you want
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
~~~
### MVA Vs Asimov Pulls
~~~
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA -n -a 5
mv output/pullComparisons output/pullComp_DataVSAsimov
~~~
### MVA Breakdowns
~~~
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-0L-ade-STXS-baseline-MVA-Full
vim /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2/output/140ifb-0L-ade-STXS-baseline-MVA-Full/plots/breakdown/muHatTable_mode11_Asimov1_SigXsecOverSM.txt 
~~~
### MVA Rankings
~~~
python WSMakerCore/scripts/makeNPrankPlots.py 140ifb-0L-ade-STXS-baseline-MVA-Full
imgcat output/140ifb-0L-ade-STXS-baseline-MVA-Full/pdf-files/pulls_SigXsecOverSM_125.pdf
~~~
### Mbb Plots
~~~
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-mBB
~~~

# Repeating for slightly newer inputs
Post-processed inputs to change the PtV normalisation in 0L and 1L have been produced and they are here:
>     /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec28            

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd inputs
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec28 .
mv 2019Dec28 SMVHVZ_2019_MVA_mc16ade_v04_STXS
~~~
Restore the launch_default_jobs.py file to the nominal run settings (seen in the above section) with the addition of explicitly running over the newer inputs.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13-L15)                                                                                  
>   >  version = "v04_STXS"                                                                                                   
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-pTVShapeUncNormsGone-MVA
mv output/SMVHVZ_2019_MVA_mc16ade_v04_STXS.140ifb-0L-ade-STXS-pTVShapeUncNormsGone-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-pTVShapeUncNormsGone-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-pTVShapeUncNormsGone-MVA

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-pTVShapeUncNormsGone-MVA  -n -a 5 -l Nominal pTVNormsGone
mv output/pullComparisons output/pullComp_Nominal_VS_pTVNormsGone
~~~

# Running Official 0L Plots for Cut-Based(CB) and Diboson(VV) Analyses. 
The final inputs to be produced for 0L has been produced and they are here:
>     /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec29   
>     /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec29_Zhf_model_STUDY_ONLY_DO_NOT_USE_FOR_PRODUCTION/

Once we understand the fit we can produce the official plots for public consumption. Essentially we want to update the 0L standalone plots in here Appendix B of https://cds.cern.ch/record/2317111/files/ATL-COM-PHYS-2018-512.pdf . Hence will need to run the WSMaker ten times. five times on CB analysis and five times over the VV analysis.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git

mv WSMaker_VHbb WSMaker_VHbb_AuxAnalyses
cd WSMaker_VHbb_AuxAnalyses
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..

cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec29 inputs/
mv inputs/2019Dec29 inputs/SMVHVZ_2019_MVA_mc16ade_v05_STXS

cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec29 inputs/
mv inputs/2019Dec29_Zhf_model_STUDY_ONLY_DO_NOT_USE_FOR_PRODUCTION inputs/SMVHVZ_2019_MVA_mc16ade_v06_STXS
~~~
1) Run the CB analysis with the EXPECTED pulls          
2) Run the CB analysis over the post-fit variables    
3) Run the CB analysis with Rankings       
4) Run the CB analysis with Breakdowns
5) Run the CB analysis with Significances   

6,7) Run the VV analysis over the with the EXPECTED pulls and post-fit variables  
8,9,10) Run the VV analysis with Rankings, Breakdowns and Significances       

Here is a summary of the plots that we require

| Designation |      Plots                           |  Runs  |
|:-----------:|:-------------------------------------|:------:|
| a           | CBA VS MVA Or/And CBA Asimov VS Data |  (1)   |
| b           | CBA ranking and breakdown            | (3,4)  |
| c           | CBA post-fit plots for mBB           |  (2)   |   
| d           | CBA Significance                     |  (5)   |         
| e           | VV VS MVA Or/And CBA Asimov VS Data  |  (6)   |
| f           | VV ranking and breakdown             | (8,9)  |
| g           | VV post-fit plots for mBB            |  (7)   | 
| h           | VV Significance                      | (10)   | 

Leaving the large parts of the launch_default_jobs.py unchanged against the default.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_AuxAnalyses
source setup.sh
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13)                                                                                  
>   >  version = "v05_STXS"                                                                                               
~~~
cd build && rm -rf *
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA
~~~
## 1) Run the CB analysis with the EXPECTED pulls 
~~~
vim scripts/analysis_plotting_config.py 
~~~
>    CHANGE what it condisers to be signal (~L10)                                                                             
>   >  vh_fit=True                                                                                                         
>   >  vh_cba=True                                                                                                         
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Analysis conditions (~L17)                                                                                       
>   >  doCutBase = True                                                                                                       

>    CHANGE flags to run over the pulls (~L60)                                                    
>   >  doExp = 0                                                                                                             

>    CHANGE Analyses outputs (~L67-80)  
>   >  runPulls = True                                                                                                       
>   >  doplots = True                                                                                                        
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-CBA-Pulls
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-CBA-Pulls_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-CBA-Pulls_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-CBA-Pulls
~~~
## 2) Run the CB analysis over the post-fit variables
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Analyses outputs (~L67-80)                                                                                       
>   >  runPulls = False                                                                                                     

>    CHANGE postfit variables of interest (~L92)                                                                             
>   >  vs2tag = ['pTV','MET','pTB1','pTB2','mBB','dRBB','dEtaBB','dPhiVBB','dEtaVBB','MEff','MEff3','dPhiLBmin','mTW','mLL','dYWH','Mtop','pTJ3','mBBJ','mBBJ3','METSig'] -> vs2tag =  ['mBB','MET']                                                                        
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-CBA-PostFit
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-CBA-PostFit_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-CBA-PostFit_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-CBA-PostFit-mBB
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-CBA-PostFit_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-CBA-PostFit_0_mc16ade_Systs_MET_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-CBA-PostFit-MET
~~~
## 3,4,5) Run the CB analysis with Rankings, Breakdowns and Significances  
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flags to get Rankings, Breakdowns and Significances (~L60-L80)                                                    
>   >  doExp = 1                                                                                                             
>   >  runPulls = False                                                                                                      
>   >  runBreakdown = True                                                                                                   
>   >  runRanks = True                                                                                                       
>   >  runP0 = True                                                                                                           
>   >  doplots = False                                                                                                        
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-CBA-Full
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-CBA-Full_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-CBA-Full_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-CBA-Full
~~~

Be reminded that thesea are only the Asimov significances. If you require the data conditional ones, you need to run again with the following flags set 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flags to get Data-Conditional Significances (~L60-L80)                                                  
>   >  doExp = 0                                                                                                             >   >  runBreakdown = True    
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-CBA-DataSignif
~~~
## 8,9,10) Run the VV analysis with Rankings, Breakdowns and Significances 
~~~
vim scripts/analysis_plotting_config.py 
~~~
>    CHANGE what it condisers to be signal (~L10)                                                                             
>   >  vh_fit=False                                                                                                         
>   >  vh_cba=False                                                                                                       

~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Analyses conditions (~L17-L19)                                                                                   
>   >  doCutBase = False                                                                                                    
>   >  doDiboson = True                                                                                                       

>    CHANGE flags to get Asimov Significances (~L60)                                                  
>   >  doExp = 1                                                                                                            
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_AuxAnalyses
source setup.sh
cd build && rm -rf *
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-Full
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-VV-Full_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-VV-Full_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-VV-Full

> OR run each flag seperately and run each one in parallel.

cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_AuxAnalyses
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-Breakdowns
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-Ranking
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-Significance

~~~
Remembering once again that if you want to get the Data-Conditional Significances you need to run again with some flags changed.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE flags to get Data-Conditional Significances (~L60-L80)                                                  
>   >  doExp = 0                                                                                                             >   >  runBreakdown = True    
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-DataSignif

~~~
## 6) Run the VV analysis over the post-fit variables 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    ADD running of post-fit variables (~L15)                                                    
>   >  doPostFit = True                                                                                                      

>    CHANGE flags to not get Rankings, Breakdowns and Significances (~L60-L80)                                               
>   >  doExp = 0                                                                                                            
>   >  runPulls = False                                                                                                      
>   >  runBreakdown = False                                                                                                  
>   >  runRanks = False                                                                                                      
>   >  runP0 = False                                                                                                       
>   >  doplots = False                                                                                                         

>    CHANGE postfit plots to run over only ones we are interested in (~L92)                                                   
  vs2tag = ['pTV','MET','pTB1','pTB2','mBB','dRBB','dEtaBB','dPhiVBB','dEtaVBB','MEff','MEff3','dPhiLBmin','mTW','mLL','dYWH','Mtop','pTJ3','mBBJ','mBBJ3','METSig'] ->   vs2tag = ['mBB','MET']
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-Postfit
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-VV-Postfit_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-VV-Postfit_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-VV-PostFit-mBB
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-VV_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-VV-PostFit_0_mc16ade_Systs_MET_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-VV-PostFit-MET
~~~
## 7) Run the VV analysis over the EXPECTED pulls 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE running of post-fit variables (~L15)                                                    
>   >  doPostFit = False

>   >  doExp = 0                                                                                                              
>   >  runPulls = True   
>   >  doplots = True                                                                                                         
~~~
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-VV-Pulls
mv output/SMVHVZ_2019_MVA_mc16ade_v05_STXS.140ifb-0L-ade-STXS-baseline-VV-Pulls_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-VV-Pulls_0_mc16ade_Systs_mvadiboson_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-VV-Pulls
~~~
## Getting the Final plots.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_AuxAnalyses
source setup.sh
~~~
### CBA Vs Asimov and VV Vs Asimov Pulls
~~~
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-CBA-Pulls -n -a 5
mv output/pullComparisons output/pullComp_CBADataVSAsimov

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-VV-Pulls -n -a 5
mv output/pullComparisons output/pullComp_VVDataVSAsimov
~~~
### CBA and VV Breakdowns
~~~
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-0L-ade-STXS-baseline-CBA-Full
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-0L-ade-STXS-baseline-VV-Full

vim /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_AuxAnalyses/output/140ifb-0L-ade-STXS-baseline-CBA-Full/plots/breakdown/muHatTable_mode11_Asimov1_SigXsecOverSM.txt 
vim /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_AuxAnalyses/output/140ifb-0L-ade-STXS-baseline-VV-Full/plots/breakdown/muHatTable_mode11_Asimov1_SigXsecOverSM.txt 
~~~
### CBA and VV Rankings
~~~
python WSMakerCore/scripts/makeNPrankPlots.py 140ifb-0L-ade-STXS-baseline-CBA-Full
imgcat output/140ifb-0L-ade-STXS-baseline-CBA-Full/pdf-files/pulls_SigXsecOverSM_125.pdf

python WSMakerCore/scripts/makeNPrankPlots.py 140ifb-0L-ade-STXS-baseline-VV-Full
imgcat output/140ifb-0L-ade-STXS-baseline-VV-Full/pdf-files/pulls_SigXsecOverSM_125.pdf
~~~
### Mbb Plots
~~~
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-VV-Pulls 140ifb-0L-ade-STXS-baseline-VV-PostFit-mBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-CBA-Pulls 140ifb-0L-ade-STXS-baseline-CBA-PostFit-mBB

~~~
Neatening things up then
~~~
cd outputs
mkdir vv-mva vh-cba

cd vv-mva
mkdir breakdown  fcc  postfit_plots  pullComparisons_0L  ranking  significances
mkdir postfit_plots/mBB postfit_plots/SRCR
cp ../pullComp_CBADataVSAsimov/* pullComparisons_0L
cp ../140ifb-0L-ade-STXS-baseline-VV-Full/pdf-files/pulls_SigXsecOverSM_125.pdf ranking
cp ../140ifb-0L-ade-STXS-baseline-VV-Full/logs/output_getSig_125.log significances
cp ../140ifb-0L-ade-STXS-baseline-VV-Full/plots/breakdown/* breakdown
cp -r ../140ifb-0L-ade-STXS-baseline-VV-Pulls/plots/fcc/* fcc
cp ../140ifb-0L-ade-STXS-baseline-VV-Pulls/plots/postfit/* postfit_plots/SRCR
cp ../140ifb-0L-ade-STXS-baseline-VV-PostFit-mBB/plots/postfit/* postfit_plots/mBB

cd ../vh-cba
mkdir breakdown  fcc  postfit_plots  pullComparisons_0L  ranking  significances
mkdir postfit_plots/mBB postfit_plots/SRCR
cp ../pullComp_CBADataVSAsimov/* pullComparisons_0L
cp ../140ifb-0L-ade-STXS-baseline-CBA-Full/pdf-files/pulls_SigXsecOverSM_125.pdf ranking
cp ../140ifb-0L-ade-STXS-baseline-CBA-Full/logs/output_getSig_125.log significances
cp ../140ifb-0L-ade-STXS-baseline-CBA-Full/plots/breakdown/* breakdown
cp -r ../140ifb-0L-ade-STXS-baseline-CBA-Pulls/plots/fcc/* fcc
cp ../140ifb-0L-ade-STXS-baseline-CBA-Pulls/plots/postfit/* postfit_plots/SRCR
cd ..

cp -r vv-mva /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2019-12-10/l0/
cp -r vh-cba /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2019-12-10/l0/
~~~

# Investigating B-tagging
With the new inputs, the 0L fit needs to be checked that the features present in the flavour sector pulls are still there, and if so they need to be investigated. To do this the first thing we need to do is to restore the launch options to the default and decorrelate the entire fit sector in PtV, SR/CR and Njet. 

## Comparison of new and old fits
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
mv /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging/output/140ifb-0L-ade-STXS-baseline-MVA/ /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging/output/140ifb-0L-ade-STXS-baseline-MVA_OLD/
mv /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Btagging/output/140ifb-0L-ade-STXS-baseline-MVA_OLD/ output/
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA_OLD -n -a 5 -l NEW OLD
mv output/pullComparisons output/pullComp_NEWvsOLDpulls
~~~
## Comparison of pulls or all analyses
~~~
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA-Pulls 140ifb-0L-ade-STXS-baseline-CBA-Pulls 140ifb-0L-ade-STXS-baseline-VV-Pulls -n -a 5 -l MVA-VH CBA-VH MVA-VV
mv output/pullComparisons output/pullComp_MVA_vs_CBA_vs_VV
~~~
## De-correlation of everything for B-tagging
The new fit has waay more B-tagging pulls, but what makes a pull.

So the indices in the pulls usually refer to an Eigenvector decomposition. In the case of the JET systematics for example, you first group them into physical categories, like Modelling, Statistical Uncertainties, and in each of these categories you do the decomposition. 

The decomposition is basically finding  a linear combination of uncertainties. If you see each systematic variation as being correlated with others than you can think of the decomposition as dioganolizing this matrix(1**). then your new "decomposed" uncertainties (or eigenvectors) are a linear combination of the ones you put in. You then label them according to their impact (1 being the one with the most impact, ...). It is important to note that it is no longer so clear what they actually are. The input systematic uncertainties are of course meaningful in the sense that one corresponds maybe to the difference between MC generator 1 and MC generator 2 but after the decomposition they are linear combinations of each other. Then you usually say I take the N most important ones separately and the ones with i > N I add in quadrature to form the N+1 NP. 

(1**)the matrix that is used for the eigenvector problem. It starts from the construction of the covariance matrix corresponding to each source of uncertainty, and then sums these covariance matrices to obtain the total covariance matrix. Being a symmetric, positive-definite matrix this can be considered as an eigenvalue problem. The eigenvectors that "solve" this problem can be seen as "directions" in which to carry out independent variations. The sizes of the variations are given by the square root of the corresponding eigenvalues.
 
The fit exhibits a pull in the 21st one, but lets say the problem is more fundamental and the half sigma pull of B_1 is more important. Will decorrelate B1 in ptV, nJets and in SRCR and see what happens.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE to construct Asimov dataset using central values from fit to data (~L62)                                           
>   >  doExp=1 -> doExp=0                                                                                                    

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)                                                         
>   >  createSimpleWorkspace = True                                                                                
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = False                                                                                              
>   >  runRanks = False                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runP0 = False                                                                                                        
>   >  runToyStudy = False                                                                                                 

>    ADD additional debug plots for shape plots(~L72)                                                                
>   >  doplots = True   

~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of B_1 systematic (~L566)                                                                                  
>   >  else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}}); ->       
>   >  else {                                                                                                                
>   >        if (sysname == "Eigen_B_1") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                           
>   >        else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}});   
>   >      }                                                                                                                
~~~ 
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py B_1_nJetPtVSRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.B_1_nJetPtVSRCRDeco_fullRes_VHbb_B_1_nJetPtVSRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-B_1_nJetPtVSRCRDeco 
~~~
Then you need to compare this output to that of the current un-decorrelated fit.
~~~
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-B_1_nJetPtVSRCRDeco  -n -a 5 -l Nominal B_1FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_B1FullDeco
~~~
It seems that B_1 pulls are not pulling any of the other pulls. Will decorrelate B21 in ptV, nJets and in SRCR and see what happens.
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of B_21 systematic instead of B_1 (~L566)                                                                   
>   >        if (sysname == "Eigen_B_1") ... -> if (sysname == "Eigen_B_21")                                                   
~~~ 
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py B_21_nJetPtVSRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.B_21_nJetPtVSRCRDeco_fullRes_VHbb_B_21_nJetPtVSRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-B_21_nJetPtVSRCRDeco 
~~~
Then you need to compare this output to that of the current un-decorrelated fit.
~~~
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-B_21_nJetPtVSRCRDeco  -n -a 5 -l Nominal B_21FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_B21FullDeco
~~~
Given that the B21 in ptV, nJets and in SRCR are also inconclusive, we will also fully de-correlate Light_0, C_0 and B_0
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of Light_0 systematic instead of B_21 (~L566)                                                               
>   >        if (sysname == "Eigen_B_1") ... -> if (sysname == "Eigen_Light_0")                                               
>   >        [ [ [ AND ] ] ]                                                                                              
>   >        if (sysname == "Eigen_B_1") ... -> if (sysname == "B_0")                                                    
>   >        [ [ [ AND ] ] ]                                                                                                  
>   >        if (sysname == "Eigen_C_1") ... -> if (sysname == "C_0")                                                         
~~~
~~~
>     Light_0
~~~ 
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py Light_0_nJetPtVSRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.Light_0_nJetPtVSRCRDeco_fullRes_VHbb_Light_0_nJetPtVSRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-Light_0_nJetPtVSRCRDeco 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-Light_0_nJetPtVSRCRDeco  -n -a 5 -l Nominal Light_0FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_Light_0FullDeco
~~~
>     C_0
~~~ 
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py C_0_nJetPtVSRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.C_0_nJetPtVSRCRDeco_fullRes_VHbb_C_0_nJetPtVSRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-C0_nJetPtVSRCRDeco 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-C_0_nJetPtVSRCRDeco  -n -a 5 -l Nominal C0FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_C_0FullDeco
~~~
>     B_0
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py B_0_nJetPtVSRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.B_0_nJetPtVSRCRDeco_fullRes_VHbb_B_0_nJetPtVSRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-B_0_nJetPtVSRCRDeco 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-B_0_nJetPtVSRCRDeco  -n -a 5 -l Nominal B_0FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_B_0FullDeco
~~~
Time to de-correlate B0,B1 and B21 simultaneously
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of B_21 and B_1 systematic in addition to B_0 (~L569)                                                       
>   >     else if (sysname == "Eigen_B_1") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                           
>   >     else if (sysname == "Eigen_B_21") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                           
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py B0B1B12_nJetPtVSRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.B0B1B12_nJetPtVSRCRDeco_fullRes_VHbb_B0B1B12_nJetPtVSRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-B0B1B12_nJetPtVSRCRDeco 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-B0B1B12_nJetPtVSRCRDeco  -n -a 5 -l Nominal B0B1B12FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_B0B1B12FullDeco
~~~
##  Addition of dummy systematics for B-tagging
One thing that could be tested, would be to introduce two dummy systs, like +10% or so, one applied just to CRHigh, and one applied just to CRLow, so see what in the fit covers for these additions 
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    REMOVE splitting of B_X systematics (~L566)                                                                               
>   >  else {                                                                                                                
>   >        if (sysname == "Eigen_B_1") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                           
>   >        else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}});   
>   >      }  ->                                                                                                               
>   >  else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}});        

>    ADD dummy of systematics (~L383)                                                                                         
>   >  if (doNewRegions){                                                                                                     
>   >      normSys("SysDummyBoson_CRLow", 0.10, SysConfig{{"Wbb","Zbb"}}.applyIn(P::descr=="CRLow"));                         
>   >      normSys("SysDummyTop_CRLow", 0.10, SysConfig{{"ttbar","stop"}}.applyIn(P::descr=="CRLow"));                       
>   >      normSys("SysDummyBoson_CRHigh", 0.10, SysConfig{{"Wbb","Zbb"}}.applyIn(P::descr=="CRHigh"));                       
>   >      normSys("SysDummyTop_CRHigh", 0.10, SysConfig{{"ttbar","stop"}}.applyIn(P::descr=="CRHigh"));                     
>   >  }    
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-CRDummySysts-MVA
mv output/SMVHVZ_2019_MVA_mc16ade_v04_STXS.140ifb-0L-ade-STXS-CRDummySysts-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-CRDummySysts-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-CRDummySysts-MVA 
~~~
Now you need to edit the pull plot scripts such that the variables you added can be seen.
~~~
vim scripts/analysisPlottingConfig.py
~~~
>    ADD dummy of systematics to the cov_classification (~L190)                                                               
>   >          "LUMI": [False, ["LUMI"],[]] -> "LUMI": [False, ["LUMI" ,"SysDummyBoson_CRLow","SysDummyBoson_CRHigh","SysDummyTop_CRLow","SysDummyTop_CRHigh",[]],                                           
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-CRDummySysts-MVA -n -a 5 -l Nominal ExtraSysts
mv output/pullComparisons output/pullComp_Nominal_VS_AdditionalCRSysts
~~~
In the fits that have been done we want to investigate problematic pulls that remain when we turn the SR into only one bin. We see that in 0L the ttbar_PS and ttbar_ME systematics have funny behaviours. Also we should check that they behave similarly to that of 1L. 
There, as there usually is a new set of inputs to look at as well:
 >   /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec10_for_second_unblinding

##  De-correlation of problematic ttbar systematics
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
cd inputs
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019Dec10_for_second_unblinding .
mv 2019Dec10_for_second_unblinding SMVHVZ_2019_MVA_mc16ade_v06_STXS

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L13)                                                                                  
>   >  version = "v06_STXS"                                                                                               

~~~
vim scripts/analysisPlottingConfig.py
~~~
>    ADD dummy of systematics to the cov_classification (~L190)                                                               
>   >          "LUMI": [False, ["LUMI" ,"SysDummyBoson_CRLow","SysDummyBoson_CRHigh","SysDummyTop_CRLow","SysDummyTop_CRHigh",[]],  ->  "LUMI": [False, ["LUMI"],[]]                            

~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of ttbar_PS systematics (~L566)                                                                             
>   >  m_histoSysts.insert({ "SysBDTr_ttbar_PS", SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.applyTo("ttbar").decorrIn({ (P::nLep==2),(P::nLep==1)&&(P::binMin==75)}) }); ->     m_histoSysts.insert({ "SysBDTr_ttbar_PS", SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.applyTo("ttbar").decorr({P::nJet, P::binMin, P::descr}) });

>    COMMENT OUT dummy systematics (~L383)                                                                                    
>   >  /* if (doNewRegions){                                                                                                   
>   >      normSys("SysDummyBoson_CRLow", 0.10, SysConfig{{"Wbb","Zbb"}}.applyIn(P::descr=="CRLow"));                         
>   >      normSys("SysDummyTop_CRLow", 0.10, SysConfig{{"ttbar","stop"}}.applyIn(P::descr=="CRLow"));                       
>   >      normSys("SysDummyBoson_CRHigh", 0.10, SysConfig{{"Wbb","Zbb"}}.applyIn(P::descr=="CRHigh"));                       
>   >      normSys("SysDummyTop_CRHigh", 0.10, SysConfig{{"ttbar","stop"}}.applyIn(P::descr=="CRHigh"));                     
>   >  }  */  
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSSplit
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSSplit_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSSplit_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_All

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_All -n -a 5 -l Nominal ttbar_PSDecorr_All
mv output/pullComparisons output/pullComp_Nominal_VS_ttbar_PSDecorr_All
~~~ 

Split partly as well. 
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of ttbar_PS systematics (~L566)                                                                             
>   >   m_histoSysts.insert({ "SysBDTr_ttbar_PS", SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.applyTo("ttbar").decorr({P::nJet, P::binMin, P::descr}) }); ->  m_histoSysts.insert({ "SysBDTr_ttbar_PS", SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.applyTo("ttbar").decorr({P::nJet, P::binMin}) });                                                                                                                          
~~~    
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJptV
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-Nominal-MVA-ttbar_PSDecorr_nJptV_fullRes_VHbb_140ifb-0L-ade-STXS-Nominal-MVA-ttbar_PSDecorr_nJptV_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJptV 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJptV -n -a 5 -l Nominal ttbar_PS-Decorr_nJptV
mv output/pullComparisons output/pullComp_Nominal_VS_ttbar_PSDecorr_nJptV
~~~
Now do the same for ttbar_ME. Essentially replace all instances of "PS" with "ME" and visa versa. Once you are done in the same steps as above you will run for the full decorrelation
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_All
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_All_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_All_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_All

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_All -n -a 5 -l Nominal ttbar_MEDecorr_All
mv output/pullComparisons output/pullComp_Nominal_VS_ttbar_MEDecorr_All
~~~
And the for the partial decorrelation in just nJet and ptV
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_nJptV
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_nJptV_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_nJptV_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_nJptV 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ttbar_MEDecorr_nJptV -n -a 5 -l Nominal ttbar_ME-Decorr_nJptV
mv output/pullComparisons output/pullComp_Nominal_VS_ttbar_MEDecorr_nJptV
~~~
The next stage is to de-correlate the ttbarPS in njet only in the standard 0L fit and the oneBin fit.
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of ttbar_PS systematics (~L566)                                                                             
>   >   m_histoSysts.insert({ "SysBDTr_ttbar_PS", SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.applyTo("ttbar").decorr({P::nJet, P::binMin}) }); ->  m_histoSysts.insert({ "SysBDTr_ttbar_PS", SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.applyTo("ttbar").decorr({P::nJet}) });                             
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJ
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJ_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJ_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJ

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJ -n -a 5 -l Nominal ttbar_PS-Decorr_nJ
mv output/pullComparisons output/pullComp_Nominal_VS_ttbar_PSDecorr_nJ
~~~
Then to add the one bin in the SR
~~~
vim src/binning_vhbbrun2.cpp
~~~
>    ADD splitting of ttbar_PS systematics (~L566)                                                                           
>   >   //Force oneBin in the SR                                                                                             
>   >   if (doNewRegions && (c(Property::descr).Contains("SR")) ){                                                           
>   >    return oneBin(c);                                                                                                   
>   >   }                                                                                                                     

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJOneBin
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJOneBin_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJOneBin_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJOneBin

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-ttbar_PSDecorr_nJOneBin -n -a 5 -l Nominal ttbar_PS-Decorr_nJOneBin
mv output/pullComparisons output/pullComp_Nominal_VS_ttbar_PSDecorr_nJOneBin
~~~                                                                                           

##  De-correlation of Light_0
Now we need to look into each of the pulls in particular to try and fin out what is going on.  The first one we will do it with is the Light_0 pull. For this we will de-correlate in pTV, nJ and SRCR separately 

Then to add the one bin in the SR
~~~
vim src/binning_vhbbrun2.cpp
~~~
>    COMMENT OUT single bin in the SR (~L118)                                                                   
>   >    /* //Force oneBin in the SR                                                                                         
>   >   if (doNewRegions && (c(Property::descr).Contains("SR")) ){                                                           
>   >    return oneBin(c);                                                                                                   
>   >   } */                                                                                                                  

~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of Light_0 systematic (~L566)                                                                             
>   >  else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}}); ->     
>   >  else {                                                                                                                
>   >        if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr(P::nJet) });                                                
>   >        [ [ [ AND ] ] ]   

>   >        if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr(P::binMin) });                                                     
>   >        [ [ [ AND ] ] ]                                                                                                  
>   >        if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr(P::descr) });                  

>   >        else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}});   
>   >      }                                                                                                                
~~~ 
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py Light_0_nJetDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.Light_0_nJetDeco_fullRes_VHbb_Light_0_nJetDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/Light_0_nJetDeco
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA Light_0_nJetDeco -n -a 5 -l Nominal Light_0_nJetDeco
mv output/pullComparisons output/pullComp_Nominal_VS_Light_0_nJetDeco

python scripts/launch_default_jobs.py Light_0_pTVDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.Light_0_pTVDeco_fullRes_VHbb_Light_0_pTVDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/Light_0_pTVDeco
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA Light_0_pTVDeco -n -a 5 -l Nominal Light_0_pTVDeco
mv output/pullComparisons output/pullComp_Nominal_VS_Light_0_pTVDeco

python scripts/launch_default_jobs.py Light_0_SRCRDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.Light_0_SRCRDeco_fullRes_VHbb_Light_0_SRCRDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/Light_0_SRCRDeco
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA Light_0_SRCRDeco -n -a 5 -l Nominal Light_0_SRCRDeco
mv output/pullComparisons output/pullComp_Nominal_VS_Light_0_SRCRDeco
~~~
Then make a webpage. Or copy them to your /eos/  
~~~
cp -r output/pullComp_Nominal_VS_Light_0* ~/public/flavDecorr/
cd ~/public/flavDecorr/
python "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2/WSMakerCore/macros/webpage/createHtmlOverview.py"
~~~
##  Full Decorrelation of all of B_0, C_0 and Light_0
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of B_0 and C_0 to the splitting of Light_0 systematics (~L566)                                             
>   >   if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                         
>   >   else if (sysname == "Eigen_B_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                         
>   >   else if (sysname == "Eigen_C_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr({P::nJet, P::binMin, P::descr}) });                                                           

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py Light_0-B_0-C_0_FullDeco
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.Light_0-B_0-C_0_FullDeco_fullRes_VHbb_Light_0-B_0-C_0_FullDeco_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-MVA-Light_0-B_0-C_0_FullDeco
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-Light_0-B_0-C_0_FullDeco -n -a 5 -l Nominal Flavour_0_FullDeco
mv output/pullComparisons output/pullComp_Nominal_VS_Light_0-B_0-C_0FullDeco
~~~
##  Increasing the Normalisation Uncertainties
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    CHANGE normalisation levels of Vx (~L576)   
>   >    // Wl and Zl                                                                                                        
>   >    sampleNormSys("Wl", 0.32); -> sampleNormSys("Wl", 0.64);                                                            
>   >    sampleNormSys("Zl", 0.18); -> sampleNormSys("Zl", 0.36);                                                             
>   >                                                                                                                        
>   >    // Wcl and Zcl                                                                                                      
>   >    sampleNormSys("Wcl", 0.37);  -> sampleNormSys("Wcl", 0.74);                                                           
>   >    sampleNormSys("Zcl", 0.23);  -> sampleNormSys("Zcl", 0.46);                                                                                       
  
>    COMMENT OUT splitting of Light_0 systematic (~L576)                                                                       
>   >    //if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorr(P::nJet) });                                                                                   
>   >    else m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}}); ->  m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}});                     

~~~    
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-MVA-DoubleNormSysts
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-MVA-DoubleNormSysts_fullRes_VHbb_140ifb-0L-ade-STXS-MVA-DoubleNormSysts_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-DoubleNormSysts
python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-DoubleNormSysts -n -a 5 -l Nominal DoubleNormSysts
mv output/pullComparisons output/pullComp_Nominal_VS_DoubleNormSysts
~~~
Then adds in NormSysts for Vbl
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD normalisaition levels of Vbls (~L168)   
>   >    // Wbl and Zbl                                                                                                      
>   >    sampleNormSys("Wbl", 0.84);                                                            
>   >    sampleNormSys("Zbl", 0.56);                                                            
>   >                                                                                                                        
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-MVA-DoubleNormSysts_Vbl
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-MVA-DoubleNormSysts_Vbl_fullRes_VHbb_140ifb-0L-ade-STXS-MVA-DoubleNormSysts_Vbl_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-DoubleNormSysts_Vbl

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-DoubleNormSysts 140ifb-0L-ade-STXS-MVA-DoubleNormSysts_Vbl -n -a 5 -l Nominal DoubleNormSysts DoubleNormSysts_Vbl
mv output/pullComparisons output/pullComp_Nominal_VS_DoubleNormSysts_Vbl
~~~

##  Splitting the Light_0 into process and njet
Now we know that the V+lights are a small contribution we want to split this component into both njets and processes. Also we want to try and see what effect the increased norm has on this. 
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>   ADD decorrelation for flavour tagging (~L553)                                                                            
>   >    SysConfig FTag_Process_Decorr_Config = noSmoothConfig;                                                               
>   >    FTag_Process_Decorr_Config.decorrFun([](const PropertiesSet& pset, const Sample& s) {                                 
>   >        if( s.name() == "Zbb" || s.name() == "Wbb" || s.name() == "Zbc" || s.name() == "Wbc"  || s.name() == "Zcc" || s.name() == "Wcc" || s.name() == "ttbar" || s.hasKW("Diboson") || s.hasKW("Higgs")) return "_nonVlight";                      
>   >        else if( s.name() == "Zbl" || s.name() == "Wbl" || s.name() == "Wcl" || s.name() == "Zcl" || s.name() == "Wl" || s.name() == "Zl" ) return "_Vlight";                                                                                           
>   >        else return "";                                                                                                   
>   >    });                                                                                                                 

>   ADD selective decorrelation for problematic flavour tagging pulls (~L583)
>   >    if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , FTag_Process_Decorr_Config.decorr(P::nJet) });  
>   >    m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig}); -> else m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});

Since there are many ways to skin a cat, alternatively you can try to define a keyword in the samples builder and then decorrelate to it in the systematics list builder.
~~~
vim src/samplesbuilder_vhbbrun2.cpp
~~~
>   ADD new keywork for V+lights (~L104)                                                                            
>   >    {"Vlight", {"Zbl","Zcl","Zl","Wbl","Wcl","Wl"}}, 

>   ADD selective decorrelation for problematic flavour tagging pulls (~L583)
>   >    if (sysname == "Eigen_Light_0") m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}.decorrTo({"Vlight", "Vlight"}).decorr(P::nJet) });  
>   >    m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig}); -> else m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-MVA-Light_0_nJetProcDecorr
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-MVA-Light_0_nJetProcDecorr_fullRes_VHbb_140ifb-0L-ade-STXS-MVA-Light_0_nJetProcDecorr_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-MVA-Light_0_nJetProcDecorr

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-Light_0_nJetProcDecorr -n -a 5 -l Nominal L-0_nJ_ProcDecorrNormx2
mv output/pullComparisons output/pullComp_Nominal_VS_Light_0_nJProcDecorr_DoubleNorm
~~~
##  Creating a new systematic.
Given that we now think that the pull in Light_0 is due to mis-modelling on Light_0 itself, we can test this hypothesis. If we create a systematic that acts like Light_0, but we give it a larger variation then the fit should preferentially pull it over Light_0. From the NormVal plots we see that we have roughly a effect of 30% per light jet. So we shall be varying this systematic by about 70%. 
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>   ADD new Systematic (L160)                                                                                           
>   >    normSys("SysDummyLight0", 0.70, SysConfig{"Vlight"})                                                               

>   RESTORE flavour tagging decorrelations to their default (L559)                                                           
>   >    for(auto sysname: btagSysts) {                                                                                      
>   >      if(!PCBTInputs) m_histoSysts.insert({ "SysFT_EFF_"+sysname , noSmoothConfig});                                    
>   >      else {
>   >        m_histoSysts.insert({ "SysFT_EFF_"+sysname , SysConfig{T::shape, S::noSmooth, Sym::symmetriseOneSided}});       
>   >      }                                                                                                                 
>   >    }                                                                                                                   


Now you need to edit the pull plot scripts such that the sytematic you added can be seen.
~~~
vim scripts/analysisPlottingConfig.py
~~~
>    ADD dummy of systematics to the cov_classification (~L190)                                                               
>   >    "BTag": [False, ["SysFT_EFF_Eigen", "SysFT_EFF_extrapolation"], []], -> "BTag": [False, ["SysFT_EFF_Eigen", "SysFT_EFF_extrapolation", "SysDummyLight0"], []],                  

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-MVA-AddLight-likeSyst
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-MVA-AddLight-likeSyst_fullRes_VHbb_140ifb-0L-ade-STXS-MVA-AddLight-likeSyst_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-MVA-AddLight-likeSyst

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-MVA-AddLight-likeSyst -n -a 5 -l Nominal AddLight-likeSyst
mv output/pullComparisons output/pullComp_Nominal_VS_AddLight-likeSyst
~~~
Now we want to do a oneBin fit on this version to see if the new pull has impaced the shape, or the normalisation part of the systematic
~~~
vim src/binning_vhbbrun2.cpp
~~~
>    ADD section to inroduce one bin into the signal region (~L118)                                                           
>   >   //Force oneBin in the SR                                                                                             
>   >   if (doNewRegions && (c(Property::descr).Contains("SR")) ){                                                           
>   >    return oneBin(c);                                                                                                   
>   >   }                                                                                                                     
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-MVA-AddLight-likeSyst-OneBin
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-MVA-AddLight-likeSyst-OneBin_fullRes_VHbb_140ifb-0L-ade-STXS-MVA-AddLight-likeSyst-OneBin_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-MVA-AddLight-likeSyst-OneBin

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-OneBin 140ifb-0L-ade-STXS-MVA-AddLight-likeSyst 140ifb-0L-ade-STXS-MVA-AddLight-likeSyst-OneBin  -n -a 5 -l Nominal OneBin Light-likeSyst Light-likeSyst-1Bin 
mv output/pullComparisons output/pullComp_Nominal_VS_OneBin_VS_Light-likeSyst_VS_Light-likeSyst-1Bin
~~~

##  Running a no pruning fit.
At a loss of what to try next, we want to have a look at what happens to the pruned systematics in the fit if they were not to be pruned. First we need to restore all the files to their factory settings
~~~
vim src/binning_vhbbrun2.cpp
~~~
>    COMMENT OUT section to inroduce one bin into the signal region (~L118)                                                   
>   >   /*                                                                                                                   
>   >   //Force oneBin in the SR                                                                                             
>   >   if (doNewRegions && (c(Property::descr).Contains("SR")) ){                                                           
>   >    return oneBin(c);                                                                                                   
>   >   }                                                                                                                     
>   >   */                                                                                                                 
~~~
vim scripts/analysisPlottingConfig.py
~~~
>    REMOVE dummy of systematics to the cov_classification (~L185)                                                           
>   >     "BTag": [False, ["SysFT_EFF_Eigen", "SysFT_EFF_extrapolation", "SysDummyLight0"], []], -> "BTag": [False, ["SysFT_EFF_Eigen", "SysFT_EFF_extrapolation"], []]                
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>   COMMENT OUT new Systematic (L160)                                                                                        
>   >    // normSys("SysDummyLight0", 0.70, SysConfig{"Vlight"})                                                             

Now we need to turn off the pruning.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    ADD flag to the baseline_configs to turn off pruning (~L150)                                                           
>   >  'DoPruneSysts': False,                                                                                            

~~~
vim WSMakerCore/src/engine.cpp 
~~~
>    CHANGE default flag for doing pruning in the fit to false  (~L184)                                                       
>   >  bool doPruneSyst = m_config.getValue("DoPruneSyst", false); //note pruneOneSideShapeSysts below is not affected by this switch!

>   COMMENT OUT pruning of one-sided shape systematics (L228)                                                             
>   >      //sic.pruneOneSideShapeSysts(); // should happen in very few circumstances                                        

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-NoPruning
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-NoPruning_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-NoPruning_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-NoPruning

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-NoPruning -n -a 5 -l Nominal NoPruning
mv output/pullComparisons output/pullComp_Nominal_VS_NoPruning
~~~
##  Running alternative threshold prunings in the fit.
The no-pruning fit was rather informative and the Light_0 pull was reduced simply by adding in the extra pulled systematics. Now if we can play with the pruning we might be able to pinpoint why this was the case. 
Now we need to turn back on the pruning adn restore the launcher file to the default settings.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)                                                         
>   >  createSimpleWorkspace = True                                                                                
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = False                                                                                              
>   >  runRanks = False                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runP0 = False                                                                                                        
>   >  runToyStudy = False                                                                                                 

>    ADD additional debug plots for shape plots(~L72)                                                                
>   >  doplots = True                                                                                                        

>    ADD flag to the baseline_configs to turn on pruning (~L150)                                                           
>   >  'DoPruneSysts': True,                                                                                            
~~~
vim WSMakerCore/src/engine.cpp 
~~~
>    CHANGE default flag for doing pruning in the fit to true  (~L184)                                                       
>   >  bool doPruneSyst = m_config.getValue("DoPruneSyst", true); //note pruneOneSideShapeSysts below is not affected by this switch!

>   COMMENT IN pruning of one-sided shape systematics (L228)                                                             
>   >      sic.pruneOneSideShapeSysts(); // should happen in very few circumstances                                        

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

time python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-2
~~~
You can delete this after you have the timing information as this is the same as the nominal.
Now we want to change the level of pruning in the fit.
~~~
vim WSMakerCore/src/sampleincategory.cpp
~~~
>    CHANGE Pruining Threshold in both sensitive and non-sensitive bins (L820,L832)
>   >   return this->isSysBelow(sensitiveBins, sys, hsig, 0.01) &&                                                         
>   >   return this->isSysBelow(bins, sys, hbkg, 0.001) && //Changed from 0.005                                              

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

time python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-Pruning1_01
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-Pruning1_01_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-Pruning1_01_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-Pruning1_01

vim WSMakerCore/src/sampleincategory.cpp
~~~
>    CHANGE Pruining Threshold in both sensitive and non-sensitive bins again(L820,L832)
>   >   return this->isSysBelow(sensitiveBins, sys, hsig, 0.005) &&                                                         
>   >   return this->isSysBelow(bins, sys, hbkg, 0.0005) && //Changed from 0.005                                             
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

time python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-Pruning05_005
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-Pruning05_005_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-Pruning05_005_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-Pruning05_005

vim WSMakerCore/src/sampleincategory.cpp
~~~
>    CHANGE Pruining Threshold in both sensitive and non-sensitive bins again(L820,L832)
>   >   return this->isSysBelow(sensitiveBins, sys, hsig, 0.0025) &&                                                         
>   >   return this->isSysBelow(bins, sys, hbkg, 0.00025) && //Changed from 0.005                                             
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..

time python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-Pruning025_0025
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-Pruning025_0025_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-Pruning025_0025_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-Pruning025_0025

python WSMakerCore/scripts/comparePulls.py -w TEST 140ifb-0L-ade-STXS-baseline-MVA-Pruning1_01 140ifb-0L-ade-STXS-baseline-MVA-Pruning025_0025 -n -a 5 -l Nominal PruneThr1_01 PruneThr025_0025
mv output/pullComparisons output/pullComp_NoPruning_VS_Pruning1_01_VS_Pruning025_0025
~~~
Then you can move all of the pull plots to a folder. I have called mine pullComps and I have added some additional structure
~~~
cd output/pullComps
python "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2/WSMakerCore/macros/webpage/createHtmlOverview.py"
cd ..
cp -r pullComps ~/www
~~~
##  Running fit details for the non-pruning fit.
So after this we might remove the pruning for Light_0. To get a better overall picture on the effect this will have on the analysis we will have to run the fit without pruning with the breakdowns and the rankings.

Firstly to run the breakdowns edit
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE running flags to run on the breakdowns (~L66)                                                                   
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = True                                                                                                 

>    ADD flag to the baseline_configs to turn off pruning (~L150)                                                           
>   >  'DoPruneSysts': False,                                                                                            

~~~
vim WSMakerCore/src/engine.cpp 
~~~
>    CHANGE default flag for doing pruning in the fit to false  (~L184)                                                       
>   >  bool doPruneSyst = m_config.getValue("DoPruneSyst", false); //note pruneOneSideShapeSysts below is not affected by this switch!

>   COMMENT OUT pruning of one-sided shape systematics (L228)                                                             
>   >      //sic.pruneOneSideShapeSysts(); // should happen in very few circumstances                                        

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-NoPruning-breakdowns
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-NoPruning-breakdowns_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-NoPruning-breakdowns_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-NoPruning-breakdowns
python WSMakerCore/scripts/mergeBreakdown.py 140ifb-0L-ade-STXS-baseline-MVA-NoPruning-breakdowns

vim /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2/output/140ifb-0L-ade-STXS-baseline-MVA-NoPruning-breakdowns/plots/breakdown/muHatTable_mode11_Asimov1_SigXsecOverSM.txt 
~~~
Then to run the rankings you don't have to run the pulls again or the shapes so this should be quicker to run
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE running flags to run on the breakdowns (~L66)                                                                   
>   >  runPulls = False                                                                                                       
>   >  runBreakdown = False                                                                                                 
>   >  runRanking = True                                                                                                     
>   >  doplots = False                                                                                                       
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-NoPruning-rankings

mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-NoPruning-rankings_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-NoPruning-rankings_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-NoPruning-rankings

python WSMakerCore/scripts/makeNPrankPlots.py 140ifb-0L-ade-STXS-baseline-MVA-NoPruning-rankings
imgcat output/140ifb-0L-ade-STXS-baseline-MVA-NoPruning-rankings/pdf-files/pulls_SigXsecOverSM_125.pdf
~~~
##  Running fit with the pruning only in the Light_X systematics.
This is the final recommentation for the fit. To run pruning on only the LIght_X systematics.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE running flags to run on the breakdowns (~L66)                                                                   
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = False                                                                                                 

>    ADD flag to the baseline_configs to turn off pruning (~L150)                                                           
>   >  'DoPruneSysts': True,                                                                                                 

~~~
vim WSMakerCore/src/engine.cpp 
~~~
>    CHANGE default flag for doing pruning in the fit to true  (~L184)                                                       
>   >  bool doPruneSyst = m_config.getValue("DoPruneSyst", true); //note pruneOneSideShapeSysts below is not affected by this switch!

>   COMMENT IN pruning of one-sided shape systematics (L228)                                                             
>   >      sic.pruneOneSideShapeSysts(); // should happen in very few circumstances                                        
~~~
vim WSMakerCore/src/sampleincategory.cpp
~~~
>    CHANGE Pruining Threshold in both sensitive and non-sensitive bins to original values(L820,L832)                          
>   >   return this->isSysBelow(sensitiveBins, sys, hsig, 0.02) &&                                                         
>   >   return this->isSysBelow(bins, sys, hbkg, 0.005) &&

>    ADD clause to have no pruning for all Light_X systematics (~L565)                                                     
>   >   if(sysobj.first.first != SysType::shape){ -> if(sysobj.first.first != SysType::shape || sysobj.second.name.Contains("Eigen_Light")) {                                                                               

>    ADD clause to have no pruning for all Light_X systematics (~L621,L822, L835)                                            
>   >   (!sys.name.Contains("Eigen_Light")) &&                                                                               

>    ADD exception to chi^2 small shape systematic pruning (~L909)
>   >   if(sys.name.Contains("Eigen_Light")) {return false;}                                                                 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-noLightPruning
mv output/SMVHVZ_2019_MVA_mc16ade_v06_STXS.140ifb-0L-ade-STXS-baseline-MVA-noLightPruning_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-noLightPruning_0_mc16ade_Systs_mva_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-noLightPruning

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA-NoPruning 140ifb-0L-ade-STXS-baseline-MVA-noLightPruning -n -a 5 -l NoPruning NoLightPruning
mv output/pullComparisons output/pullComp_NoPruning_VS_NoLightPruning

python WSMakerCore/scripts/comparePulls.py -w TEST 140ifb-0L-ade-STXS-baseline-MVA-noLightPruning -n -a 5 -l Pruning NoLightPruning
mv output/pullComparisons output/pullComp_Pruning_VS_NoLightPruning
~~~
Now the last thing that needs to be done is to move all the pull comparisons to one place and then make them such that everyone can access them.
~~~
mv output/pullComp_* output/pullComps
cd output/pullComps
python "/afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2/WSMakerCore/macros/webpage/createHtmlOverview.py"
cd ..
cp -r pullComps ~/www
~~~
# Running THE FINAL (seriously this is the last time I am doing this). Luckily for me i'm only doing the postfit plots.

This time instead of checking out the master, we are getting a specific tag. Also ROOT has been updated but we are using the old one still. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
source getMaster.sh 00-03-02bis
~~~
## Running VV and CBA analysis plots
The inputs used are post-splitting and post-processed. They are available here: /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2020Feb04_VVunblindingClosure/SMVHVZ_2019_MVA_mc16ade_v03_STXS_GSC_TTBARBC-0LEP_postprocessed

For details on how to do this run see the Earlier Section entitled: Running Official 0L Plots for Cut-Based(CB) and Diboson(VV) Analyses. Remebering that you will need to create a new input configs file.

## Running MVA postfit plots
This is going to be a bit more involved than the last time. Luca has created some inputs with all the variables but without the post processing. 
>   /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_allMVAVariables/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15allVarsZr32-24-Reader-Resolved-03.root

They need to be split and then post-processed. First we need a new inputConfigs file
~~~
vim/inputConfigs/SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS.txt
~~~
>   ADD 0L All variables                                                                                                     
>   >   CoreRegions                                                                                                         
>   >   ZeroLepton /ZeroLep/r32-15_allMVAVariables/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15allVarsZr32-24-Reader-Resolved-03.root binMV2c10B1,binMV2c10B2,binMV2c10B1B2,dEtaBB,dPhiVBB,dRBB,dPhiBB,HT,pTB1,pTB2,pTJ3,mBB,mBBJ,MET,softMET                       
~~~
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS
~~~
Once this is done the inputs need to be post-processed. However the standard post-processing happens only on the mbb & MET in 0L. We need to put the post-processing on all the variables. 
~~~
vim scripts/RemoveNormImpact.py
~~~
>   ADD full list of 0L variables to process (L181)                                                                           
>   >   variables_0lep = ["mva", "mvadiboson", "mBB", "MET"] -> variables_0lep = ["mva", "mvadiboson", "mBB", "MET", "binMV2c10B1","binMV2c10B2","binMV2c10B1B2","dEtaBB","dPhiVBB","dRBB","dPhiBB","HT","pTB1","pTB2","pTJ3","mBBJ","softMET"]

Then run the script to post-process the variables. WARNING. This takes some considerable time.
~~~
python scripts/RemoveNormImpact.py --indir inputs/SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS/ --outdir SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed/
~~~
Now we will run a postfit MVA to generate the postfit workspaces.
~~~
vim scripts/analysisPlottingConfig.py 
~~~
>    CHANGE plotting configuration to mva default (~L10)                                                                   
>   >  vh_fit=True                                                                                                           
>   >  vh_cba=False                                                                                                          
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L15)                                                                                    
>   >  doPostFit = True                                                                                                       

>    CHANGE alternative analysis to False (~L17)                                                                            
>   >  doCutBase = False                                                                                                     
>   >  doDiboson = False                                                                                                     

>    CHANGE the default channel(s) to run on (~L51)                                                                          
>   >  channels = ["0"]                                                                                                     

>    CHANGE to construct Asimov dataset using central values from fit to data (~L62)                                         
>   >  doExp=1 -> doExp=0                                                                                                    

>    CHANGE what you want to run in the 0L standalone fit (~L69)                                                             
>   >  createSimpleWorkspace = True                                                                                          
>   >  runPulls = False                                                                                                       
>   >  runBreakdown = False                                                                                                   
>   >  runRanks = False                                                                                                       
>   >  runLimits = False                                                                                                    
>   >  runP0 = False                                                                                                        
>   >  runToyStudy = False                                                                                                 

>    ADD no additional debug plots for shapes (~L72)                                                                
>   >  doplots = False

>    CHANGE postfit variables of interest (~L92)                                                                            
>   >  vs2tag = ['pTV','MET','pTB1','pTB2','mBB','dRBB','dEtaBB','dPhiVBB','dEtaVBB','MEff','MEff3','dPhiLBmin','mTW','mLL','dYWH','Mtop','pTJ3','mBBJ','mBBJ3','METSig'] -> vs2tag =  ["mBB", "MET", "binMV2c10B1","binMV2c10B2","binMV2c10B1B2","dEtaBB","dPhiVBB","dRBB","dPhiBB","HT","pTB1","pTB2","pTJ3","mBBJ","softMET"]
~~~
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA

mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_binMV2c10B1B2_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1B2 
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_binMV2c10B1_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_binMV2c10B2_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B2
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_dPhiVBB_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiVBB 
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_softMET_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_dEtaBB_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dEtaBB
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_dPhiBB_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiBB
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_mBBJ_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBBJ
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_pTB1_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB1
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_pTB2_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB2
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_pTJ3_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTJ3
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_dRBB_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dRBB
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_MET_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-MET
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_mBB_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBB
mv SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_HT_STXS_FitScheme_1 140ifb-0L-ade-STXS-baseline-MVA-PostFit-HT
~~~
Want to now make the zero lepton postfit plots by comparing them to an existing workspace. We want to do this twice. Once when compared against the zero lepton standalone fit, and once when compared against the combined 012L fit.
~~~
mkdir 140ifb-012L-ade-STXS-baseline-MVA
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-05/comb/vh-mva/
cp -r fccs /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/outputs/140ifb-012L-ade-STXS-baseline-MVA

python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1B2
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B2
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dEtaBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiVBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dRBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-HT
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBBJ
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-MET
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB1
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB2
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTJ3
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET

mkdir output/012L-0L-Postfit
mv output/140ifb-0L-ade-STXS-baseline-MVA-PostFit-* output/012L-0L-Postfit

python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1B2
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B2
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dEtaBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiVBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-dRBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-HT
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBBJ
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-MET
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB1
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB2
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTJ3
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET

mkdir output/0L-0L-Postfit
mv output/140ifb-0L-ade-STXS-baseline-MVA-PostFit-* output/0L-0L-Postfit
~~~
Finally we need to put these plots somewhere where people can easily access them.
~~~
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-05/l0/vh-mva/
mkdir 012L-0L-Postfit 0L-0L-Postfit 

cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1/plots/postfit 012L-0L-Postfit/Postfit-binMV2c10B1 
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1B2/plots/postfit 012L-0L-Postfit/Postfit-binMV2c10B1B2 
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B2/plots/postfit 012L-0L-Postfit/Postfit-binMV2c10B2 
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dEtaBB/plots/postfit 012L-0L-Postfit/Postfit-dEtaBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiBB/plots/postfit 012L-0L-Postfit/Postfit-dPhiBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiVBB/plots/postfit 012L-0L-Postfit/Postfit-dPhiVBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dRBB/plots/postfit 012L-0L-Postfit/Postfit-dRBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-HT/plots/postfit 012L-0L-Postfit/Postfit-HT
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBB/plots/postfit 012L-0L-Postfit/Postfit-mBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBBJ/plots/postfit 012L-0L-Postfit/Postfit-mBBJ
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-MET/plots/postfit 012L-0L-Postfit/Postfit-MET
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB1/plots/postfit 012L-0L-Postfit/Postfit-pTB1
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB2/plots/postfit 012L-0L-Postfit/Postfit-pTB2
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTJ3/plots/postfit 012L-0L-Postfit/Postfit-pTJ3
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET/plots/postfit 012L-0L-Postfit/Postfit-softMET


cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1/plots/postfit 0L-0L-Postfit/Postfit-binMV2c10B1 
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B1B2/plots/postfit 0L-0L-Postfit/Postfit-binMV2c10B1B2 
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-binMV2c10B2/plots/postfit 0L-0L-Postfit/Postfit-binMV2c10B2 
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dEtaBB/plots/postfit 0L-0L-Postfit/Postfit-dEtaBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiBB/plots/postfit 0L-0L-Postfit/Postfit-dPhiBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dPhiVBB/plots/postfit 0L-0L-Postfit/Postfit-dPhiVBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-dRBB/plots/postfit 0L-0L-Postfit/Postfit-dRBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-HT/plots/postfit 0L-0L-Postfit/Postfit-HT
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBB/plots/postfit 0L-0L-Postfit/Postfit-mBB
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-mBBJ/plots/postfit 0L-0L-Postfit/Postfit-mBBJ
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-MET/plots/postfit 0L-0L-Postfit/Postfit-MET
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB1/plots/postfit 0L-0L-Postfit/Postfit-pTB1
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTB2/plots/postfit 0L-0L-Postfit/Postfit-pTB2
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-pTJ3/plots/postfit 0L-0L-Postfit/Postfit-pTJ3
cp -r /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET/plots/postfit 0L-0L-Postfit/Postfit-softMET
~~~
## Changing the binning on the softMET postfit plots.
~~~
vim src/binning_vhbbrun2.cpp
~~~
>    ADD binning stipulation on the softMET postfits where the others are (~L302)                                             
>   >    if (c(Property::dist) == "softMET" ) res = {1}; //To be tested                                                       

>    ADD range change for softMET postfits plots where the others are (~L413)                                             
>   >    if (c(Property::dist) == "softMET" ) changeRangeImpl(h,0,60);                                                    

Re-run the softMET plots in their new folder(s)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/
source setup.sh
cd build
cmake ..
make -j8
cd ..

python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET

mv output/SMVHVZ_2019_MVA_mc16ade_0L_v03_AllVars_STXS_postprocessed.140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET_0_mc16ade_Systs_softMET_STXS_FitScheme_1 output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET
cp -r output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET-012

python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-012L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET-012

python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET
~~~
Ensure that all the plots are up to scratch.
~~~
imgcat output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET/plots/postfit/*.png
imgcat output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET-012/plots/postfit/*.png
~~~
If you are not happy with the binning remove the softMET plot workspace and make them again by starting from this section again.
~~~
rm -rf output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET
rm -rf output/140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET-012
>   ## Changing the binning on the softMET postfit plots.
~~~
Once you are happy with the plots and they do not need to be re-done.
~~~
cd output/012L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET/plots/
mv postfit postfitOLD
mv ../../../140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET-012/plots/postfit .
cp -r postfit/* /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-05/comb/vh-mva/postfit_all/l0

cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Feb2020/output/0L-0L-Postfit/140ifb-0L-ade-STXS-baseline-MVA-PostFit-softMET/plots/
mv postfit postfitOLD
mv ../../../140ifb-0L-ade-STXS-baseline-MVA-postfitsoftMET/plots/postfit .
cp -r postfit/* /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/outputs/2020-02-05/l0/vh-mva/0L-0L-Postfit/Postfit-softMET/
~~~
