# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about generating and creating ntuples from VHbb CxAOD's and testing them. #

## "VHbb 2 Lepton Reader Inputs" ##

Last Edited: 17-06-2018
-------------------------------------------------------------------------------

# Setup

The first thing that you want to do is ensure that you are up to date with the framework. To do this you should have access to the latest branch/tag of the CxAOD Framework. It's good to check online
(https://gitlab.cern.ch/CxAODFramework/FrameworkSub/tree/master/bootstrap) but as it's said in VHBB_2Lep_Reader_Inputs.md there is a process of continual updates.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS &&lsetup git
cp /afs/cern.ch/user/a/abuzatu/work/public/BuzatuAll/BuzatuATLAS/CxAODFramework/getMaster.sh .
source getMaster.sh r31-10 CxAODFramework_tag_r31-10 1 1
> source getMaster.sh origin/master CxAODFramework_branch_21.2.33 1 1 [for a branch]
~~~

## Reconfiguration of necessary files.
So the standard variable settings in the newly obtained directory are not necessary going to be the same as what you want. All the important files will need to be checked.

### FrameworkExe/scripts/SubmitReader.sh
>   DRIVER = "LSF" (L55)                                                                                        
>   JOBSIZELIMITMB="-1"  (L58)                                                                                  
>   NRFILESPERJOB="100"  (L59)                                                                               

### FrameworkExe/data/framework-read-automatic.cfg
>   vector<string> TMVATrainingTool.InputVarNames = pTV mLL MET dRBB mBB pTB1 pTB2 pTJ3 mBBJ dPhiVBB dEtaVBB (L80)           
>   string bQueue              = 8nh (L91)

### CxAODReader_VHbb/Root/AnalysisReader_VHbb2Lep.cxx
>   > L1062                                                                                                
>   m_histSvc->BookFillHist("pTV", 20, 0, 500, m_MVAInputs2l->pTV, m_weight);                                            
>   m_histSvc->BookFillHist("mLL", 20, 80, 100, m_MVAInputs2l->mLL, m_weight);                                            
>   m_histSvc->BookFillHist("MET", 25,0, 500, m_MVAInputs2l->MET, m_weight);                                                
>   m_histSvc->BookFillHist("dRBB", 20, 0, 5, m_MVAInputs2l->dRBB, m_weight);                                               
>   m_histSvc->BookFillHist("mBB", 25, 0., 500., m_MVAInputs2l->mBB, m_weight);
>   m_histSvc->BookFillHist("pTB1", 50, 0., 500., m_MVAInputs2l->pTB1, m_weight);                                         
>   m_histSvc->BookFillHist("pTB2", 50, 0., 500., m_MVAInputs2l->pTB2, m_weight);                                            
>   m_histSvc->BookFillHist("pTJ3", 50, 0., 500., m_MVAInputs2l->pTJ3, m_weight);                                         
>   m_histSvc->BookFillHist("mBBJ", 15, 0, 750, m_MVAInputs2l->mBBJ, m_weight);                                  
>   m_histSvc->BookFillHist("dPhiVBB", 15, 1.5, 3.15, m_MVAInputs2l->dPhiVBB, m_weight);                                
>   m_histSvc->BookFillHist("dEtaVBB", 20, 0, 5, m_MVAInputs2l->dEtaVBB, m_weight);                             
>   }                                                                               
               
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/build/
setupATLAS
make -j6
~~~
# Cutflow Challenge

