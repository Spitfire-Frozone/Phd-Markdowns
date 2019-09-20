# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about the testing new trigger regimes for the 2 Lepton Analysis for VHbb. #

## VHbb 2 Lepton Trigger Study Part 2 ##

Last Edited: 30-08-2019
-------------------------------------------------------------------------------

# Setup Script
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
cp /afs/cern.ch/user/v/vhbbframework/public/CxAODBootstrap_VHbb/scripts/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_master_july2019 1 1
~~~
>   source getMaster.sh r31-10 CxAODFramework_tag_r31-10 1 1 [for a tag]


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
>   >   
>   >   METANDMUONTRIGGERCOMBin2L -> DOMETANDMUONTRIGGER

## 2) Proliferate new Variables to config files
~~~
vim CxAODOperation_VHbb/CxAODReader_VHbb/data/framework-read-automatic.cfg
~~~
> MOVE MET and muon trigger flag to general part of code (~L60 from ~L104)
>   >   
>   >   bool doMETMuonTrigger               = DOMETANDMUONTRIGGER # if true, it merges allows running of the MET and Muon triggers simultaneously in a logical 'or'

> CHANGE name of 2L MET trigger application flag in # 2 LEPTON SPECIFIC SWITCHES part of the code (~L103)
>   >   
>   >   applyMETTriggerto2L -> METTriggerin2L

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
>   >   
>   >    m_applyMETTriggerto2L -> m_METTriggerin2L
>   >    m_METandMuonTriggerCombin2L -> m_doMETMuonTrigger
> CHANGE conditional initialisation of the trigger
>   >   
>   >    if (m_METTriggerin2L) -> if (m_METTriggerin2L || m_doMETMuonTrigger)
> CUT all of ""bool TriggerTool_VHbb::getDecisionAndSFwithMET(double& triggerSF)"" from TriggerTool_VHbb2Lep.cxx to TriggerTool_VHbb.cxx

## 4) Add necessary config variables from TriggerTool_VHbb1Lep.cxx and TriggerTool_VHbb2Lep.cxx to TriggerTool_VHbb.cxx
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb.cxx
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb1Lep.cxx
vim CxAODOperation_VHbb/CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.cxx
~~~
> CUT all of lines of config declarations relating to m_doMETMuonTrigger, m_METTriggerin2L, m_do_1L_MuonTrigger and m_pT(W/Z)cutVal from TriggerTool_VHbb1lep::TriggerTool_VHbb1lep and TriggerTool_VHbb2lep::TriggerTool_VHbb2lep to TriggerTool_VHbb::TriggerTool_VHbb (L6)
>   >  
>   >    TriggerTool_VHbb::TriggerTool_VHbb(ConfigStore& config) 
>   >      :TriggerTool(config),
>   >    
>   >        m_do_1L_MuonTrigger(false),
>   >        m_METTriggerin2L(false),
>   >        m_doMETMuonTrigger(false),
>   >        m_analysisType("2lep"),
>   >        m_pTWcutVal(150e3),
>   >        m_pTZcutVal(150e3)
>   >    {	
>   >      m_config.getif<bool>("doMETMuonTrigger", m_doMETMuonTrigger); 
>   >      m_config.getif<bool>("do_1L_MuonTrigger", m_do_1L_MuonTrigger);
>   >      m_config.getif<bool>("METTriggerin2L", m_METTriggerin2L);
>   >      m_config.getif<std::string>("analysisType", m_analysisType);
>   >      m_config.getif<double>("Trig::pTWcutVal", m_pTWcutVal);
>   >      m_config.getif<double>("Trig::pTZcutVal", m_pTZcutVal);
>   >    
>   >    }

## 5) Change 2L MET trigger code to work with 1L instances of the function as well.
Here analysis-dependent decisions need to be made as ptV is made up of different objects depending on the analysis. Declare all objects but only fill them depending on analysis type.
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb.cxx
~~~
> ADD initialasion of m_analysisType
>   >   
>   >   m_analysisType("2lep") (~L13) 

> ADD boolean to store the decision to not use the MET trigger
>   >   
>   >   bool notUsingMETTrigger; (~L45)

