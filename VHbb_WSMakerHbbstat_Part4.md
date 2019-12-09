# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 02-12-2019
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
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA
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
>   >  doExp=0 -> doExp=1                                                                                                    

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
python scripts/launch_default_jobs.py 140ifb-0L-ade-STXS-baseline-MVA-postfitVars
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-postfitVars_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitVars_0_mc16ade_Systs_MET_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-MET
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-baseline-MVA-postfitVars_fullRes_VHbb_140ifb-0L-ade-STXS-baseline-MVA-postfitVars_0_mc16ade_Systs_mBB_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-baseline-mBB
python WSMakerCore/scripts/doPlotFromWS.py -m 125 -p 3 -f 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-baseline-mBB
~~~

### Full 0L Fit
Now we have to run the full nominal standalone 0L fit. This may take some time. 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L15)                                                                                      
>   >  doPostFit = False                                                                                                       

>    CHANGE to construct Asimov dataset using central values from fit to data (~L62)                                           
>   >  doExp=1                                                                                                    

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
python WSMakerCore/scripts/makeNPrankPlots.py 140ifb-0L-ade-STXS-baseline-MVA-Full
imgcat output/140ifb-0L-ade-STXS-baseline-MVA-Full/pdf-files/pulls_SigXsecOverSM_125.pdf
~~~

# Repeating for slightly newer inputs
Post-processed inputs ot change the PtV normalisation in 0L and 1L have been produced and they are here:
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
The new fit has waay more B-tagging pulls, but what makes a pull.

So the indices in the pulls usually refer to an Eigenvector decomposition. In the case of the JET systematics for example, you first group them into physical categories, like Modelling, Statistical Uncertainties, and in each of these categories you do the decomposition. 

The decomposition is basically finding  a linear combination of uncertainties. If you see each systematic variation as being correlated with others than you can think of the decomposition as dioganolizing this matrix(1**). then your new "decomposed" uncertainties (or eigenvectors) are a linear combination of the ones you put in. You then label them according to their impact (1 being the one with the most impact, ...). It is important to note that it is no longer so clear what they actually are. The input systematic uncertainties are of course meaningful in the sense that one corresponds maybe to the difference between MC generator 1 and MC generator 2 but after the decomposition they are linear combinations of each other. Then you usually say I take the N most important ones separately and the ones with i > N I add in quadrature to form the N+1 NP. 

(1**)the matrix that is used for the eigenvector problem. It starts from the construction of the covariance matrix corresponding to each source of uncertainty, and then sums these covariance matrices to obtain the total covariance matrix. Being a symmetric, positive-definite matrix this can be considered as an eigenvalue problem. The eigenvectors that "solve" this problem can be seen as "directions" in which to carry out independent variations. The sizes of the variations are given by the square root of the corresponding eigenvalues.
 
## De-correlation of everything for B-tagging
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
Given that the B21 in ptV, nJets and in SRCR are also inconclusive, we will also fully de-correlate Light_0 and B_0
~~~
vim src/systematicslistsbuilder_vhbbrun2.cpp
~~~
>    ADD splitting of Light_0 systematic instead of B_21 (~L566)                                                               
>   >        if (sysname == "Eigen_B_1") ... -> if (sysname == "Eigen_Light_0")     
>   >        [ [ [ AND ] ] ]
>   >        if (sysname == "Eigen_B_1") ... -> if (sysname == "B_0")   
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
mv output/SMVHVZ_2019_MVA_mc16ade_v03_STXS.140ifb-0L-ade-STXS-CRDummySysts-MVA_fullRes_VHbb_140ifb-0L-ade-STXS-CRDummySysts-MVA_0_mc16ade_Systs_mva_STXS_FitScheme_1_QCDUpdated_PDFUpdated_dropTheryAccUpdated output/140ifb-0L-ade-STXS-CRDummySysts-MVA 

python WSMakerCore/scripts/comparePulls.py -w 140ifb-0L-ade-STXS-baseline-MVA 140ifb-0L-ade-STXS-CRDummySysts-MVA -n -a 5 -l Nominal ExtraSysts
mv output/pullComparisons output/pullComp_Nominal_VS_AdditionalCRSysts
