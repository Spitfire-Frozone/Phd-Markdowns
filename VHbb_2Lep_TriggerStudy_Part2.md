# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about the testing new trigger regimes for the 2 Lepton Analysis for VHbb. #

## VHbb 2 Lepton Trigger Study Part 2 ##

Last Edited: 21-11-2019
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

### Turns out that was the wrong trigger.
The HLT_xe90_mht_L1XE50 trigger was only partially available for the 2015 data period. After checking the next unprescaed trigger that is present for the whole period is the HLT_xe80_mht_L1XE50 but since you have already the master for comparison, you can just run the new triggers. Obviously the analysis was out of date and a new one needed to be checked out.
~~~

cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
cp /afs/cern.ch/user/v/vhbbframework/public/CxAODBootstrap_VHbb/scripts/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_master_october2019 1 1

cd CxAODFramework_master_october2019/
~~~
Now we want to pull in the branches we were working on earlier. 
~~~
cd source/CxAODReader_VHbb
git checkout --track origin/master-dspiteri-2L-METTriggerStudy
cd ../../source/CxAODOperations_VHbb
git checkout --track origin/master-dspiteri-2L-METTriggerStudy
~~~
Just have to make sure that this flag is turned on, but this should be the default on the branch we are on.
~~~
vim source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
>   CHANGE 2L trigger flag to true (~L281)
>   >    DO2LMETTRIGGER = true

For the first run, nothing else should need to change
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger 1L,2L a VHbb CUT D1 32-15 2lsignal none 1
mv FullBoosted_newestTrigger SignalBoosted_newestTrigger

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger 2L a VHbb CUT D1 32-15 data15 none 1
mv FullBoosted_newestTrigger DataBoosted_newestTrigger
~~~
Now we need to change the trigger to run over. 
~~~
vim source/CxAODTools/Root/TriggerTool.cxx
~~~
>   COMMENT IN/OUT the old data15 trigger for the while analysis depending on what you want to test (L102)
>   >    //ADD_TRIG(HLT_xe70,             any, data15, data15);                                                           
>   >    ADD_TRIG(HLT_xe90_mht_L1XE50,  any, data16A, data16BD3);                                                 

>   CHANGE the data16A-B MET trigger to include the data15 period (L104)                                     
>   >    ADD_TRIG(HLT_xe80_mht_L1XE50,  any, data15, data15); // Trial MET trigger configuration                             

Since this trigger is not yet in the analysis, one needs to ensure that it's added to the CommonProperties of the analysis. 
~~~
vim source/CxAODTools/Root/CommonProperties.cxx
~~~
>    LOOK for the line and if not present, add it in. (~L580)
>   >    PROPERTY_INST( Props , int , passHLT_xe80_mht_L1XE50     )
~~~
vim source/CxAODTools/Root/CommonProperties.cxx
~~~
>    LOOK for the line and if not present, add it in. (~L684)
>   >    PROPERTY_DECL( Props , int , passHLT_xe80_mht_L1XE50     )

Then run as normal.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger_xe80 1L,2L a VHbb CUT D1 32-15 2lsignal none 1

mv FullBoosted_newestTrigger_xe80 SignalBoosted_newestTrigger_xe80

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger_xe80 2L a VHbb CUT D1 32-15 data15 none 1

mv FullBoosted_newestTrigger_xe80 DataBoosted_newestTrigger_xe80
~~~
Then you need to check that all of the jobs were sucessfully submitted.
~~~
cd SignalBoosted_newestTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

cd ../SignalBoosted_newestTrigger_xe80
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

cd ../DataBoosted_newestTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1

cd ../DataBoosted_newestTrigger_xe80
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
~~~
Once all of the jobs are present then you can hadd the outputs to produce the things we run over.  
~~~
cd SignalBoosted_newestTrigger/Reader_1L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root GGZZ.root QQWH.root QQZH.root
rm GGZH.root GGZZ.root QQWH.root QQZH.root
cd ../../Reader_2L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root GGZZ.root QQWH.root QQZH.root
rm GGZH.root GGZZ.root QQWH.root QQZH.root