> CHANGE declaration of variables and calculation of ptV to something like (~L47) 
>  >   
>	 >	    const xAOD::Muon* muon1 = m_muons.at(0);
>  >	    const xAOD::Muon* muon2; //Declaration of muon here just in case 2L analysis calls this. Will just exist for the 1L one and not do anything. 
>	 >	    TLorentzVector pTVVec, muVec1;
>	 >	    muVec1.SetPtEtaPhiM(muon1->pt(), 0, muon1->phi(), 0);
>	 >	    
>	 >	    if (m_analysisType == "1lep"){ //Constructs PtV from met and the muon
>	 >	        TLorentzVector metVec;
>	 >	    	  metVec.SetPtEtaPhiM(m_met->met(), 0, m_met->phi(), 0); //There will be actual met in the event
>	 >	    	  pTVVec = muVec1 + metVec; // WVecT
>	 >	    	  if (pTVVec.Pt() < m_pTWcutVal || m_do_1L_MuonTrigger) {notUsingMETTrigger = true}
>	 >	  
>  >	    } else if (m_analysisType == "2lep"){ //Constructs PtV from the two muons
>	 >	    	  TLorentzVector muVec2;
>	 >	    	  muon2 = m_muons.at(1);
>  >	    	  muVec2.SetPtEtaPhiM(muon2->pt(), 0, muon2->phi(), 0);
>	 >	    	  pTVVec = muVec1 + muVec2; //ZVecT
>	 >	    	  if (pTVVec.Pt() < m_pTZcutVal) {notUsingMETTrigger = true}
>	 >	  
>  >	    } else { //Add instance to catch accidental calls from 0L analysis
>	 >	    	Error("TriggerTool_VHbb::getDecisionAndSFwithMET()",
>	 >	              "Currenlty this method cannot work with the 0L analysis");
>	 >	      exit(1);
>	 >	    } 

> ADD analysis-chanel dependence of MET proxy filling (~L92)
>   >   
>   >   double METx , METy;
>	  >	  if (m_analysisType == "1lep"){
>	  >	  	  METx = pTVVec.Px();
>	  >	  	  METy = pTVVec.Py();
>	  >   } else if (m_analysisType == "2lep"){
>	  >       METx = pTVVec.Px() + met->mpx();
>	  >	   	  METy = pTVVec.Py() + met->mpy();
>	  >	  }

> CHANGE selective setting of event variables(~L104-L112)
>   >   if(useMETAndMuonAtHighPtV)   setMuons({muon1, muon2}); ->   if(useMETAndMuonAtHighPtV && m_analysisType == "1lep")   setMuons({muon1});
>   >   else if(useMETAndMuonAtHighPtV && m_analysisType == "2lep")   setMuons({muon1, muon2});

## 6) Declare MET Trigger code and config variables in TriggerTool.h
~~~
vim CxAODOperation_VHbb/CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb.h
~~~
> ADD function declaration and config variables to public: part of class
> >   
> >     public: (~L8)
> >     [...]
> >     bool getDecisionAndSFwithMET(double& triggerSF);
> >     [...]
> >     //Needed to add MET Trigger to 1L and 2L Analysis
> >     bool m_do_1L_MuonTrigger    = false;
> >     bool m_METTriggerin2L       = false;
> >     bool m_doMETMuonTrigger     = false; 
> >     std::string m_analysisType  = "2lep";
> ADD cut variables to private: part of class
> >   
> >     private: (~L20)
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
> >   
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
> CHANGE new dependencies (L49/50 || ~L522/523)
>  >   
>  >     m_applyMETTriggerto2L -> m_METTriggerin2L(false),
>  >     m_METMuonTriggerCombin2L -> m_doMETMuonTrigger(false),

# Testing
When you think you have finished then you can refresh your local changes and re-build. Make sure that you go into each directory in /source/ and do git pull to make sure that each sub-repository is up to date.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
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
> CHANGE config variables
>  >   
>  >     ANASTRATEGY="Merged" (L235) 
>  >     DO2LMETTRIGGER="false" (L266)
>  >     DO2LMETANDMUONTRIGGER="false" (L267)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_oldTrigger_TEST 2L a VHbb CUT D1 32-15 2lsignal none 1