One of the things under the jurisdiction of VHbb analysis NTuples activities is to check the number of events in the cutflow of the analysis (See https://twiki.cern.ch/twiki/bin/view/AtlasProtected/HiggsbbCxAODproduction) . Basically you run the reader over a standard sample.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/
source source/FrameworkExe/scripts/setupLocal.sh
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10/ Cutflow 2L a 31-10 1
~~~
For 2 lepton R31, the files to run over are found on the spreadsheets that circulate within the group.
~~~
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10/HIGG2D4_13TeV/CxAOD_31-10_a/data16/group.phys-higgs.data16_13TeV.00300800.CAOD_HIGG2D4.r9264_p3083_p3372.31-10_CxAOD.root

/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10/HIGG2D4_13TeV/CxAOD_31-10_a/qqZllHbbJ_PwPy8MINLO/group.phys-higgs.mc16_13TeV.345055.CAOD_HIGG2D4.e5706_e5984_s3126_r9364_r9315_p3374.31-10_CxAOD.root
~~~
The data file to run over is contained alongside a list of other data16 sample folders, and under the current automatic reader configuration will run over all of them. So the thing to do is to create a cutflow-dat16 folder parallel to the data16 directory with this sample in.
~~~
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10/HIGG2D4_13TeV/CxAOD_31-10_a/
mkdir cutflow-dat16
cp -r data16/group.phys-higgs.data16_13TeV.00300800.CAOD_HIGG2D4.r9264_p3083_p3372.31-10_CxAOD.root cutflow-dat16/
~~~

Edit the file that steers the Reader
~~~
vim /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/source/FrameworkExe/data/submitReader.sh
~~~
> CHANGE variables
>   > SAMPLES+="cutflow-dat16 qqZllHbbJ_PwPy8MINLO" #R31 cutflow ~L68

Some metadata for the analysis. It is performed on MC16a, using the tag 31-10. Now running the below commands should successfully run the cutflow over the grid.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10
source source/FrameworkExe/scripts/setupLocal.sh
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10 Cutflow 2L a 31-10 1

~~~
If this has successfully run. There should be histgrams in the Cutflow/Reader_2L_MVA_31-10_a/fetch/ directory. All that is needed to be done now is for these to be hadd-ed. Then once the hadd-ed files are complete they can be opened in the TBrowser and the numbers in the Cutflow-> Nominal -> CutsMVANoWeight are the cutflow results.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/Cutflow/Reader_2L_MVA_31-10_a/fetch/
setupATLAS && lsetup root
hadd QQZllHBBJ-CUTFLOW.root hist-qq*
hadd DATA16-CUTFLOW.root hist-cut*
root -l QQZllHBBJ-CUTFLOW.root
root -l DATA16-CUTFLOW.root
-> TBrowser a
~~~
If you are happy that the numbers shown in the cutflow match those that previous people have seen. Then create an input file for the DynPlottingTool.
~~~
cd ..
vim lofinputs.txt
>   > /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/Cutflow/Reader_2L_MVA_31-10_a/fetch/DATA16-CUTFLOW.root
>   > /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/Cutflow/Reader_2L_MVA_31-10_a/fetch/QQZllHBBJ-CUTFLOW.root
~~~

## Plotting (DynPlottingTool)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git
vim DynPlottingTool/src/Plotmaker.cxx
~~~
Now make the package and run it
~~~
cd DynPlottingTool
lsetup root
make clean
make -j6
./PlotMaker -x false -l /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/Cutflow/Reader_2L_MVA_31-10_a/lofinputs.txt -a SM -q SM -d data -s qqZllHbbJ -w 1 -f 1.0 -v mBB/4-400,pTV/1-500,mLL/1-110,MET,dRBB/4,pTB1/2-400,pTB2/2-400,pTJ3/4,mBBJ/4,dPhiVBB,dEtaVBB -o VHbb2Lep_R31_Cutflow -b 1 -e ggZvvH125 2>&1 | tee VHbb2Lep_DYN.log
~~~
Then check that the kinematic variables are behaving themselves
~~~
imgcat VHbb2Lep_R31_Cutflow/2tag2jet_75_150ptv_SR_mBB.png
~~~

# Run with Systematics

## Warning this is a meaty boi so you need a workspace with 100GB+ space available.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10
setupATLAS && lsetup git
~~~
To calibrate this though, you need to look over the file submitReader.sh.
~~~
vim source/FrameworkExe/scripts/submitReader.sh
~~~
>   MODELTYPE="MVA" # MVA, SM, AZh etc
>   DRIVER="LSF"
>   JOBSIZELIMITMB="-1"
>   NRFILESPERJOB="64"

> COMMENT IN SAMPLE FILES
>   > SAMPLES=""
>   > #SAMPLES+="cutflow-dat16 qqZllHbbJ_PwPy8MINLO" #R31 cutflow
>   >
>   > if [${MCTYPEs} == a]; then
>   > SAMPLES+=" data15 data16" # old data
>   > fi
>   > if [${MCTYPEs} == d]; then
>   > SAMPLES+=" data17 " # new data
>   > fi
>   >
>   > SAMPLES+=" qqZllHbbJ_PwPy8MINLO qqWlvHbbJ_PwPy8MINLO ggZllHbb_PwPy8" # VH + extra jets, V->leptons, H->bb: Signal
>   >
>   > SAMPLES+=" WqqWlv_Sh221 ggWqqWlv_Sh222" # WW
>   > SAMPLES+=" WqqZvv_Sh221 WqqZll_Sh221 WlvZqq_Sh221" # WZ
>   > SAMPLES+=" ZqqZvv_Sh221 ZqqZll_Sh221 ggZqqZvv_Sh222 ggZqqZll_Sh222" # ZZ
>   >
>   > SAMPLES+=" stops_PwPy8 stopt_PwPy8 stopWt_PwPy8" # stop
>   > SAMPLES+=" ttbar_dil_PwPy8" # ttbar dilepton
>   > #SAMPLES+=" ttbar_nonallhad_PwPy8" # ttbar nonallhad
>   >
>   > SAMPLES+=" WenuB_Sh221 WenuC_Sh221 WenuL_Sh221 Wenu_Sh221 WmunuB_Sh221 WmunuC_Sh221 WmunuL_Sh221 Wmunu_Sh221 WtaunuB_Sh221 WtaunuC_Sh221 WtaunuL_Sh221 Wtaunu_Sh221" # W+jets
>   > SAMPLES+=" ZeeB_Sh221 ZeeC_Sh221 ZeeL_Sh221 Zee_Sh221 ZmumuB_Sh221 ZmumuC_Sh221 ZmumuL_Sh221 Zmumu_Sh221 ZtautauB_Sh221 ZtautauC_Sh221 ZtautauL_Sh221 Ztautau_Sh221 ZnunuB_Sh221 ZnunuC_Sh221 ZnunuL_Sh221 Znunu_Sh221" # Z+jets
~~~
vim source/FrameworkExe/data/framework-read-automatic.cfg
~~~
> CHANGE
>   > string bQueue              = 2nd (L90)
>   > bool nominalOnly = true -> false (L212)
>   > bool autoDiscoverVariations = false -> true (L214)

Then when you are all set, build and run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/build
setupATLAS
make -j6
cd ..
source source/FrameworkExe/scripts/setupLocal.sh
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10 /eos/user/d/dspiteri/R31-10_Systs 2L a,d 31-10 1
~~~

Now make the package and run it
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git
cd DynPlottingTool
lsetup root
make clean
make -j6
cd ..
./PlotMaker -x false -l /eos/user/d/dspiteri/R31-10_Systs/Reader_2L_MVA_31-10_a/lofinputs.txt -a SM -q SM -d data -s qqZllHbbJ -w 1 -f 1.0 -v mBB/4-400 -o VHbb2Lep_R31_Systs -b 1 -e ggZvvH125 2>&1 | tee VHbb2Lep_Systs_DYN.log
~~~

# 32-15 Running
Correct as of 19/05/2019

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
~~~
To create a test with only the MC16a and MC16d signal samples run 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalResolved 2L a,d VHbb MVA D1 32-15 signal none 1
cd SignalResolved_TEST/Reader_2L_32-15_a_MVA_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../Reader_2L_32-15_d_MVA_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..
~~~
To create a full nominal set using just MC16a run. The resulting files are more numerous so they have to be hadded in stages. Also the name matches the convention used by WSMaker (more on that later). 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 Full_a_Resolved 2L a VHbb MVA D1 32-15 none none 1

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15/run/Full_a_Resolved/Reader_2L_32-15_a_MVA_D1/
hadd g_histograms hist-g*
hadd q_histograms hist-q*
hadd t_histograms hist-t*
hadd W_histograms hist-W*
hadd Z_histograms hist-Z*
hadd LimitHistograms.VH.llbb.13TeV.mc16a.Glasgow.v1.root *_histograms
rm *_histograms
~~~

## Troubleshooting
Occasionally samples will fail and you will not notice until after you have hadded and started your analysis. To prevent future upset before you hadd the files it's probably best that you check your outputs for defects like root files with no keys. Luckily there is a script to do such a thing!!
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15/run/Full_a_Resolved/
Reader_2L_32-15_a_MVA_D1/
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_MVA_D1
~~~
>   > TFile::Init:0: RuntimeWarning: file /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15//run/Full_a_Resolved/Reader_2L_32-15_a_MVA_D1/fetch/hist-ZmumuB_Sh221-11.root has no keys
>   > TFile::Init:0: RuntimeWarning: file /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15//run/Full_a_Resolved/Reader_2L_32-15_a_MVA_D1/fetch/hist-ZeeB_Sh221-8.root has no keys

Once you have found problematic files you can resubmit those jobs, but you need to go to submit/segments and find the job number for the defunct file(s)
~~~
vim Reader_2L_32-15_a_MVA_D1/submit/segments
~~~
Oh look it's 43 and 91! Now enter the directory and re-run on direct
~~~
cd Reader_2L_32-15_a_MVA_D1
./submit/run 43 
./submit/run 91
~~~

# Resolved WSMaker 
Much of the help is from here
```
https://gitlab.cern.ch/atlas-physics/higgs/hbb/wsmaker_boostedvhbb/blob/master/boostedVHbbMiniTutorial.md
```
The Workspace maker or WSMaker is a tool which can be used to create and manipulate workspaces from VH input files.
To setup and run do
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git 
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git
cd WSMaker_VHbb
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
~~~
The configurations for the inputs (i.e what input files should be used for each lepton
channel) are stored in inputConfigs.
~~~
ls ../inputConfigs
~~~
input config files have the form
SMVHVZ_[Analysis Milestone]_[boosted/Resolved]_[MCperiod(s)]_[version]_[fixes]_[STXS].txt
SMVHVZ_Summer18_MVA_mc16a_v07_fixed2_STXS.txt
SMVHVZ_Summer18_CUT_mc16d_v07_fixed3.txt

At this stage you are faced with two choices. You can a) either try to use a standardised input created by someone else or b) you can use the WSMaker with your own inputs