cd ../../../SignalBoosted_newestTrigger_xe80/Reader_1L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root GGZZ.root QQWH.root QQZH.root
rm GGZH.root GGZZ.root QQWH.root QQZH.root
cd ../../Reader_2L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root GGZZ.root QQWH.root QQZH.root
rm GGZH.root GGZZ.root QQWH.root QQZH.root

cd ../../../DataBoosted_newestTrigger/Reader_2L_32-15_a_CUT_D1/fetch/
hadd DATA15.root hist-data*
cd ../../../DataBoosted_newestTrigger_xe80/Reader_2L_32-15_a_CUT_D1/fetch/
hadd DATA15.root hist-data*

cd ../../../..
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_october2019/","DataBoosted","newest","newest","DATA15.root","2L","32-15","a","CUT","D1","SR","","_xe80")'
mv run/DataBoosted-newestandnewest_TriggerPlots run/DataBoosted-xe70vsxe80_2L_a_TriggerPlots

vim  ../TriggerStudyPlots.cxx
~~~
>   COMMENT IN new samples to run over (~L112)                                                                      
>   > {"ggZllH125", "qqZllH125", "qqWlvH125"}; // 2L Signal                                                
>   > //= {"data"} //data only                                                                                          

>    CHANGE legend sub-titles (~L516, ~L525, ~L566)                                                                    
>   >   Form("Data (%) Gain ..." -> Form("Signal (%) Gain ..."                                                 
>   >   #it{#sqrt{s}} = 13 TeV, Data15" -> #it{#sqrt{s}} = 13 TeV, MC16a"                                                 
~~~
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_october2019/","SignalBoosted","newest","newest","SIGNAL.root","2L","32-15","a","CUT","D1","SR","","_xe80")'
mv run/SignalBoosted-newestandnewest_TriggerPlots run/SignalBoosted-xe70vsxe80_2L_a_TriggerPlots

root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_october2019/","SignalBoosted","newest","newest","SIGNAL.root","1L","32-15","a","CUT","D1","SR","","_xe80")'
mv run/SignalBoosted-newestandnewest_TriggerPlots run/SignalBoosted-xe70vsxe80_1L_a_TriggerPlots
~~~
Now we just need to test the addition of both the xe70 and the xe80 triggers at the same time. 
~~~
vim source/CxAODTools/Root/TriggerTool.cxx
~~~
>   COMMENT IN/OUT the old data15 trigger for the while analysis depending on what you want to test (L102)
>   >    ADD_TRIG(HLT_xe70,             any, data15, data15);                                                         
>   >    ADD_TRIG(HLT_xe80_mht_L1XE50,  any, data15, data15); // Trial MET trigger configuration                             
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_newestTrigger_xe70xe80 1L a VHbb CUT D1 32-15 2lsignal none 1

cd SignalBoosted_newestTrigger_xe70xe80
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_1L_32-15_a_CUT_D1

cd Reader_1L_32-15_a_CUT_D1/fetch/
hadd GGZH.root hist-ggZ*
hadd GGZZ.root hist-ggZqq*
hadd QQWH.root hist-qqW*
hadd QQZH.root hist-qqZ*
hadd SIGNAL.root GGZH.root GGZZ.root QQWH.root QQZH.root
rm GGZH.root GGZZ.root QQWH.root QQZH.root

cd ../../../..
root -b -l -q '../TriggerStudyPlots.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/", "CxAODFramework_master_october2019/","SignalBoosted","newest","newest","SIGNAL.root","1L","32-15","a","CUT","D1","SR","","_xe70xe80")'
mv run/SignalBoosted-newestandnewest_TriggerPlots run/SignalBoosted-xe70vsxe70xe80_1L_a_TriggerPlots
~~~

Since the number of data evens in these bins are quite low, the last test that we want to do is to check how many data would pass the standard trigger.
Now have to make sure that this flag is turned off, which is NOT the default of the branch we are on.
~~~
vim source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
>   CHANGE 2L trigger flag to true (~L281)
>   >    DO2LMETTRIGGER = true

For the first run, nothing else should need to change
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 DataBoosted_oldTrigger 2L a VHbb CUT D1 32-15 data15 none 1

cd DataBoosted_oldTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D1
~~~
If nothing is wrong, then
~~~
cd Reader_2L_32-15_a_CUT_D1/fetch/
hadd DATA15.root hist-data15-*
~~~
## WSMaker Input Tests

The last job then is to create a 2L boosted input with this change and to run with systematics. This is going to be a very large so you will need to be able to run somewhere that has some space. Your /eos space - CERNBox has 1T of space. However you can write to eos directly if you follow [this](https://gitlab.cern.ch/CxAODFramework/CxAODReader/blob/master/Root/AnalysisReader.cxx#L177) . Since the MET Trigger changes introduced do not have any syst associated with them and the lepton trigger syst are tiny (they get pruned in fact), and you don't add any for the met trigger, systematics do NOT need to be run over. 

Hence there will be two inputs that have to be introduced. Both without systematics, but one with DO2LMETTRIGGER="true" and the other with DO2LMETTRIGGER="false". The ones without DO2LMETTRIGGER="true" are the nominal produced by the group. One could run the DO2LMETTRIGGER="true" as a sanity check, but the nominal should be the inputs produced by the group. 
~~~
vim CxAODOperation_VHbb/scripts/submitReader.sh
~~~
>   CHANGE Analysis strategy to be merged (ensure it is changed)                                                            
>   >    ANASTRATEGY="Merged"                                                                                               

>   ADD limitations to the size of the outputs (~L198)                                                                      
>   >    DOONLYINPUTS="true"     
>   >    JOBSIZELIMITMB="15000"

>   CHANGE to run over systematics (~L205,L220)                                                                          
>   >    NOMINALONLY="true"                                                                                                

>   CHANGE to run over new Trigger regime (~L303)                                                                     
>   >    DO2LMETTRIGGER="true"                                                                                              
 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_newestTrigger 2L a,d,e VHbb CUT D 32-15 none none 1
~~~
After submission ensure that none of the jobs failed!
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
cd run/FullBoosted_newestTrigger

python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_d_CUT_D
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D
~~~
Now lets hadd our stuff together to make the final input! You will need to rename it such that WSMaker can get info from the name. v1 = muon trigger enabled and v2 = MET trigger enabled.
~~~
source ../../../VHbbHaddAll2LadeCUT.sh
mv Reader_2L_32-15_ade_CUT_D/fetch/2LEPINPUT.root Reader_2L_32-15_ade_CUT_D/fetch/LimitHistograms.VH.llbb.13TeV.mc16ade.Glasgow.v2.VR.root
~~~
Now if you want to run over the old regime. 
~~~
vim CxAODOperation_VHbb/scripts/submitReader.sh
~~~

>   CHANGE to run over new Trigger regime (~L303)                                                                     
>   >    DO2LMETTRIGGER="false"                                                                                            
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run

../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 FullBoosted_oldTrigger 2L a,d,e VHbb CUT D 32-15 none none 1

~~~
Once again after submission check that none of the jobs failed!
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
cd run/FullBoosted_oldTrigger

python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_a_CUT_D
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_d_CUT_D
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D
~~~
Now lets hadd our stuff together to make the final input for my changes! v1 = MET trigger not enabled.
~~~
source ../../../VHbbHaddAll2LadeCUT.sh
mv Reader_2L_32-15_ade_CUT_D/fetch/2LEPINPUT.root Reader_2L_32-15_ade_CUT_D/fetch/LimitHistograms.VH.llbb.13TeV.mc16ade.Glasgow.v1.VR.root
~~~

# Running Boosted WSMaker
If you want to compare the significance this change has on the analysis you will need to run the fit. This time in addition to the nominal 2L inputs you will use the ones generated by the reader when running the study. To do this however you need to run over the data as well. 

So now lets get a copy of the Boosted WSMaker
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git 
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_boostedVHbb.git
cd WSMaker_boostedVHbb
source setup.sh
mkdir inputs
~~~
We will need to run and split that new milestone inputs for 2L and compare them against the ones where the trigger regime has changed. Luckily for us the former has already been done and run and the split inputs can be found here:
> RESOLVED                                                                                                                 
> /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/TwoLep/r32-15_customCDI_20190814/v2/fullSysts
> BOOSTED                                                                                                                    
> /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/BoostedVHbb2019/TwoLep/LimitHistograms.VH.llbb.13TeV.mc16ade.UIOWAUSTC.v07.VR.root
> /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/BoostedVHbb2019/TwoLep/LimitHistograms.VH.llbb.13TeV.mc16ade.UIOWAUSTC.v07.VR.extension_nominal.root
If you want more details you can check out [The](https://github.com/Spitfire-Frozone/Phd-Markdowns/blob/master/VHbb_WSMakerHbbstat.md)  [input](https://github.com/Spitfire-Frozone/Phd-Markdowns/blob/master/VHbb_WSMakerHbbstat_Part2.md) [markdowns](https://github.com/Spitfire-Frozone/Phd-Markdowns/edit/master/VHbb_WSMakerHbbstat_Part3.md)
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
~~~
First we need to create a file to split the nominal inputs
~~~
vim inputConfigs/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05.txt
~~~
>   ADD newly generated inputs
>   >  ZeroLepton ZeroLep/LimitHistograms.VH.vvbb.13TeV.mc16ade.PISA.v3.VR.root      mJ                             
>   >  OneLepton  OneLep/LimitHistograms.VH.lvbb.13TeV.mc16ade.UCL.v2.VR.root        mJ                                   
>   >  TwoLepton  TwoLep/LimitHistograms.VH.llbb.13TeV.mc16ade.UIOWAUSTC.v07.VR.root mJ                                       
~~~
cd build && cmake ..
make -j10
cd ..
SplitInputs -v SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05 -inDir /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/BoostedVHbb2019
~~~
Then we need to create a file to split the new inputs for new 2L. This is more complicated and made easier by moving the samples you want locally

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
mkdir 012Linputs
cp /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_october2019/run/FullBoosted_newestTrigger/Reader_2L_32-15_ade_CUT_D/fetch/LimitHistograms.VH.llbb.13TeV.mc16ade.Glasgow.v2.VR.root  012Linputs/
cp /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/BoostedVHbb2019/OneLep/LimitHistograms.VH.lvbb.13TeV.mc16ade.UCL.v2.VR.root 012Linputs/
cp /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/BoostedVHbb2019/ZeroLep/LimitHistograms.VH.vvbb.13TeV.mc16ade.PISA.v3.VR.root 012Linputs/

vim inputConfigs/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_METTrigger.txt
~~~
>   ADD newly generated inputs
>   >  ZeroLepton LimitHistograms.VH.vvbb.13TeV.mc16ade.PISA.v3.VR.root      mJ                                         
>   >  OneLepton  LimitHistograms.VH.lvbb.13TeV.mc16ade.UCL.v2.VR.root       mJ                                            
>   >  TwoLepton  LimitHistograms.VH.llbb.13TeV.mc16ade.Glasgow.v2.VR.root   mJ                                              
~~~
cd build && cmake ..
make -j10
cd ..
SplitInputs -v SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_METTrigger -inDir /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb/012Linputs/
~~~
So nothing in this life is ever that easy though. By running defaultly on lxplus, these inputs have Z+jets extensions that are not present in the default inputs, so the improvement one would get by adding the MET Trigger to the 2L analysis is convoluted with the increase of adding the Z+jet extensions. Weitao kindly provided some samples which have only the extensions in. Assuming that the increase in Z+jet extensions and the MET trigger is uncorrelated, then these significances can be found and the relative increases divided out to isolate the increase solely due to the MET Trigger.
~~~
cp /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/BoostedVHbb2019/TwoLep/LimitHistograms.VH.llbb.13TeV.mc16ade.UIOWAUSTC.v07.VR.extension_nominal.root .

vim inputConfigs/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_Extensions.txt
~~~
>   ADD newly generated inputs
>   >  ZeroLepton LimitHistograms.VH.vvbb.13TeV.mc16ade.PISA.v3.VR.root      mJ                                         
>   >  OneLepton  LimitHistograms.VH.lvbb.13TeV.mc16ade.UCL.v2.VR.root       mJ                                  
>   >  TwoLepton  LimitHistograms.VH.llbb.13TeV.mc16ade.UIOWAUSTC.v07.VR.extension_nominal.root      mJ                       
~~~
cd build && cmake ..
make -j10
cd ..
SplitInputs -v SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_Extensions -inDir /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb/012Linputs/
~~~
Then you will want to edit the launcher. For the full fit you will want to run twice for each set of inputs. 
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L11)                                                                                    
>   >  version = "v05"                                                                                             
>   >  doPostFit = False                                                                                                

>    CHANGE all do cutbase block to 'true' (~L13)                                                                          
>   >  doCutBase = True         (~L17)                                                                                                                                                     
>    CHANGE variable such that you run over the new CR's (~L34)                                                             
>   >  doNewRegions = True                                                                                                       

>    CHANGE variables so you are running the observed 0L standalone fit (~L31-L41)                                           
>   >  channels = ["2"]         (~L34)                                                                                        
>   >  MCTypes = ["mc16ade"]    (~L37)                                                                                       

>   >  syst_type = ["MCStat"]   (~L41)      #["StatOnly"] not working                                                      

>    CHANGE variables to run on pre-fit Asimov (~L45)                                                                  
>   >  doExp = "1" # "0" to run observed, "1" to run expected only                                                           

>    CHANGE variables to run on the batch as this will be hefty job (~L48)                                                   
>   >  run_on_batch = False                                                                                             

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)
>   >  createSimpleWorkspace = True                                                                                
>   >  runP0 = True                                                                                                       
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = True                                                                                              
>   >  runRanks = True                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runToyStudy = False                                                                                                 

>    ADD additional debug plots for shape plots (~L60)                                                                
>   >  doplots = True                                                                                                        
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-baseline-CUT-doExp1
mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05.140ifb-2L-ade-STXS-baseline-CUT-doExp1_fullRes_boostedVHbb_140ifb-2L-ade-STXS-baseline-CUT-doExp1_2_mc16ade_MCStat_mBB_VR output/140ifb-2L-ade-STXS-baseline-CUT-doExp1
~~~
To understand why this is being run twice one can consult Vol III of Dwayne PhD Research - p412
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE variables to run on post-fit Asimov (~L45)
>   >  doExp = "0" # "0" to run observed, "1" to run expected only

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)
>   >  createSimpleWorkspace = True                                                                                
>   >  runP0 = True                                                                                                          
>   >  runPulls = True                                                                                                      
>   >  runBreakdown = False                                                                                              
>   >  runRanks = False                                                                                                       
>   >  runLimits = False                                                                                                                                                                                                  
>    ADD additional debug plots for shape plots (~L60)                                                                
>   >  doplots = False                                                                                                      
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-baseline-CUT-doExp0
mv output/mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05.140ifb-2L-ade-STXS-baseline-CUT-doExp0_fullRes_boostedVHbb_140ifb-2L-ade-STXS-baseline-CUT-doExp0_2_mc16ade_MCStat_mBB_VR output/140ifb-2L-ade-STXS-baseline-CUT-doExp0
~~~
For the second set of inputs you do as before but changing the version name
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L11)                                                                                    
>   >  version = "v05_METTrigger"                                                                                            
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-METTrig-CUT-doExp0
mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_METTrigger.140ifb-2L-ade-STXS-METTrig-CUT-doExp0_fullRes_boostedVHbb_140ifb-2L-ade-STXS-METTrig-CUT-doExp0_2_mc16ade_MCStat_mBB_VR output/140ifb-2L-ade-STXS-METTrig-CUT-doExp0

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE variables to run on pre-fit Asimov (~L45)
>   >  doExp = "1" # "0" to run observed, "1" to run expected only

