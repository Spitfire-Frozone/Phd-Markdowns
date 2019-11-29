# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 28-11-2019
-------------------------------------------------------------------------------
## Milestone 2 Fits on latest inputs

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
>    0-lepton :/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root
>    1-lepton : /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/OneLep/r32-15_postMS2_20191128/LimitHistograms.VHbb.1Lep.13TeV.mc16ade.UCL.32-15.root
>    2-lepton : /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/TwoLep/r32-15_postMS2_20191124/v1/fullSysts/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.Kyoto.r32-15_postMS2_20191124.v1.root

The first thing to do is to check the yields against the previous inputs used. 
~~~
vim scripts/InputsCheck_VHbb.py
~~~
>    CHANGE Inputs to compare and the common area where they are kept (~L28-30)
>   >  eos_dir='/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/'                             
>   >  IC.input_ref  = eos_dir+'ZeroLep/r32-15_PCBT_20191113/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford-newExt-PCBT.32-15.root'
>   >  IC.input_test = eos_dir+'ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root'

>    CHANGE the Lepton channel to 0 lepton.
>   >  IC.lep = 'L0'
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
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MVA
~~~
Create a website so that everyone can see your plots
~~~
cd output
python "../WSMakerCore/macros/webpage/createHtmlOverview.py"
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
>   >  doExp=0 -> doExp=1  

>    CHANGE what you want to run in the 0L standalone fit (~L69)                                                          
>   >  runPulls = False                                                                                                 

>    CHANGE postfit variables of interest (~L92)
>   >  vs2tag = ['pTV','MET','pTB1','pTB2','mBB','dRBB','dEtaBB','dPhiVBB','dEtaVBB','MEff','MEff3','dPhiLBmin','mTW','mLL','dYWH','Mtop','pTJ3','mBBJ','mBBJ3','METSig'] -> vs2tag =  ['pTV','MET']
~~~
Then once you are ready you can run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_Milestone2
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-postfitVars
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-postfitVars_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitVars_0_mc16ade_Systs_MET_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-mBB
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-postfitVars_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitVars_0_mc16ade_Systs_pTV_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MET
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-mBB


### Full 0L Fit
Now we have to run the full nominal standalone 0L fit.
