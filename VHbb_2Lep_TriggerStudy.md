# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about the testing new trigger regimes for the 2 Lepton Analysis for VHbb. #

## VHbb 2 Lepton Trigger Study ##
===============================================================================
Last Edited: 02-07-2019
-------------------------------------------------------------------------------

# Setup Script
Search https://gitlab.cern.ch/CxAODFramework/FrameworkSub to see what the latest released version is.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
cp /afs/cern.ch/user/a/abuzatu/work/public/BuzatuAll/BuzatuATLAS/CxAODFramework/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_branch_21.2.46 1 1
> source getMaster.sh r31-10 CxAODFramework_tag_r31-10 1 1 [for a tag]
~~~
If you install 32-04 you should get the following message -
~~~
Finished compilation, so doing source x86_64-slc6-gcc62-opt/setup.sh
Going to the run folder
/afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-04/run
You can now test the Maker or Reader
For Maker, you can test in one go for all possible cases with
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh
To make this easier for you to run many times as you develop, we create an alias t to this command
COMMAND=alias t="source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh testLocallyAsInCIPipelineTasks.txt"

Careful as this will launch many jobs in parallel, and in lxplus even one job is slow, all jobs is hopeless!
To see which jobs will be run, open this file
emacs -nw ../source/CxAODOperations_VHbb/data/DxAOD/info/testLocallyAsInCIPipeline.txt
Comment out those that you do not want to be run, so that you can test only the one you want
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh testLocallyAsInCIPipelineTasks.txt
If you are on lxplus, there would be too many jobs to run in parallel, so let's run a reduced number
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh testLocallyAsInCIPipelineTasksReduced.txt
and the new test samples with patch for MET Trigger skimming in 0L and 1L
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh testLocallyAsInCIPipelineTasks2.txt

But you can also run on just one sample locally directly with submitMaker.sh
Run without any arguments, and it will show you the arguments to run out of the box with copy/paste from the command line arguments shown
source ../CxAODOperations_VHbb/scripts/submitMaker.sh
To make this easier for you to run many times as you develop, we create an alias m to this command
COMMAND=alias m="source ../CxAODOperations_VHbb/scripts/submitMaker.sh"
After every test, if needed, you can/shoud run ./clean.sh to remove the files produced (Maker, .root, configs, logs).
To submit to the grid, from the same place, with other examples from the submitMaker.sh

You can also run the Reader from this folder, either locally or on a batch system
source ../CxAODOperations_VHbb/scripts/submitReader.sh
To make this easier for you to run many times as you develop, we create an alias r to this command
COMMAND=alias r="source ../CxAODOperations_VHbb/scripts/submitReader.sh"

CxAODFramework for VHbb (Maker+Reader) installed. Good luck exploring CxAOD!
~~~
You can test the checkout with the submission of some trial jobs by doing
~~~
source ../source/FrameworkExe/scripts/testLocallyAsInCIPipeline.sh
-   OR   -
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh
~~~
Then once you get this to work, everytime you log in you can do.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.46
source source/FrameworkExe/scripts/setupLocal.sh
~~~
To submit to the grid, from this point it's best to close the terminal used to get the branch, open a new one, cd to the folder that contains the build, source and run directories, and submit in one go with
~~~
source source/FrameworkExe/scripts/submitToGrid.sh
~~~
which will tell you the different arguments it wants. However sometimes you want to get rid of the build folder and start again in case of errors. If this happens just

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/build
rm -rf * (be very very careful)
cd ..
setupATLAS && asetup 21.2.27,AnalysisBase
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd build
cmake ../source
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
~~~

#  Feasability
The main idea of this study is to see how much of a gain one gets in the 2L channel by introducing a different trigger strategy. At a zeroth-order check what one can easily do is re-run the well established cutflow for the old and new trigger strategies and see if there are any yield changes.

The problem with this is that there isn't much MET in the signal region where the cutflow is based, so the but there should be some visible changes.

For 2 lepton R31, the directories to run over are found on the spreadsheets that circulate within the group. I will use the 31-10 samples but with the latest relase of the framework at this point which happens to be 31-24 (32-07 and 32-15 later on).

>  /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10/HIGG2D4_13TeV/CxAOD_31-10_a/cutflow-dat16

>  /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10/HIGG2D4_13TeV/CxAOD_31-10_a/qqZllHbbJ_PwPy8MINLO

If you check the branched of the CxAOD's here you want to see if the HLT MET branches have events in them. For example
>   EventInfo___NominalAuxDyn.passHLT_xe90_pufit_L1XE50

## Running the old Trigger Regime for Signal MC

Edit the file that steers the Reader
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
cd CxAODFramework_tag_r31-24/
vim /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE variables
>   >     NUMBEROFEVENTS="-1" ~L70
>   >     DRIVER="LSF" ~L73
>   >     JOBSIZELIMITMB="-1" ~L77
>   >     NRFILESPERJOB="50" ~L78 (or 20)
>   >     NOMINALONLY="true" ~L102
>   >     DO_ICHEP="true" ~L109

The main part of this study is to see whether we can get a significant increase of events in the signal region. The steps outlined above are about seeing the maximum possible impact the inclusion of these cuts can have. Hence the next check is to run over all of the MC signal and see whether the difference is useful. If you don't get an appreciable increase in events then there is no point doing this study.

>   COMMENT OUT DO_ICHEP if (~L426)
>   ADD in an override such that SAMPLES1FINAL has another input ~L703)
>   >     elif [[ ${SAMPLES1FINAL} == "2lsignal" ]]; then 
>   >     SAMPLES1="qqZllHbbJ_PwPy8MINLO qqWlvHbbJ_PwPy8MINLO ggZllHbb_PwPy8"

Since there is already a MET trigger in the 1L reader, then it makes sense that the way it is implimented in there should be attemped to be copied over to the 2L first. The 1 lepton reader pulls met, mu and electron information from an object called selectionResult. The objects in selectionResult will differ between the 1 and 2 lepton channel as you declare only 1 lepton (el and mu) but 2 for the 2 lepton obviously (el1, el2, mu1, mu2).

>   CxAODReader_VHbb/Root/AnalysisReader_VHQQ1Lep.cxx
>   CxAODTools_VHbb/Root/VHbb2lepEvtSelection.cxx
>   CxAODTools_VHbb/CxAODTools_VHbb/VHbb2lepEvtSelection.h