vim ../source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
>  >    DO2LMETTRIGGER="true" (L266)
~~~
cd run/
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_newestTrigger_TEST 2L a VHbb CUT D1 32-15 2lsignal none 1

~~~
Next one should check that all the inputs were fine. Resubmitting failes ones as necessary.
~~~
cd run/SignalBoosted_oldTrigger_TEST
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1

cd ../SignalBoosted_newestTrigger_TEST
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
~~~
Now to create the sample SIGNAL root files to test. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
cd run/SignalBoosted_oldTrigger_TEST/Reader_2L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root QQWH.root QQZH.root
rm GGZH.root QQWH.root QQZH.root
cd -

cd run/SignalBoosted_newestTrigger_TEST/Reader_2L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root QQWH.root QQZH.root
rm GGZH.root QQWH.root QQZH.root
cd -

vim ../TriggerStudyPlots.cxx
~~~
> CHANGE vector of samples to the signal ones
>   >     std::vector samples = {"ggZllH125", "qqZllH125", "qqWlvH125"}; //Signal
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","SignalBoosted","old","newest","SIGNAL.root","2L","32-15","a","CUT","D1","SR","_TEST","_TEST")'
cd run
mv SignalBoosted-oldandnewest_TriggerPlots SignalBoosted-oldandnewest_a_TriggerPlots
cd SignalBoosted-oldandnewest_a_TriggerPlots
imgcat *.pdf
~~~

# Merging Updates
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
vim CxAODTools_VHbb/CxAODTools_VHbb/TriggerTool_VHbb.h
~~~
- Remove initialisation of MET Trigger Variables
- Harmonise naming procedures. m_do_1L_MuonTrigger -> m_doMuonTriggerin1L & m_METTriggerin2L -> m_doMETTriggerin2L
~~~
vim CxAODTools_VHbb/Root/TriggerTool_VHbb.cxx
vim CxAODTools_VHbb/Root/TriggerTool_VHbb2Lep.cxx
vim CxAODTools_VHbb/Root/TriggerTool_VHbb1Lep.cxx
vim CxAODReader_VHbb/Root/AnalysisReader_VHQQ.cxx
vim CxAODReader_VHbb/Root/AnalysisReader_VHQQ2Lep.cxx
vim CxAODReader_VHbb/CxAODReader_VHbb/AnalysisReader_VHQQ.h
vim CxAODReader_VHbb/data/framework-read.cfg
vim CxAODReader_VHbb/data/framework-read-automatic.cfg
~~~
- Propogate name changes throughout code.
~~~
vim CxAODTools_VHbb/Root/TriggerTool_VHbb.cxx
~~~
- Remove excess if statements 


# Producing Final Plots
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
cd ../run

vim ../source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
>  >    DO2LMETTRIGGER="false" (L266)
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_oldTrigger 2L a,d,e VHbb CUT D1 32-15 none none 1
~~~

But since there have been changes to 1L as well as 2L, one needs to generate plots for 1L to show that the code doesn't affect the plots. 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_oldTrigger 1L a VHbb CUT D1 32-15 none none 1

vim ../source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
>  >    DO2LMETTRIGGER="true" (L269)
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger 2L a,d,e VHbb CUT D1 32-15 none none 1

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger 1L a VHbb CUT D1 32-15 none none 1
~~~

We also want to generate some final plots for the Resolved analysis as well.
~~~
vim ../source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
>  >    ANASTRATEGY="Resolved" (L235)
>  >    DO2LMETTRIGGER="false" (L269)
>  >    DOPTVSPLITTING250GEV="true" (L828)
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullResolved_oldTrigger 2L a,d,e VHbb MVA D1 32-15 none none 1

vim ../source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
>  >    DO2LMETTRIGGER="true" (L269)
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullResolved_newestTrigger 2L a,d,e VHbb MVA D1 32-15 none none 1
~~~

Once those files have been submitted to the grid and have run. Next one should check that all the inputs were fine. Resubmitting failes ones as necessary.
~~~
cd run/FullBoosted_oldTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_d_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

cd ../FullBoosted_newestTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_d_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

cd ../FullResolved_oldTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_MVA_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_d_MVA_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_MVA_D1