## Standardised inputs for WSMaker 

Since I will use the WSmaker on the 31-15 samples that I just run on for the resolved analysis I need a sample set that has the _MVA_ tag to represent the resolved analysis and the MC generation period _MC16a_. The latest version with all this is SMVHVZ_Summer18_MVA_mc16a_v07_fixed3.txt. This file will contain 0,1 and 2L samples so one should delete the ones that are not needed. The file name is a path on eos and when you are splitting you will need to provide the rest of the absolute path. 
~~~
vim ../inputConfigs/SMVHVZ_Summer18_MVA_mc16a_v07_fixed3.txt
~~~
>   Delete 0L and 1L lines (L1-3)
~~~
SplitInputs -v SMVHVZ_Summer18_MVA_mc16a_v07_fixed3 -inDir /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/Summer2018
~~~
The split files will then be stored in a folder in inputs/SMVHVZ_Summer18_MVA_mc16a_v07_fixed3

Then to run things you need to open launch_default_jobs.py
~~~
vim scripts/launch_default_jobs.py
~~~
>   CHANGE variable version (everything after the '_' after the MC period) L13
>   >     version = v07_fixed3
>   CHANGE other input variables
>   >     doCutBase = False (L17)
>   >     doCutBasePlots = False # switch to get convergence on the for plots
>   >     doDiboson = False (L19)
>   >     Add1LMedium = False (L20)
>   >     doTTbarDataDriven2L = False (L21)
>   >     do_mbb_plot = False (L22)
>   >     doSTXS = False (L26) # options for STXS
>   >     doExtra = False (L35) # options for HL-LHC
>   SET channels, MCTypes, and systematics to the ones you want to run over
>   >     channels = ["2"] # channels = ["0", "1", "2", "012"] (L41)
>   >     MCTypes = ["mc16a"] # MCTypes = ["mc16ad","mc16d","mc16a"] (L43)
>   >     syst_type = ["MCStat"] # Options = ["Systs", "StatOnly", "MCStat"] (L46)
MCstat = adds weights to rare MC processes vs STATonly = MC has no uncertainty
>   CHANGE run options 
>   >     signalInjection = 0 # DEFAULT 0: no injection (L49)
>   >     doExp = "0" # "0" to run observed, "1" to run expected only (L50)
>   >     run_on_batch = False (L53)
>   >     createSimpleWorkspace = True (L56)
>   >     runPulls = False (L57)
>   >     runBreakdown = False (L58)
>   >     runRanks = False (L59)
>   >     runLimits = False (L60)
>   >     runP0 = False (L61)
>   >     runToyStudy = False (L62)   
>   >     doplots = False # Turns on additional debug plots (L65)
>   >     anaType = "MVA" #overwrite for cut-based (L68)

