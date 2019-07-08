# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about the testing new trigger regimes for the 2 Lepton Analysis for VHbb. #

## VHbb 2 Lepton Trigger Study Part 2 ##

Last Edited: 02-07-2019
-------------------------------------------------------------------------------

# Setup Script
Search https://gitlab.cern.ch/CxAODFramework/FrameworkSub to see what the latest released version is.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
source getMaster.sh origin/master CxAODFramework_master 1 1
> source getMaster.sh r31-10 CxAODFramework_tag_r31-10 1 1 [for a tag]
~~~

# Editing MET Trigger code to also work with 1L
There are 9 steps
1) Update Variables in submitReader
2) Proliferate new Variables to config files
3) Move TriggerTool_VHbb2Lep.cxx code to TriggerTool_VHbb.cxx
4) Add necessary config variables from TriggerTool_VHbb1Lep.cxx and TriggerTool_VHbb2Lep.cxx to TriggerTool_VHbb.cxx
5) Change 2L MET trigger code to work with 1L instances of the function as well.
6) Declare MET Trigger code and config variables in TriggerTool.h
7) Remove MET Trigger code, variables and declalarations from TriggerTool_VHbb1lep.cxx, TriggerTool_VHbb1lep.h, TriggerTool_VHbb2lep.cxx and TriggerTool_VHbb2lep.h,   
8) Change function calls and add includes in AnalysisReader_VHQQ2Lep.cxx and 1L equivalent
9) Change names of variables in AnalysisReader_VHQQ.cxx and AnalysisReader_VHQQ.h
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source
~~~
I won't go into as much detail as I have before for the sake of brevity, but I will go over each of these points briefly.
## 1) Update Variables in submitReader
~~~
vim CxAODOperation_VHbb/scripts/submitReader.sh
~~~
> CHANGE variable names (~L266, L407, L1076)
> >   METANDMUONTRIGGERCOMBin2L -> DOMETANDMUONTRIGGER

## 2) Proliferate new Variables to config files
~~~
vim CxAODOperation_VHbb/CxAODReader_VHbb/data/framework-read-automatic.cfg
~~~
> MOVE MET and muon trigger flag to general part of code (~L60 from ~L104)
> >   bool doMETMuonTrigger               = DOMETANDMUONTRIGGER # if true, it merges allows running of the MET and Muon triggers simultaneously in a logical 'or'
> CHANGE name of 2L MET trigger application flag in # 2 LEPTON SPECIFIC SWITCHES part of the code (~L103)
> >   applyMETTriggerto2L -> METTriggerin2L

Repeat the same steps with the next configuration file, but instead initialising  all the variables from the submitReader. Initialise them to 'false'
~~~
vim CxAODOperation_VHbb/CxAODReader_VHbb/data/framework-read.cfg
~~~
bool doMETMuonTrigger               = false (~L57)
bool METTriggerin2L                 = false (~L90)

## 3) Move TriggerTool_VHbb2Lep.cxx code to TriggerTool_VHbb.cxx
This is a simple copy-paste job, but while you 
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb.cxx
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.cxx
~~~
> CHANGE all instances of config member variables to new names (~L6,L7,L9,L10,L27...)
> >    m_applyMETTriggerto2L -> m_METTriggerin2L
> >    m_METandMuonTriggerCombin2L -> m_doMETMuonTrigger
> CHANGE conditional initialisation of the trigger
> >    if (m_METTriggerin2L) -> if (m_METTriggerin2L || m_doMETMuonTrigger)
> CUT all of ""bool TriggerTool_VHbb::getDecisionAndSFwithMET(double& triggerSF)"" from TriggerTool_VHbb2Lep.cxx to TriggerTool_VHbb.cxx

## 4) Add necessary config variables from TriggerTool_VHbb1Lep.cxx and TriggerTool_VHbb2Lep.cxx to TriggerTool_VHbb.cxx
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb.cxx
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb1Lep.cxx
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.cxx
~~~
> CUT all of lines of config declarations relating to m_doMETMuonTrigger, m_METTriggerin2L, m_do_1L_MuonTrigger and m_pT(W/Z)cutVal from TriggerTool_VHbb1lep::TriggerTool_VHbb1lep and TriggerTool_VHbb2lep::TriggerTool_VHbb2lep to TriggerTool_VHbb::TriggerTool_VHbb
## 5) Change 2L MET trigger code to work with 1L instances of the function as well.
Here analysis-dependent decisions need to be made as ptV is made up of different objects depending on the analysis. Declare all objects but only fill them depending on analysis type.
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb.cxx
~~~
> ADD initialasion of m_analysisType
> >   m_analysisType("2lep") (~L13) 

> ADD boolean to store the decision to not use the MET trigger
> >   bool notUsingMETTrigger; (~L45)