Rebuild the analysis
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/
cat source/CxAODBootstrap_VHbb/bootstrap/release.txt
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" && asetup 21.2.49,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
~~~
Some metadata for the analysis. It is performed on MC16a, using the tag 31-24. Now running the below commands should successfully run the cutflow over the grid.
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10 Signal_oldTrigger 2L a VHbb MVA D1 31-10 1
~~~
If this has successfully run. There should be histgrams in the Signal_oldTrigger/Reader_2L_MVA_31-10_a/fetch/ directory. The next time you want to then all that is needed to be done is for these to be hadd-ed. Then once the hadd-ed files are complete they can be opened in the TBrowser and the numbers in the Cutflow-> Nominal -> CutsMVANoWeight are the cutflow results.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/run/Signal_oldTrigger/Reader_2L_31-10_a_MVA_D1/fetch
setupATLAS && lsetup root
hadd SIGNAL.root hist-*
root -l SIGNAL.root
~~~
Now you can paste these commands in to get the bin contents and axis names
~~~
TBrowser fu (you need to navigate and select the CutsMVANoWeight)
CutsMVANoWeight->GetXaxis()->GetBinLabel(1)
CutsMVANoWeight->GetBinContent(1)
CutsMVANoWeight->GetXaxis()->GetBinLabel(2)
CutsMVANoWeight->GetBinContent(2)
CutsMVANoWeight->GetXaxis()->GetBinLabel(3)
CutsMVANoWeight->GetBinContent(3)
CutsMVANoWeight->GetXaxis()->GetBinLabel(4)
CutsMVANoWeight->GetBinContent(4)
CutsMVANoWeight->GetXaxis()->GetBinLabel(5)
CutsMVANoWeight->GetBinContent(5)
CutsMVANoWeight->GetXaxis()->GetBinLabel(6)
CutsMVANoWeight->GetBinContent(6)
CutsMVANoWeight->GetXaxis()->GetBinLabel(9)
CutsMVANoWeight->GetBinContent(9)
CutsMVANoWeight->GetXaxis()->GetBinLabel(10)
CutsMVANoWeight->GetBinContent(10)
CutsMVANoWeight->GetXaxis()->GetBinLabel(11)
CutsMVANoWeight->GetBinContent(11)
CutsMVANoWeight->GetXaxis()->GetBinLabel(12)
CutsMVANoWeight->GetBinContent(12)
CutsMVANoWeight->GetXaxis()->GetBinLabel(13)
CutsMVANoWeight->GetBinContent(13)
CutsMVANoWeight->GetXaxis()->GetBinLabel(14)
CutsMVANoWeight->GetBinContent(14)
CutsMVANoWeight->GetXaxis()->GetBinLabel(15)
CutsMVANoWeight->GetBinContent(15)
CutsMVANoWeight->GetXaxis()->GetBinLabel(16)
CutsMVANoWeight->GetBinContent(16)
CutsMVANoWeight->GetXaxis()->GetBinLabel(17)
CutsMVANoWeight->GetBinContent(17)
CutsMVANoWeight->GetXaxis()->GetBinLabel(18)
CutsMVANoWeight->GetBinContent(18)

~~~

## Running the new Trigger Regime for Signal MC

Now what we want to do is run the same commands but with a different trigger strategy.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/
setupATLAS
vim /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/source/CxAODReader_VHbb/Root/AnalysisReader_VHQQ2Lep.cxx
~~~
> ADD infomation about MET into pass2LepTrigger (~L289)
>   >     const xAOD::MissingET *met = selectionResult.met; (~L316)
>   >     m_triggerTool->setMET(met); (~L326)
>   >     bool triggerDec;
>   >     if (m_applyMETTriggerto2L) {
>   >     triggerDec = ((TriggerTool_VHbb2lep *)m_triggerTool)->getDecisionAndSFwithMET(triggerSF_nominal);  // Trial Trigger Regime
>   >     } else {
>   >     triggerDec = m_triggerTool->getDecisionAndScaleFactor(triggerSF_nominal);  // Standard Trigger Regime
>   >     }

COMMENT OUT
>   >     bool triggerDec = m_triggerTool->getDecisionAndScaleFactor(triggerSF_nominal); //Standard Trigger Regime (~L328)

Then you need to edit the trigger so it takes into account this new trigger. Currently there is a getDecisionAndScaleFactor present in the 0 and 2 lepton, but to add a MET trigger "smartly" we need a different regime "getDecisionAndSFwithMET". This needs to be edited and copied into TriggerTool.cxx

We want to add regions where we use certain cuts and all 1 lepton variables such as WVec need to be changes to ZVec.
~~~
vim source/CxAODTools_VHbb/Root/TriggerTool_VHbb1lep.cxx
vim source/CxAODTools_VHbb/Root/TriggerTool_VHbb2lep.cxx
~~~
>    ADD getDecisionAndSFwithMET(~L30)
>   >     bool TriggerTool_VHbb2lep::getDecisionAndSFwithMET(double& triggerSF) {
>   >
>   >     bool useMETAndMuonAthighptV=false;
>   >
>   >     const xAOD::MissingET* met = m_met;
>   >
>   >     // for electrons: ensure no MET trigger is allowed and return:
>   >     if (m_electrons.size()) {
>   >     setMET(nullptr);
>   >     bool decision = getDecisionAndScaleFactor(triggerSF);
>   >     setMET(met);
>   >     return decision;
>   >     }
>   >
>   >     // To decide on whether to use a single lepton trigger or MET trigger, construct pTZ
>   >     const xAOD::Muon* muon1 = *(m_muons.begin());
>   >     const xAOD::Muon* muon2 = *(m_muons.end());
>   >
>   >     TLorentzVector muVec1, muVec2;
>   >     muVec1.SetPtEtaPhiM(muon1->pt(), 0, muon1->phi(), 0);
>   >     muVec2.SetPtEtaPhiM(muon2->pt(), 0, muon2->phi(), 0);
>   >     TLorentzVector ZVecT = muVec1 + muVec2;
>   >
>   >     // for lower pT(Z) use muon trigger (don't allow MET):
>   >     if (ZVecT.Pt() < m_pTZcutVal) {
>   >     setMET(nullptr);
>   >     bool decision = getDecisionAndScaleFactor(triggerSF);
>   >     setMET(met);
>   >     return decision;
>   >     }
>   >     //If we get to this point we want to only use the MET trigger
>   >     xAOD::MissingETContainer* metCont = new xAOD::MissingETContainer();
>   >     xAOD::MissingETAuxContainer* metContAux = new xAOD::MissingETAuxContainer();
>   >     metCont->setStore( metContAux );

>   >     // otherwise use MET trigger, adding the muon to MET:
>   >     xAOD::MissingET* metWithMu = new xAOD::MissingET();
>   >     metCont -> push_back(metWithMu);
>   >     metWithMu -> setMpx(ZVecT.Px());
>   >     metWithMu -> setMpy(ZVecT.Py());
>   >     setMET(metWithMu);

>   >     if(useMETAndMuonAthighptV)   setMuons({muon1, muon2});
>   >     else  setMuons({});
>   >
>   >     //setMET(met);
>   >     bool decision = getDecisionAndScaleFactor(triggerSF);
>   >
>   >     setMET(met);
>   >     setMuons({muon1, muon2});
>   >
>   >     return decision;
>   >     }