cd ../FullResolved_newestTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_MVA_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_d_MVA_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_MVA_D1
~~~
Once all the jobs have succeeded. Now to hadd the files
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 

cd run/FullBoosted_oldTrigger
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll2LadeCUT.sh
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll1LaCUT.sh
cd ../FullBoosted_newestTrigger
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll2LadeCUT.sh
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll1LaCUT.sh

cd ../FullResolved_oldTrigger
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll2LadeMVA.sh
cd ../FullResolved_newestTrigger
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll2LadeMVA.sh
~~~
And now the money plots
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/

vim ../TriggerStudyPlots.cxx
~~~
> CHANGE vector of samples to the signal ones (~L110)
>   >     std::vector samples = {"ggZllH125", "qqZllH125", "qqWlvH125"}; //2L Signal

> CHANGE the legend of the plots (~L495)
>   >     legend.AddEntry(TProfileInputPlot2, Form("2L Signal (%) Gain = %f",improvement), "-");   
~~~

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullBoosted","old","newest","SIGNAL.root","2L","32-15","ade","CUT","D1","SR")'
mv run/FullBoosted-oldandnewest_TriggerPlots run/SignalBoosted-oldandnewest_ade_TriggerPlots

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullResolved","old","newest","SIGNAL.root","2L","32-15","ade","MVA","D1","SR")'
mv run/FullResolved-oldandnewest_TriggerPlots run/SignalResolved-oldandnewest_ade_TriggerPlots


vim ../TriggerStudyPlots.cxx
~~~
> CHANGE vector of samples to the everything ones (~L110)
>   >     std::vector samples = {"ggZllH125", "ggZvvH125", "ggZZ", "qqWlvH125", "qqZllH125", "qqZvvH125", "stops", "stopt", "stopWt", "ttbar", "Wbb", "Wbc", "Wbl", "Wcc", "Wcl", "Wl", "WZ", "Zbb","Zbc","Zbl","Zcc", "Zcl", "Zl", "ZZ"}; //Everything 

> CHANGE the legend of the plots
>   >     legend.AddEntry(TProfileInputPlot2, Form("2LS+B(%) Gain = %f",improvement), "-");   
~~~

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullBoosted","old","newest","2LEPALL.root","2L","32-15","ade","CUT","D1","SR")'
mv run/FullBoosted-oldandnewest_TriggerPlots run/FullBoosted-oldandnewest_ade_TriggerPlots

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullBoosted","old","newest","2LEPALL.root","1L","32-15","a","CUT","D1","SR")'
mv run/FullBoosted-oldandnewest_TriggerPlots run/FullBoosted-oldandnewest_1L_a_TriggerPlots

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullResolved","old","newest","2LEPALL.root","2L","32-15","ade","MVA","D1","SR")'
mv run/FullResolved-oldandnewest_TriggerPlots run/FullResolved-oldandnewest_ade_TriggerPlots
~~~

## Comparison with the master branch.
Although technically it is a box-ticking exercise, it is important to test the changes themselves such that the act of putting in a flag, even though it is set to false does nothing. For this we will need to test the 'old' version of the 1L trigger on the development branch to a run on the master. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
~~~
Ensure that ALL the sub-directories are up-to-date. Here I have only put the ones I've edited, but all the others have to be edited as well.  
~~~
cd source/CxAODOperations_VHbb
git checkout master
git pull

cd ../CxAODTools_VHbb
git checkout master
git pull
 
cd ../CxAODReader_VHbb
git checkout master
git pull
 
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
lsetup 'lcgenv -p LCG_94 x86_64-centos7-gcc8-opt numpy'
cd ../run

vim ../source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
> CHANGE 
>  >    ANASTRATEGY="Merged" (L235)
>  >    DOPTVSPLITTING250GEV="true" (L752)
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_master 1L,2L a VHbb CUT D1 32-15 none none 1
~~~
So now these inputs just need to be tested and hadded in the usual way
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_july2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 