> CHANGE declaration of variables and calculation of ptV to something like (~L47) 
>	>	    const xAOD::Muon* muon1 = m_muons.at(0);
>	>	    const xAOD::Muon* muon2; //Declaration of muon here just in case 2L analysis calls this. Will just exist for the 1L one and not do anything. 
>	>	    TLorentzVector pTVVec, muVec1;
>	>	    muVec1.SetPtEtaPhiM(muon1->pt(), 0, muon1->phi(), 0);
>	>	    
>	>	    if (m_analysisType == "1lep"){ //Constructs PtV from met and the muon
>	>	        TLorentzVector metVec;
>	>	    	  metVec.SetPtEtaPhiM(m_met->met(), 0, m_met->phi(), 0); //There will be actual met in the event
>	>	    	  pTVVec = muVec1 + metVec; // WVecT
>	>	    	  if (pTVVec.Pt() < m_pTWcutVal || m_do_1L_MuonTrigger) {notUsingMETTrigger = true}
>	>	  
>	>	    } else if (m_analysisType == "2lep"){ //Constructs PtV from the two muons
>	>	    	  TLorentzVector muVec2;
>	>	    	  muon2 = m_muons.at(1);
>	>	    	  muVec2.SetPtEtaPhiM(muon2->pt(), 0, muon2->phi(), 0);
>	>	    	  pTVVec = muVec1 + muVec2; //ZVecT
>	>	    	  if (pTVVec.Pt() < m_pTZcutVal) {notUsingMETTrigger = true}
>	>	  
>	>	    } else { //Add instance to catch accidental calls from 0L analysis
>	>	    	Error("TriggerTool_VHbb::getDecisionAndSFwithMET()",
>	>	              "Currenlty this method cannot work with the 0L analysis");
>	>	      exit(1);
>	>	    } 
> ADD analysis-chanel dependence of MET proxy filling (~L92)
> >   double METx , METy;
>	>	  if (m_analysisType == "1lep"){
>	>	  	  METx = pTVVec.Px();
>	>	  	  METy = pTVVec.Py();
>	>   } else if (m_analysisType == "2lep"){
>	>       METx = pTVVec.Px() + met->mpx();
>	>	   	  METy = pTVVec.Py() + met->mpy();
>	>	  }
> CHANGE selective setting of event variables(~L104-L112)
> >   if(useMETAndMuonAtHighPtV)   setMuons({muon1, muon2}); ->   if(useMETAndMuonAtHighPtV && m_analysisType == "1lep")   setMuons({muon1});
> >   else if(useMETAndMuonAtHighPtV && m_analysisType == "2lep")   setMuons({muon1, muon2});

## 6) Declare MET Trigger code and config variables in TriggerTool.h
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb.h
~~~
> ADD function declaration and config variables to public: part of class
> >   public: (~L8)
> >    [...]
> >     bool getDecisionAndSFwithMET(double& triggerSF);
> >  
> >     //Needed to add MET Trigger to 1L and 2L Analysis
> >     bool m_do_1L_MuonTrigger    = false;
> >     bool m_METTriggerin2L       = false;
> >     bool m_doMETMuonTrigger     = false; 
> >     std::string m_analysisType  = "2lep";
> ADD cut variables to private: part of class
> >   private: (~L20)
> >   
> >     double m_pTWcutVal          = 150e3;
> >     double m_pTZcutVal          = 150e3;  

## 7) Remove MET Trigger code, variables and declalarations from TriggerTool_VHbb1lep.cxx, TriggerTool_VHbb1lep.h, TriggerTool_VHbb2lep.cxx and TriggerTool_VHbb2lep.h.
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb1Lep.h
vim CxAODOperation_VHbb/CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb2Lep.h
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb1Lep.cxx
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.cxx
~~~
> REMOVE all code duplicated in these files in TriggerTool_VHbb.cxx and TriggerTool_VHbb.h

## 8) Change function calls and add includes in AnalysisReader_VHQQ2Lep.cxx and 1L equivalent
~~~
vim CxAODOperation_VHbb/CxAODReader_VHbb/Root/AnalysisReader_VHQQ1Lep.cxx
vim CxAODOperation_VHbb/CxAODReader_VHbb/Root/AnalysisReader_VHQQ2Lep.cxx
~~~
> ADD New dependencies
> >   #include "CxAODTools_VHbb/TriggerTool_VHbb.h" (~L10)
> CHANGE Function Call
> >    bool triggerDec = ((TriggerTool_VHbb[X]Lep *)m_triggerTool) -> bool triggerDec = ((TriggerTool_VHbb *)m_triggerTool)
> For 2L change variable names 
> >   if (m_METTriggerin2L || m_METMuonTriggerin2L) -> (m_METTriggerin2L || m_METMuonTriggerin2L)


## 9) Change names of variables in AnalysisReader_VHQQ.cxx and AnalysisReader_VHQQ.h
~~~
vim CxAODOperation_VHbb/CxAODReader_VHbb/Root/AnalysisReader_VHQQ.cxx
vim CxAODOperation_VHbb/CxAODReader_VHbb/Root/AnalysisReader_VHQQ.h
~~~
> CHANGE New dependencies (L49/50 || ~L522/523)
> >  m_applyMETTriggerto2L -> m_METTriggerin2L(false),
> >  m_METMuonTriggerCombin2L -> m_doMETMuonTrigger(false),

When you think you have finished then you can refresh your local changes and re-build. Make sure that you go into each directory in /source/ and do git pull to make sure that each sub-repository is up to date. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ..
~~~
Now we need to run the same plots as before to test the code can reproduce the old plots. For the Boosted Analysis: Ensuring first that 
~~~
vim CxAODOperation_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
> >   ANASTRATEGY="Merged" (L235) 
> >   DO2LMETTRIGGER="false" (L266)
> >   DO2LMETANDMUONTRIGGER="false" (L267)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_oldTrigger_TEST 2L a VHbb CUT D1 32-15 2lsignal none 1
~~~
vim CxAODOperation_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
> >   DO2LMETTRIGGER="true" (L266)
~~~
cd run/
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_newTrigger_TEST 2L a VHbb CUT D1 32-15 2lsignal none 1

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master/","SignalBoosted","old","newest","SIGNAL.root","2L","32-15","ade","CUT", "D1","SR")'
~~~