Once a configuration is chosen, one just has to type
~~~
python launch_default_jobs.py myOutputTag
~~~
That will send jobs in parallel on the local machine. The config files will appear in configs/, the workspaces will appear in workspaces/, and the outputs will appear in... many places depending on what is run: plots/, fccs/, logs/, possibly root-files/... There is an open issue to improve on that: WSMaker#18

## Personal inputs for WSMaker 
Essentially you will follow similar instructions to the above but you have to create your own file pointing to the right directory. So starting from the same point. Here you will notice that there are three parts to each line. The first part selects the VHbb analysis, the second is the latter part of the path to the histograms file which includes the file name itself, and the last part denotes the variables you want to run over. 
~~~
cd ../inputConfigs
vim SMVHVZ_Summer18_MVA_mc16a_v07_Dwayne2L.txt
~~~
The name of this file is relatively arbitary. If you decide to go against this naming convention then you have to edit another line later on. 

>   ADD these lines to the newly created file.
>   >     CoreRegions
>   >     TwoLepton CxAODFramework_tag_r32-15/run/Full_a_Resolved/Reader_2L_32-15_a_MVA_D1/LimitHistograms.VH.llbb.13TeV.mc16a.Glasgow.v1.root mva,mvadiboson,mBBMVA

Then with that done, to generate the inputs you have to run in the main WSMaker_VHbb folder
~~~
cd ..
SplitInputs -v SMVHVZ_Summer18_MVA_mc16a_v01_Dwayne2L -inDir /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15/run
~~~
If this command works outputs should appear in the directory inputs/SMVHVZ_Summer18_MVA_mc16a_v01_Dwayne2L.

Then to run things you need to open launch_default_jobs.py.  All the changes listed above still apply here. But in addition to this you will need to change the version.
~~~
vim scripts/launch_default_jobs.py
~~~
>   CHANGE variable version (everything after the '_' after the MC period) (L13)
>   >     version = v01_Dwayne2L

If you named the input configuration file unconventionally, put the full name of that file in Input Version 
>   CHANGE name of the InputVersion (L254)
>   >     InputVersion = [insert name here]
~~~
python scripts/launch_default_jobs.py 31-15_MC16a_Full
~~~
Following on from the point I ran my own configuration. I want to see some plots 
~~~
cd output/SMVHVZ_Summer18_MVA_mc16a_v01_Dwayne2L.31-15_MC16a_Full_fullRes_VHbb_31-15_MC16a_Full_2_mc16a_MCStat_mva/plots
~~~