cd run/FullBoosted_masterTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1
~~~
>    qqZllHbbJ_PwPy8MINLO-0, qqWlvHbbJ_PwPy8MINLO-0, qqWlvHbbJ_PwPy8MINLO-9, WenuB_Sh221-0, WmunuB_Sh221-34, ttbar_dilep_PwPy8-36
~~~
vim Reader_1L_32-15_a_CUT_D1/submit/segments
~~~
>    1, 3, 12, 36, 121, 364
~~~
cd Reader_1L_32-15_a_CUT_D1
./submit/run 1
./submit/run 3
./submit/run 12
./submit/run 36
./submit/run 121
./submit/run 364

cd ..
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
~~~
>    qqZllHbbJ_PwPy8MINLO-0, qqZllHccJ_PwPy8MINLO-4, ttbar_nonallhad_PwPy8-9
~~~
vim Reader_2L_32-15_a_CUT_D1/submit/segments
~~~
>    1, 15, 159
~~~
cd Reader_2L_32-15_a_CUT_D1
./submit/run 1
./submit/run 15
./submit/run 159
~~~
Once all the jobs have succeeded. Now to hadd the files
~~~
cd run/FullBoosted_masterTrigger
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll1LaCUT.sh

cd Reader_2L_32-15_a_CUT_D1/fetch
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAllDeleteData.sh
~~~
Now to create one set of plots against the master.
~~~
vim ../TriggerStudyPlots.cxx
~~~
> CHANGE vector of samples to the everything ones (~L110)
>   >     std::vector samples = {"ggZllH125", "ggZvvH125", "ggZZ", "qqWlvH125", "qqZllH125", "qqZvvH125", "stops", "stopt", "stopWt", "ttbar", "Wbb", "Wbc", "Wbl", "Wcc", "Wcl", "Wl", "WZ", "Zbb","Zbc","Zbl","Zcc", "Zcl", "Zl", "ZZ"}; //Everything 

> CHANGE the legend of the plots
>   >     legend.AddEntry(TProfileInputPlot2, Form("2LS+B(%) Gain = %f",improvement), "-");   
~~~

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullBoosted","master","old","2LEPALL.root","2L","32-15","a","CUT","D1","SR")'
mv run/FullBoosted-masterandold_TriggerPlots run/FullBoosted-masterandold_2L_a_TriggerPlots

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_july2019/","FullBoosted","master","old","2LEPALL.root","1L","32-15","a","CUT","D1","SR")'
mv run/FullBoosted-masterandold_TriggerPlots run/FullBoosted-masterandold_1L_a_TriggerPlots
~~~

## Post-milestone checks.
Now after the plots have been circulated that show the inclusion of this trigger change is both good for the analysis and not going to change anything for 1L, there still remains an issue of the missing trigger in data15 and the MC16a. Since those events do not have  MET trigger, when the code snippet switched from triggering on muons to MET, these events would just not be triggered. There were two main schools of thought here for an investigation to rectify this. 

- Trigger on the Single Lepton
- Use a different MET trigger for data15. 

But if there is a patch in the derivation that fixes this, then this isn't needed. The first option is harder to impliment but the second one can be done reasonably quickly. Since the data15 events have the trigger used in the data16, we can check the effect of replacing the xe70 trigger with this one on the 1L analysis. The theory is that all the events that pass the xe90-[...] trigger should pass the xe70 trigger.

After checking out a new version of the Analysis
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 

vim source/xAODOperations_VHbb/scripts/submitReader.sh
~~~
>   CHANGE 2L trigger flag to true (~L281)
>   >    DO2LMETTRIGGER = true
~~~
vim source/CxAODTools/Root/TriggerTool.cxx
~~~
>   COMMENT IN/OUT the old data15 trigger for the while analysis depending on what you want to test (L102)
>   >    //ADD_TRIG(HLT_xe70,             any, data15, data15);
>   >    //ADD_TRIG(HLT_xe90_mht_L1XE50,  any, data16A, data16BD3);

>   CHANGE the data16A-B MET trigger to include the data15 period (L104)
>   >    ADD_TRIG(HLT_xe90_mht_L1XE50,  any, data15, data16BD3); // Trial MET trigger configuration
~~~
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run
~~~
There are 4 tests to run

- TEST 1
Replacing the xe70 trigger with xe90 one in 1L and comparing it to the 1L standard new implimentation
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger_xe90 1L a VHbb CUT D1 32-15 none none 1

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/run/FullBoosted_newestTrigger_xe90
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll1LaCUT.sh