>   ADD in Z pT to cut on ~L8
>   >       m_pTZcutVal = 150e3;
>   >       m_config.getif<double>("Trig::pTZcutVal", m_pTZcutVal);
>   ADD in a MET triggering ~L26
>   >       addLowestUnprescaledMET();
~~~
vim source/CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.h
~~~
>   ADD declaration of new trigger structure.
>    >     bool getDecisionAndSFwithMET(double& triggerSF);
>   ADD private data member for new pTZ cut (~L17)
>    >     private:
>    >     double m_pTZcutVal;

The last thing to do is to ensure that all the triggers you are interested are defined in the CommonProperties.cxx and .h files.
~~~
vim source/CxAODTools/CxAODTools/CommonProperties.h
vim source/CxAODTools/Root/CommonProperties.cxx
~~~

Then build and run again. Making sure to set up the right AnalysisBase.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/
cat source/CxAODBootstrap_VHbb/bootstrap/release.txt
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" && asetup 21.2.49,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10 Signal_newTrigger 2L a VHbb MVA D1 31-10 1

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/run/Signal_newTrigger/Reader_2L_31-10_a_MVA_D1/fetch
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
hadd SIGNAL.root hist-*
~~~

Once this is done the resulting trigger plots want to be extracted from the root files and prettied up.
~~~
vim /afs/cern.ch/work/d/dspiteri/VHbb/TriggerStudyPlots.cxx
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r31-24/","Signal","new","SIGNAL.root","2L","31-10","a","MVA", "D1","SR")'
imgcat run/Signal_TriggerPlots/CutflowTC_withPad.png
imgcat run/Signal_TriggerPlots/CutflowTJ_withPad.png
imgcat run/Signal_TriggerPlots/pTV_withPad.png
imgcat run/Signal_TriggerPlots/pTL1_withPad.png
imgcat run/Signal_TriggerPlots/pTL2_withPad.png
imgcat run/Signal_TriggerPlots/dEtaLL_withPad.png
imgcat run/Signal_TriggerPlots/dPhiLL_withPad.png
~~~

#  Next Stages: 32-07 Samples
Now the code has been able to run, the use of the 31-10 samples generated over a year ago for the ICEP conference in July of this year is probably not the best. There have been reports of things like lepton trigger requirements at the Maker for these samples.  So before any conclusions can be drawn from plots it would be good to update the samples which are run over. Luckily this should only include having to change the sample area to
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07/HIGG2D4_13TeV/CxAOD_32-07_a/

Then we also need to differentiate the sample output names as the names generated for the 31-07 will be the same.
~~~
mv Signal_oldTrigger/ Signal_31-10_oldTrigger/
mv Signal_newTrigger/ Signal_31-10_newTrigger/
mv Signal_TriggerPlots/ Signal_31-10_TriggerPlots/
mv Ttbar_oldTrigger/ Ttbar_31-10_oldTrigger/
mv Ttbar_newTrigger/ Ttbar_31-10_newTrigger/
mv Ttbar_TriggerPlots/ Ttbar_31-10_TriggerPlots/
~~~
Then since you are most likely using another version of the code. Make sure that
1) Edit SAMPLESCOMMON1 in submitReader.sh to only contain the SIGNAL sample.
 Do this by entering only the signals in the final command with the submitReader.sh
2) Ensure that the correct trigger decision function is created and called in AnalysisReader_VHQQ2Lep.cxx (L328)
3) Ensure that #include "xAODMissingET/MissingETContainer.h" #include "xAODMissingET/MissingETAuxContainer.h" are present in TriggerTool_VHbb2Lep.h (L6/7)
4) Ensure that   m_pTZcutVal and m_config.getif<double>("Trig::pTZcutVal", m_pTZcutVal);   is defined in TriggerTool_VHbb2lep in TriggerTool_VHbb2Lep.cxx and TriggerTool_VHbb2Lep.h (L8/9)
5) Ensure that the function getDecisionAndSFwithMET is defined in TriggerTool_VHbb2Lep.cxx and TriggerTool_VHbb2Lep.h (L33)
6) Ensure that the correct triggers are initialised/commented out in TriggerTool_VHbb2Lep.cxx (L26)

Feel free to run again.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalResolved_oldTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalResolved_newTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalResolved_newerplusTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalResolved_newerplusTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/run/Signal_oldTrigger/Reader_2L_32-07_a_MVA_D1/fetch
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
hadd SIGNAL.root hist-g* hist-q*
cd ../../..
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-24/run/SignalResolved_newTrigger/Reader_2L_32-07_a_MVA_D1/fetch
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
hadd SIGNAL.root hist-g* hist-q*
~~~
Or if you ran using the 'direct' driver
~~~
cd Signal_oldTrigger/Reader_2L_32-07_a_MVA_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../..
cd Signal_newTrigger/Reader_2L_32-07_a_MVA_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..
~~~
Then when you have generated your new samples, you can recreate the same plots with the same commands but pointing into a different folder.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalMuon","new","newer","SIGNAL.root","2L","32-07","a","MVA", "D1","SR")'

imgcat run/Signal_TriggerPlots/CutflowTC_withPad.png
imgcat run/Signal_TriggerPlots/CutflowTJ_withPad.png
imgcat run/Signal_TriggerPlots/pTV_withPad.png
imgcat run/Signal_TriggerPlots/pTL1_withPad.png
imgcat run/Signal_TriggerPlots/pTL2_withPad.png
imgcat run/Signal_TriggerPlots/MET_withPad.png
imgcat run/Signal_TriggerPlots/mLL_withPad.png
~~~
## Running the new Trigger Regime for the full production

With the excess seen in just the signal chanel, the question becomes whether it can be seen when the full production is ran over. First we shall run over all of MC16a