>    CHANGE variables to run on the batch as this will be hefty job (~L48)
>   >  run_on_batch = False                                                                                             

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)
>   >  createSimpleWorkspace = True                                                                                
>   >  runP0 = True                                                                                                          
>   >  runPulls = True                                                                                                       
>   >  runBreakdown = True                                                                                              
>   >  runRanks = True                                                                                                
>   >  runLimits = False                                                                                                      
>   >  runToyStudy = False                                                                                                 

>    ADD additional debug plots for shape plots (~L60)                                                                
>   >  doplots = True                                                                                                     
~~~

cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-METTrig-CUT-doExp1
mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_METTrigger.140ifb-2L-ade-STXS-METTrig-CUT-doExp1_fullRes_boostedVHbb_140ifb-2L-ade-STXS-METTrig-CUT-doExp1_2_mc16ade_MCStat_mBB_VR output/140ifb-2L-ade-STXS-METTrig-CUT-doExp1
~~~
In addition to this one can run a systematic fit on the set of inputs generated by my framwork locally, so the ones with both the addition from the Z+jets extenstions and the MET Trigger Study in 2L and compare the post fit plots to the ones in the note.
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE the fit type to running on systematics                                                                           
>   >  syst_type = ["Systs"]   (~L41)      #["StatOnly"] not working                                                      
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-METTrig-CUT-doExp1-Systs
mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_METTrigger.140ifb-2L-ade-STXS-METTrig-CUT-doExp1-Systs_fullRes_boostedVHbb_140ifb-2L-ade-STXS-METTrig-CUT-doExp1-Systs_2_mc16ade_Systs_mBB_VR output/140ifb-2L-ade-STXS-METTrig-CUT-doExp1-Systs
~~~
For the third set of inputs you do as before but changing the version name
~~~
vim scripts/launch_default_jobs.py 
~~~
>    CHANGE Global run conditions (~L11)                                                                                     
>   >  version = "v05_Extensions"                                                                                             
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-Xten-CUT-doExp1
mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_Extensions.140ifb-2L-ade-STXS-Xten-CUT-doExp1_fullRes_boostedVHbb_140ifb-2L-ade-STXS-Xten-CUT-doExp1_2_mc16ade_MCStat_mBB_VR output/140ifb-2L-ade-STXS-Xten-CUT-doExp1