vim  ../../../TriggerStudyPlots.cxx
~~~
>    COMMENT in the sample list with everything in it apart from data. (~L114)
>   >    = {"ggZllH125", "ggZvvH125", "ggZZ", "qqWlvH125", "qqZllH125", "qqZvvH125", "stops", "stopt", "stopWt", "ttbar", "Wbb", "Wbc", "Wbl", "Wcc", "Wcl", "Wl", "WZ", "Zbb","Zbc","Zbl","Zcc", "Zcl", "Zl", "ZZ"}; //Everything 
>    CHANGE legend sub-titles (~L516)                                                                        
>   >   Form("Bkg (%) Gain ..." -> Form("S+B (%) Gain ..."                                                
~~~

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_august2019/","FullBoosted","newest","newest","2LEPALL.root","1L","32-15","a","CUT","D1","SR","","_xe90")'
mv run/FullBoosted-newestandnewest_TriggerPlots run/FullBoosted-xe70vsxe90_1L_a_TriggerPlots

~~~
- TEST 2
Adding the xe90 trigger to the xe70 trigger in 1L ...

- TEST 3
... and 2L and comparing it to the standard new implimentation
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger_xe70xe90 1L,2L a VHbb CUT D1 32-15 none none 1

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/run/FullBoosted_newestTrigger_xe90xe70
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll1LaCUT.sh

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_august2019/","FullBoosted","newest","newest","2LEPALL.root","1L","32-15","a","CUT","D1","SR","","_xe70xe90")'
mv run/FullBoosted-newestandnewest_TriggerPlots run/FullBoosted-xe70vsxe70xe90_1L_a_TriggerPlots

python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1

cd Reader_2L_32-15_a_CUT_D1/fetch
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddAll.sh

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_august2019/","FullBoosted","newest","newest","2LEPALL.root","2L","32-15","a","CUT","D1","SR","","_xe70xe90")'
mv run/FullBoosted-newestandnewest_TriggerPlots run/FullBoosted-xe70vsxe70xe90_2L_a_TriggerPlots

~~~
- TEST 4
And then for the 2L data case, because I don't keep the data for the master I will need to re-run it. 
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger_xe70 2L a VHbb CUT D1 32-15 data15 data15 1

cd FullBoosted_newestTrigger_xe70
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1

cd Reader_2L_32-15_a_CUT_D1/fetch
hadd DATA15.root hist-data15* 
cd ../../../..

vim  ../TriggerStudyPlots.cxx
~~~
>   ADD new samples to run over (~L112)
>   >  {"data"}; //data only   
>   COMMENT out running over all samples (~L113)
>    CHANGE legend sub-titles (~L516, ~L525, ~L566)                                                                        
>   >   Form("S+B (%) Gain ..." -> Form("data (%) Gain ..."  
>   >   #it{#sqrt{s}} = 13 TeV, MC16a" -> #it{#sqrt{s}} = 13 TeV, Data15"
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_august2019/","FullBoosted","newest","newest","DATA15.root","2L","32-15","a","CUT","D1","SR","","_xe70xe90")'
mv run/FullBoosted-newestandnewest_TriggerPlots run/FullBoosted-xe70vsxe70xe90_2L_data_a_TriggerPlots
~~~

# Running WSMaker
If you want to compare the significance this change has on the analysis you will need to run the fit. This time instead of the nominal 2L inputs you will use the ones generated by the reader when running the study. To do this however you need to run over the data as well. 

When you run the command 
>    ../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullResolved_newestTrigger 2L a,d,e VHbb MVA D1 32-15 none none 1

to generate the Full Run 2 impact of the MET trigger change, instead of running 
>     source VHbbHaddAll2LadeCUT.sh
you run 
>     source VHbbHaddBoosted2LInputs.sh

If you need to you should only have to re-run over the data for MC16ade
>      ../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullResolved_newestTrigger 2L a,d,e VHbb MVA D1 32-15 data15 data16 1

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_august2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
cd run/FullBoosted_newestTrigger
source /afs/cern.ch/work/d/dspiteri/VHbb/VHbbHaddBoosted2LInputs.sh
~~~