Now you have to carefully edit the submitReader.sh file to ensure that it only contains the samples you want
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
vim source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
>   ADD 2L Signal line (~L277)
>   >     SAMPLESCOMMON1+=" qqZllHbbJ_PwPy8MINLO qqWlvHbbJ_PwPy8MINLO ggZllHbb_PwPy8" # VH + extra jets, V->leptons, H->bb for 2L
>   COMMENT out all lines that are not contain samples not in MC16a for 2L
>   >     SAMPLESCOMMON1+=" qqZvvHbbJ_PwPy8MINLO qqZllHbbJ_PwPy8MINLO qqWlvHbbJ_PwPy8MINLO ggZvvHbb_PwPy8 ggZllHbb_PwPy8" # VH + extra jets, V->leptons, H->bb (L278)
>   >     SAMPLESCOMMON1+=" qqZvvHccJ_PwPy8MINLO qqZllHccJ_PwPy8MINLO qqWlvHccJ_PwPy8MINLO ggZvvHcc_PwPy8 ggZllHcc_PwPy8" # VH + extra jets, V->leptons, H->cc (L279)
>   >     # SAMPLESCOMMON1+=" WincHJZZ4l_PwPy8MINLO ZincHJZZ4l_PwPy8MINLO" # VH + extra jets, V->inclusive, H->ZZ->4leptons (L280)
> COMMENT out partial lines that contain some samples that are in MC16a for 2L (ALL BUT)
>   >     bbHinc_aMCatNLOPy8 (L281)
>   >     ttHinc_aMCatNLOPy8 (L282)

Then you run again over the old trigger using these similar commands 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 Full_a_oldTrigger_direct 2L a VHbb MVA D1 32-07 2lsignal none 1
~~~
Then after making the necessary adjestments/comments, you run over the new trigger
1) Ensure that the correct trigger decision function is called in AnalysisReader_VHQQ2Lep.cxx (L328)
2) Ensure that the correct triggers are initialised/commented out in TriggerTool_VHbb2Lep.cxx (L26)
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 Full_a_newTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1
~~~

# Running for Boosted

It was noticed that most of the increases seem to be in the tails of the plots and therefore it seems that a good thing to check was if the boosts were good for the Boosted Analysis. Not much has to be changed to see this. 

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
vim source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
>   CHANGE the analysis you are running (L235)
>   >     ANASTRATEGY = "Resolved" -> "Merged"

~~~
vim TriggerStudyPlots.cxx
~~~
>   COMMENT out the parts only true for the resolved Analysis (L102 & L104)
>   >     std::vector<TString> jetstagsofinterest = {}; 
>   >     std::vector<TString> ptvregions = {};

>   ADD filling of PtV and jets/tags depending on analysis strategy (L106)
>   >     if (modelType =="MVA"){ //Resolved Analysis
>   >     jetstagsofinterest.push_back("2tag2jet");
>   >     jetstagsofinterest.push_back("2tag3pjet");
>   >     //There is no 2tag3jet and 2tag4pjet in newer versions of the CxAOD's
>   >     ptvregions.push_back("0_75ptv");
>   >     ptvregions.push_back("75_150ptv");
>   >     ptvregions.push_back("150ptv");
>   >     } else if (modelType =="CUT"){ //Boosted Analysis
>   >     jetstagsofinterest.push_back("2tag1pfat0pjet");
>   >     ptvregions.push_back("0_250ptv");
>   >     ptvregions.push_back("250_400ptv");
>   >     ptvregions.push_back("400ptv");
>   >     } else {
>   >     std::cout<<"You need to specify an analysis regime"<<std::endl;
>   >     exit(1);
>   >     }

>   ADD plot retrieval dependence on analysis strategy (L140)
>   >     if (modelType == "MVA"){ 
>   >     plot2Get = samples.at(k)+"_"+jetstagsofinterest.at(j)+"_"+ptvregions.at(l)+"_"+region+"_"+variablesofinterest.at(i);
>   >     } else if (modelType == "CUT"){
>   >     plot2Get = samples.at(k)+"_"+jetstagsofinterest.at(j)+"_"+ptvregions.at(l)+"_"+region+"_noaddbjetsr_"+variablesofinterest.at(i);
>   >     }

Once this is done, you can re-run the release and 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_oldTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_newTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1
~~~
Then hadd the outputs and generate the plots!
~~~
cd SignalBoosted_oldTrigger/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../..
cd SignalBoosted_newTrigger/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-15/","SignalBoosted","old","new","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'

imgcat run/SignalBoosted_TriggerPlots/CutflowTC_withPad.png
imgcat run/SignalBoosted_TriggerPlots/CutflowTJ_withPad.png
imgcat run/SignalBoosted_TriggerPlots/pTV_withPad.png
imgcat run/SignalBoosted_TriggerPlots/pTL1_withPad.png
imgcat run/SignalBoosted_TriggerPlots/pTL2_withPad.png
imgcat run/SignalBoosted_TriggerPlots/MET_withPad.png
imgcat run/SignalBoosted_TriggerPlots/mLL_withPad.png
~~~

# Running a different trigger regime ( in Boosted)

Currently we have been defining the MET to cut on using the muon information in the event a "muon psuedo-MET" as it were. One thing to look at would be to see if the same kind of gains can be made using the real MET in the event. Since we already have an "old" trigger setup from the previous exercise, all that should need happen is the edit of the trigger regime and re-running of the framework.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
vim source/CxAODTools_VHbb/Root/TriggerTool_VHb2Lep.cxx
~~~
>  COMMENT IN MET trigger regime (L27)
>   >     addLowestUnprescaledMET();
>  COMMENT OUT old way setting the MET (L74)
>   >     setMET(metWithMu);
>   ADD new way of setting the MET is set (L75)
>   >     setMET(met);

 Also ensure that the correct trigger decision function is called in AnalysisReader_VHQQ2Lep.cxx (L308)

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_newerTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1

cd SignalBoosted_newerTrigger/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalBoosted","old","newer","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'
~~~

The way the trigger works is that the MET object to trigger on that I construct is made up of the information from the muons in the event. To have a useful OR of the triggers, when this constructed MET is used, the muons in that event are removed. However in the case where the normal MET is used perhaps removal of the muons leads to an overall decrease in efficiency. The re-running of both of the above trigger regimes gives us an opportunity to see what kind of things having an 'AND' of the MET and MUON triggers mean. This is easy to edit. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-15/
vim source/CxAODTools_VHbb/Root/TriggerTool_VHb2Lep.cxx
~~~
>  CHANGE Muon switch to TRUE (L33)
>   >     bool useMETAndMuonAthighptV=true;