vim scripts/launch_default_jobs.py 
~~~
>    CHANGE variables to run on post-fit Asimov (~L45)
>   >  doExp = "0" # "0" to run observed, "1" to run expected only

>    CHANGE what you want to run in the 0L standalone fit (~L62-L69)
>   >  createSimpleWorkspace = True                                                                                
>   >  runP0 = True                                                                                                      
>   >  runPulls = True                                                                                                      
>   >  runBreakdown = False                                                                                              
>   >  runRanks = False                                                                                             
>   >  runLimits = False                                                                                                                                                                                                  
>    ADD additional debug plots for shape plots (~L60)                                                                
>   >  doplots = False                                                                                                    
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_boostedVHbb
source setup.sh
cd build
cmake ..
make -j8
cd ..
python scripts/launch_default_jobs.py 140ifb-2L-ade-STXS-Xten-CUT-doExp0
mv output/SMVHVZ_BoostedVHbb2019_CUT_mc16ade_v05_Extensions.140ifb-2L-ade-STXS-Xten-CUT-doExp0_fullRes_boostedVHbb_140ifb-2L-ade-STXS-Xten-CUT-doExp0_2_mc16ade_MCStat_mBB_VR output/140ifb-2L-ade-STXS-Xten-CUT-doExp0

~~~
The flagship test is the significance. To obtain the significances of the different fits you can print the last 10 lines of the log to the screen.
~~~
cd output
tail -n 10 140ifb-2L-ade-STXS-baseline-CUT-doExp1/logs/output_getSig_125.log
tail -n 10 140ifb-2L-ade-STXS-Xten-CUT-doExp1/logs/output_getSig_125.log
tail -n 10 140ifb-2L-ade-STXS-METTrig-CUT-doExp1/logs/output_getSig_125.log
~~~