In addition to this you need to ensure that the correct one of setMET(metWithMu) [for new or newplus] or setMET(met) [for newer and newerplus] is commented in/out. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_newerplusTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1

cd SignalBoosted_newerplusTrigger/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalBoosted","old","newerplus","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'


AND 

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_newplusTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1

cd SignalBoosted_newplusTrigger/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalBoosted","old","newplus","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'


imgcat run/SignalBoosted-oldandnew_TriggerPlots/*.png
imgcat run/SignalBoosted-oldandnewer_TriggerPlots/*.png
imgcat run/SignalBoosted-oldandnewplus_TriggerPlots/*.png
imgcat run/SignalBoosted-oldandnewerplus_TriggerPlots/*.png
imgcat run/SignalBoosted-newandnewer_TriggerPlots/*.png
imgcat run/SignalBoosted-newplusandnewerplus_TriggerPlots/*.png

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_newestplusTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1

~~~

## To re-run the Resolved Analysis

Make sure that you change ANASTRATEGY from Merged -> Resolved in submitreader.sh and you change it back when you run in the Boosted regime again. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalResolved_newerTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalResolved_newerplusTrigger 2L a VHbb MVA D1 32-07 2lsignal none 1


cd SignalResolved_newerplusTrigger/Reader_2L_32-07_a_MVA_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalResolved","old","newerplus","SIGNAL.root","2L","32-07","a","MVA", "D1","SR")'


cd SignalResolved_newerTrigger/Reader_2L_32-07_a_MVA_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalResolved","old","newer","SIGNAL.root","2L","32-07","a","MVA", "D1","SR")'

mv run/*_TriggerPlots run/ResolvedPlots/
imgcat run/ResolvedPlots/SignalResolved-oldandnewer_TriggerPlots/*.png
imgcat run/ResolvedPlots/SignalResolved-oldandnewerplus_TriggerPlots/*.png
~~~

## Adding another MET proxy.

Since the inclusive MET trigger that is used only makes use of calorimeter information, there is no muon correction to the reco-MET. One possible idea of a proxy would be to add an additional MET proxy in the form of vector added event reco-MET and the dimuon pt. To test this I will only do one run. This run will have the  bool useMETAndMuonAthighptV=true (TriggerTool_VHbb2Lep.cxx) to include the muon information in the trigger decision
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
vim source/CxAODTools_VHbb/Root/TriggerTool_VHbb2lep.cxx
~~~
>   COMMENT OUT  metWithMu declaration and filling (~L69-73)
>   ADD IN filling in of  new object called recoMETWithMu (~L75)
>   >     //Comment in this to fill a MET object with vector-added muon pt and reco met to trigger on :
>   >     xAOD::MissingET* recoMETWithMu = new xAOD::MissingET();
>   >     metCont -> push_back(recoMETWithMu);
>   >     double METx = ZVecT.Px() + met->mpx();
>   >     double METy = ZVecT.Py() + met->mpy();
>   >     recoMETWithMu -> setMpx(METx);
>   >     recoMETWithMu -> setMpy(METy);

>   ADD IN setMET to recoMETWithMu (~83)
>   >     setMET(recoMETWithMu);
>   COMMENT OUT  setMET with metWithMu or met (~L84-85)

Then just rebuild and run as normal
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_newestplusTrigger 2L a VHbb CUT D1 32-07 2lsignal none 1

cd SignalBoosted_newestplusTrigger/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalBoosted","old","newestplus","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'

mv run/*_TriggerPlots run/BoostedPlots/
imgcat run/BoostedPlots/SignalBoosted-oldandnewestplus_TriggerPlots/*.png
~~~

# Adding a switch to the analysis for the study

Once the main code has been agreed upon, the next stage is to add some flags to the configuration files such that anyone who wants to perform this analysis can do so easily by just turning on a simple switch. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/source/
setupATLAS && lsetup git
~~~
The first thing to do is to define and add in the switch.
~~~
vim CxAODReader_VHbb/data/framework-read-automatic.cfg
~~~
>   ADD 2 lepton specific switch  (~L98)
>   >     # 2 LEPTON SPECIFIC SWITCHES
>   >     # -----------------------------
>   >     bool applyMETTriggerto2L            = METIN2L
>   >     string METMuonTriggerCombin2L       = TRIGGERCOMBIN2L
>   ADD printing out of configuration (~L424)
>   >     echo "METIN2L=${METIN2L}"
>   >     echo "TRIGGERCOMBIN2L=${TRIGGERCOMBIN2L}"
>   ADD switch to Variable list  (~L809)
~~~
vim CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> ADD definition and filling of the switch (L266)
>   >     METIN2L="false"
>   >     # Introduces the MET trigger to the 2L analysis at a given PtZ value. Default is 150GeV
>   >     if [[ ${METIN2L} == "true" ]]; then 
>   >     # Choose the combination of MET and Muon Trigger in 2L 
>   >     TRIGGERCOMBIN2L="METOnly" # "METOnly", "METwithMuon"
>   >     else
>   >     TRIGGERCOMBIN2L="null"
>   >     fi 
>   Add printout of new configuration values (~L426)
>   >     echo "METIN2L=${METIN2L}"
>   >     echo "TRIGGERCOMBIN2L=${TRIGGERCOMBIN2L}"
>   Add new variables to VARIABLES string (~L811)
>   >     VARIABLEs="... METIN2L TRIGGERCOMBIN2L ..."

Then create a member variable that stores the switch
~~~
vim CxAODReader_VHbb/Root/AnalysisReader_VHQQ.cxx
~~~
> ADD member variables to  AnalysisReader_VHQQ::AnalysisReader_VHQQ() (~L30)
>   >     m_applyMETTriggerto2L(false),
>   >     m_METMuonTriggerCombin2L("METOnly"),
> ADD reading of switch from member variables in initializeSelection() method (~L205)
>   >     m_config->getif<bool>("applyMETTriggerto2L", m_applyMETTriggerto2L);
>   >     m_config->getif<string>("METMuonTriggerCombin2L", m_METMuonTriggerCombin2L);  

Then add these new variables to the header file.
~~~
vim CxAODReader_VHbb/CxAODReader_VHbb/AnalysisReader_VHQQ.h
~~~
> ADD declaration of member variables to the header file (~L475)
>   >     bool m_applyMETTriggerto2L = false;        //!
>   >     std::string m_METMuonTriggerCombin2L;      //!

Since the thing you want to add affects the Tools package and the Tools package is a dependency of the Reader, the variables have to be added here as well. You can edit the base TriggerTool.cxx package as the TriggerTool_VHbb2Lep.h points to TriggerTool_VHbb.h which in turn points to TriggerTool.h.
~~~
vim CxAODTools/Root/TriggerTool.cxx
~~~
> ADD member variables to  TriggerTool::TriggerTool(ConfigStore& config) (~L11)
>   >     m_applyMETTriggerto2L(false),
>   >     m_METMuonTriggerCombin2L("METOnly"),
> ADD reading of switch from member variables  (~L31)
>   >     m_config.getif<bool>("applyMETTriggerto2L", m_applyMETTriggerto2L);
>   >     m_config.getif<std::string>("METMuonTriggerCombin2L", m_METMuonTriggerCombin2L); 

Then add these new variables to the header file.
~~~
vim CxAODTools/CxAODTools/TriggerTool.h
~~~
> ADD declaration of member variables to the header file (~L125)
>   >     //Needed to add MET Trigger to 2L Analysis
>   >     bool m_applyMETTriggerto2L = false;
>   >     std::string m_METMuonTriggerCombin2L; 

Then you have to add what you want the switch to do to the analysis.
~~~
vim CxAODReader_VHbb/Root/AnalysisReader_VHbb2Lep.cxx
~~~
> CHANGE trigger initialisation to if statement (L308)
>   >     bool triggerDec;
>   >     if (m_applyMETTriggerto2L) {
>   >     triggerDec = ((TriggerTool_VHbb2lep *)m_triggerTool)->getDecisionAndSFwithMET(triggerSF_nominal);  // Trial Trigger Regime
>   >     } else {
>   >     triggerDec = m_triggerTool->getDecisionAndScaleFactor(triggerSF_nominal); //Standard Trigger Regime
>   >     }
~~~
vim CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.cxx
~~~
>   ADD Reader header file to be able to use the newly-defined switches (L1)
>   >     #include "CxAODReader_VHbb/AnalysisReader_VHQQ.h" 

>   CHANGE Trigger initialisation (L28)
>   >     addLowestUnprescaledMET(); -> if (m_applyMETTriggerto2L)   addLowestUnprescaledMET();
>   ADD control of MET and Muon boolean
>   >     bool useMETAndMuonAthighptV;
>   >     if (m_METMuonTriggerCombin2L == "METOnly") {
>   >     useMETAndMuonAthighptV = false;
>   >     } else if (m_METMuonTriggerCombin2L == "METwithMuon") {  
>   >     useMETAndMuonAthighptV = true;
>   >     }

Then we can try running the analysis with these switches. 
Make sure that you toggle the settings in submitReader.sh
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r32-07/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_TEST 2L a VHbb CUT D1 32-07 2lsignal none 1
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-07 SignalBoosted_NULL 2L a VHbb CUT D1 32-07 2lsignal none 1

cd SignalBoosted_TEST/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../..
cd SignalBoosted_TEST2/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../..
cd SignalBoosted_NULL/Reader_2L_32-07_a_CUT_D1
mkdir fetch && mv hist-* fetch && cd fetch
hadd SIGNAL.root hist-* && cd ../../../..
~~~

Since this will be the final form of the analysis you will have to move some things around
~~~
mkdir Samples
mv SignalResolved* Samples/
mv SignalBoosted_n* Samples/
mv SignalBoosted_o* Samples/
mv SignalBoosted_NULL/ SignalBoosted_oldTrigger
mv SignalBoosted_TEST2/ SignalBoosted_newestplusTrigger
mv SignalBoosted_TEST/ SignalBoosted_newestTrigger
mv ResolvedPlots Samples/
mv BoostedPlots Samples/
mv Samples/ SamplesAndOldPlots/
cd ..
vim ../TriggerStudyPlots.cxx
~~~
>  Add new type of legend entry to the Trigger 
>   >     if (triggertype1 == "newest"){legend1 = "No Muon, MET + 'MET' > 150 PtV";} (~L180)
>   >     if (triggertype2 == "newest"){legend2 = "No Muon, MET + 'MET' > 150 PtV";} (~L187)

~~~
cd ..

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalBoosted","old","newestplus","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_tag_r32-07/","SignalBoosted","old","newest","SIGNAL.root","2L","32-07","a","CUT", "D1","SR")'

~~~
# Running on the master branch with 32-15 samples

These will serve as the final plots for this study. Copy the changes made on the development branch to a freshly checked out and built copy of the master. (easier said than done). Then run the following. We will not be running the resolved. 

## 32-15 2L signal samples
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j6
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run

~~~
For the Boosted Analysis:
Ensuring first that Ensuring first that ANASTRATEGY="Merged"
METIN2L="false" in submitReader.sh
DO2LMETTRIGGER="false" || DO2LMETANDMUONTRIGGER="false" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_oldTrigger 2L a,d,e VHbb CUT D1 32-15 2lsignal none 1
~~~
Then set  METIN2L="true"  and TRIGGERCOMBIN2L="METOnly" in submitReader.sh
DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="false" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_newestTrigger 2L a,d,e VHbb CUT D1 32-15 2lsignal none 1
~~~
Then set  METIN2L="true"  and TRIGGERCOMBIN2L="METwithMuon" in submitReader.sh
DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="true" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_newestplusTrigger 2L a,d,e VHbb CUT D1 32-15 2lsignal none 1
~~~
After this the output needs to be formatted into a format to be fed into my plots file. 
~~~

cd SignalBoosted_oldTrigger/Reader_2L_32-15_a_CUT_D1/fetch
hadd SIGNAL_a.root hist-* && mv SIGNAL_a.root ../.. && cd ../..
cd Reader_2L_32-15_d_CUT_D1/fetch
hadd SIGNAL_d.root hist-* && mv SIGNAL_d.root ../.. && cd ../..
cd Reader_2L_32-15_e_CUT_D1/fetch
hadd SIGNAL_e.root hist-* && mv SIGNAL_e.root ../.. && cd ../..
mkdir Reader_2L_32-15_ade_CUT_D1 && cd Reader_2L_32-15_ade_CUT_D1
mkdir fetch && cd ..
hadd SIGNAL.root SIGNAL_* && mv SIGNAL* Reader_2L_32-15_ade_CUT_D1/fetch
cd ..

cd SignalBoosted_newestTrigger/Reader_2L_32-15_a_CUT_D1/fetch
hadd SIGNAL_a.root hist-* && mv SIGNAL_a.root ../.. && cd ../..
cd Reader_2L_32-15_d_CUT_D1/fetch
hadd SIGNAL_d.root hist-* && mv SIGNAL_d.root ../.. && cd ../..
cd Reader_2L_32-15_e_CUT_D1/fetch
hadd SIGNAL_e.root hist-* && mv SIGNAL_e.root ../.. && cd ../..
mkdir Reader_2L_32-15_ade_CUT_D1 && cd Reader_2L_32-15_ade_CUT_D1
mkdir fetch && cd ..
hadd SIGNAL.root SIGNAL_* && mv SIGNAL* Reader_2L_32-15_ade_CUT_D1/fetch
cd ..

cd SignalBoosted_newestplusTrigger/Reader_2L_32-15_a_CUT_D1/fetch
hadd SIGNAL_a.root hist-* && mv SIGNAL_a.root ../.. && cd ../..
cd Reader_2L_32-15_d_CUT_D1/fetch
hadd SIGNAL_d.root hist-* && mv SIGNAL_d.root ../.. && cd ../..
cd Reader_2L_32-15_e_CUT_D1/fetch
hadd SIGNAL_e.root hist-* && mv SIGNAL_e.root ../.. && cd ../..
mkdir Reader_2L_32-15_ade_CUT_D1 && cd Reader_2L_32-15_ade_CUT_D1
mkdir fetch && cd ..
hadd SIGNAL.root SIGNAL_* && mv SIGNAL* Reader_2L_32-15_ade_CUT_D1/fetch
cd ../..


root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","SignalBoosted","old","newest","SIGNAL.root","2L","32-15","ade","CUT", "D1","SR")'

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","SignalBoosted","old","newestplus","SIGNAL.root","2L","32-15","ade","CUT", "D1","SR")'
~~~

# Merging Updates
- Replace METIN2L with DO2LMETTRIGGER and 
TRIGGERCOMBIN2L with DO2LMETANDMUONTRIGGER
- Make both booleans.

- Edited files
submitReader.sh
framework-read-automatic.cfg
framework-read.cfg
AnalysisReader_VHQQ.cxx
AnalysisReader_VHQQ.h

- Move declaration of VHbb specific things to TriggerTool_VHbb or TriggerTool_VHbb,  and turn both of the added

- Edited files
TriggerTool.cxx
TriggerTool.h
TriggerTool_VHbb.cxx
TriggerTool_VHbb.h
TriggerTool_VHbb2Lep.cxx


# Boosted 32-15 2L background samples

Now it has been shown that there is an increase in the 2L Signal, we need to run over the background.
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_a
There are a lot of samples here so you want to run over a test first to see if everything is A-OK. Z+jets is the main background to 2L boosted (https://indico.cern.ch/event/780736/contributions/3304112/attachments/1789359/2914430/2019_02_01_WSboostedVHbb_.pdf) and the MET trigger changes the muon contribution so a good test sample to run over is ZmumuB. There are two ZmumuB samples, a Sherpa one and a MadGraph-Pythia one. The nominal sample is the Sherpa one, and the MadGraph is used for 2-point systematics. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build 
make -j6
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run
~~~
DO2LMETTRIGGER="false" || DO2LMETANDMUONTRIGGER="false" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 ZmumuBBoosted_oldTrigger 2L a VHbb CUT D1 32-15 ZmumuB_Sh221 none 1
~~~
DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="false" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 ZmumuBBoosted_newestTrigger 2L a VHbb CUT D1 32-15 ZmumuB_Sh221 none 1
~~~
DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="true" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 ZmumuBBoosted_newestplusTrigger 2L a VHbb CUT D1 32-15 ZmumuB_Sh221 none 1


cd ZmumuBBoosted_oldTrigger/Reader_2L_32-15_a_CUT_D1/fetch
hadd ZMUMUB.root hist-* && cd ../../..
cd ZmumuBBoosted_newestTrigger/Reader_2L_32-15_a_CUT_D1/fetch
hadd ZMUMUB.root hist-* && cd ../../..
cd ZmumuBBoosted_newestplusTrigger/Reader_2L_32-15_a_CUT_D1/fetch
hadd ZMUMUB.root hist-* && cd ../../../..

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","ZmumuBBoosted","old","newest","ZMUMUB.root","2L","32-15","a","CUT", "D1","SR")'

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","ZmumuBBoosted","old","newestplus","ZMUMUB.root","2L","32-15","a","CUT", "D1","SR")'
~~~
## 32-15 2L full running.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build 
make -j6
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run
~~~
DO2LMETTRIGGER="false" || DO2LMETANDMUONTRIGGER="false" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_oldTrigger 2L a,d,e VHbb CUT D1 32-15 none none 1
~~~
DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="false" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger 2L a,d,e VHbb CUT D1 32-15 none none 1
~~~
DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="true" 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestplusTrigger 2L a,d,e VHbb CUT D1 32-15 none none 1
~~~

Then same as the last times, the resulting files need to be hadd-ed 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/run/FullBoosted_oldTrigger/Reader_2L_32-15_a_CUT_D1/fetch

rm hist-data*
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd STOPs.root hist-stop*
hadd TTBAR_DILEP.root hist-ttbar_dilep*
hadd TTBAR_1PLUSLEP.root hist-ttbar_nonallhad*
hadd WENUB.root hist-WenuB*
hadd WENU.root hist-Wenu_Sh221*
hadd WMUNUB.root hist-WmunuB*
hadd WMUNU.root hist-Wmunu_Sh221*
hadd WTAUNUB.root hist-WtaunuB*
hadd WTAUNU.root hist-Wtaunu_Sh221*
hadd WZ.root  hist-WlvZ* hist-WqqZ*
hadd ZZ.root  hist-ZbbZ* hist-ZqqZ*
hadd ZEEB.root  hist-ZeeB*
hadd ZEE.root  hist-Zee_Sh221*
hadd ZMUMUB.root  hist-ZmumuB*
hadd ZMUMU.root  hist-Zmumu_Sh221-*
hadd ZTAUMTAUB.root  hist-ZtautauB*
hadd ZTAUMTAU.root  hist-Ztautau_Sh221*

hadd 2LEPALL.root GGZH.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

hadd 2LBKG.root GGZZ.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

rm GGZH.root GGZZ.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/run/FullBoosted_newestTrigger/Reader_2L_32-15_a_CUT_D1/fetch

rm hist-data*
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd STOPs.root hist-stop*
hadd TTBAR_DILEP.root hist-ttbar_dilep*
hadd TTBAR_1PLUSLEP.root hist-ttbar_nonallhad*
hadd WENUB.root hist-WenuB*
hadd WENU.root hist-Wenu_Sh221*
hadd WMUNUB.root hist-WmunuB*
hadd WMUNU.root hist-Wmunu_Sh221*
hadd WTAUNUB.root hist-WtaunuB*
hadd WTAUNU.root hist-Wtaunu_Sh221*
hadd WZ.root  hist-WlvZ* hist-WqqZ*
hadd ZZ.root  hist-ZbbZ* hist-ZqqZ*
hadd ZEEB.root  hist-ZeeB*
hadd ZEE.root  hist-Zee_Sh221*
hadd ZMUMUB.root  hist-ZmumuB*
hadd ZMUMU.root  hist-Zmumu_Sh221-*
hadd ZTAUMTAUB.root  hist-ZtautauB*
hadd ZTAUMTAU.root  hist-Ztautau_Sh221*

hadd 2LEPALL.root GGZH.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

hadd 2LBKG.root GGZZ.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

rm GGZH.root GGZZ.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/run/FullBoosted_newestplusTrigger/Reader_2L_32-15_a_CUT_D1/fetch

rm hist-data*
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd STOPs.root hist-stop*
hadd TTBAR_DILEP.root hist-ttbar_dilep*
hadd TTBAR_1PLUSLEP.root hist-ttbar_nonallhad*
hadd WENUB.root hist-WenuB*
hadd WENU.root hist-Wenu_Sh221*
hadd WMUNUB.root hist-WmunuB*
hadd WMUNU.root hist-Wmunu_Sh221*
hadd WTAUNUB.root hist-WtaunuB*
hadd WTAUNU.root hist-Wtaunu_Sh221*
hadd WZ.root  hist-WlvZ* hist-WqqZ*
hadd ZZ.root  hist-ZbbZ* hist-ZqqZ*
hadd ZEEB.root  hist-ZeeB*
hadd ZEE.root  hist-Zee_Sh221*
hadd ZMUMUB.root  hist-ZmumuB*
hadd ZMUMU.root  hist-Zmumu_Sh221-*
hadd ZTAUTAUB.root  hist-ZtautauB*
hadd ZTAUTAU.root  hist-Ztautau_Sh221*

hadd SIGNAL.root GGZH.root QQWH.root QQZH.root 

hadd 2LEPALL.root GGZH.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

hadd 2LBKG.root GGZZ.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 

rm GGZH.root GGZZ.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUMTAUB.root ZTAUMTAU.root 
~~~
Also rather than copy-pasting this all these times you can alternatively source VHbbHaddAlladeMVA.sh which has all of these hadd-ing commands.

Then you just need to produce the final plots! However since the decision was made to not have both of the triggered firing at the same time, the newestplus convention is now obselete.

Ensure that TriggerPlots.cxx changes each time you run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master
~~~
  std::vector<TString> samples   = {"ggZZ","stops", "stopt", "stopWt", "ttbar", "Wbb", "Wbc", "Wbl", "Wcc", "Wcl", "Wl", "WZ", "Zbb","Zbc","Zbl","Zcc", "Zcl", "Zl", "ZZ"}; //All Backgrounds
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","FullBoosted","old","newest","2LBKG.root","2L","32-15","a","CUT", "D1","SR")'
cd run
mv FullBoosted-oldandnewest_TriggerPlots BkgBoosted-oldandnewest_TriggerPlots
mv BkgBoosted-oldandnewest_TriggerPlots 32-15Plots
cd ..
~~~
  std::vector<TString> samples   = {"ggZllH125", "qqZllH125", "qqWlvH125"}; //Signal
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","SignalBoosted","old","newest","SIGNAL.root","2L","32-15","a","CUT", "D1","SR")'
mv run/SignalBoosted-oldandnewest_TriggerPlots run/32-15Plots
~~~

All of this will only get plots for MC16a. You have to repeat the steps for MC16d and MC16e

# Resolved 32-15 2L samples
You see now the with the splitting of this into regions and the fact that the resolved analysis wants to another region for ptv (the extreme region). We have to go back to the resolved and rerun with the region splitting. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
vim source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
>   CHANGE the analysis you are running (L235)
>   >     ANASTRATEGY = "Merged" -> "Resolved"


DO2LMETTRIGGER="false" || DO2LMETANDMUONTRIGGER="false" (L265)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/build 
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullResolved_oldTrigger 2L a,d,e VHbb MVA D1 32-15 none none 1
~~~


DO2LMETTRIGGER="true" || DO2LMETANDMUONTRIGGER="false" (L265)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/build 
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullResolved_newestTrigger 2L a,d,e VHbb MVA D1 32-15 none none 1


cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"

~~~
Checking that all the inputs were fine.
~~~
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_MVA_D1
~~~
If there are problematic root files follow the instructions under the "Troubleshooting" heading in the file VHbb_2Lep_Reader_Inputs.md
~~~
cd run/FullResolved_oldTrigger/
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAlladeMVA.sh
cd ../FullResolved_newestTrigger/
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAlladeMVA.sh

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master

~~~
std::vector<TString> samples   = {"ggZllH125", "qqZllH125", "qqWlvH125"}; //Signal
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","SignalBoosted","old","newest","SIGNAL.root","2L","32-15","a","CUT", "D1","SR")'
cd run
mv SignalBoosted-oldandnewest_TriggerPlots SignalBoosted-oldandnewest_a_TriggerPlots
cd SignalBoosted-oldandnewest_a_TriggerPlots
imgcat *.pdf
~~~
std::vector<TString> samples   = {"ggZZ","stops", "stopt", "stopWt", "ttbar", "Wbb", "Wbc", "Wbl", "Wcc", "Wcl", "Wl", "WZ", "Zbb","Zbc","Zbl","Zcc", "Zcl", "Zl", "ZZ"}; //All Backgrounds
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","FullBoosted","old","newest","2LBKG.root","2L","32-15","a","CUT", "D1","SR")'
cd run
mv FullBoosted-oldandnewest_TriggerPlots BkgBoosted-oldandnewest_a_TriggerPlots
cd BkgBoosted-oldandnewest_a_TriggerPlots
imgcat *.pdf
~~~


