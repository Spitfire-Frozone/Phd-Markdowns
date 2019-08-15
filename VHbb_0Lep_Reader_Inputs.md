# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about generating and creating ntuples from VHbb CxAOD's and testing them. #

## "VHbb 0 Lepton Reader Inputs" ##

Last Edited: 15-08-2019
-------------------------------------------------------------------------------

# Setup

The first thing that you want to do is ensure that you are up to date with the framework. To do this you should have access to the latest branch/tag of the CxAOD Framework. It's good to check online
(https://gitlab.cern.ch/CxAODFramework/FrameworkSub/tree/master/bootstrap) but as it's said in VHbb_2Lep_Reader_Inputs.md there is a process of continual updates.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS &&lsetup git
cp /afs/cern.ch/user/v/vhbbframework/public/CxAODBootstrap_VHbb/scripts/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_master_0Linputs2019 1 1
~~~

## Reconfiguration of necessary files.

Setup:
- CxAOD production r32-15
- latest CxAODReader, CxAODReader_VHbb, CxAODTools, CorrsAndSysts, CxAODOperations_VHbb
- all systematics: NOMINALONLY="false"
- only fit inputs: DOONLYINPUTS="true"
- start with non-STXS, we will need STXS_stage=2 for the final configuration 
      -> rerunning only the signal in STXS mode after the rest of the inputs are fully done
- b-tagging: make sure you use
      -> BTAGGINGCONFIGS="MV2c10 70 AntiKt4EMTopoJets FixedCut"
      -> Latest custom CDI: BTAGGINGCDIFILE="/afs/cern.ch/user/i/iluise/public/2019-13TeV_Feb19_CDI-2019-02-18_v1_Continuous.root"
- split histograms at ptv=250: DOPTVSPLITTING250GEV="true"
- truth-tagging: use full hybrid, mode H
- harmonized CBA/MVA: DONEWREGIONS="true"
      -> we should  consider a "false" production, but I propose to launch it only once this one is done
- periods: files for a, d, e, ad, ade
- include low pTV (75-150 GeV)
Feel free to remind me any point we discussed on the previous productions you might think useful.

Outputs:
- outputs should be stored here:
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/XLep/r32-15_customCDI_DATE/
with XLep the folder of your channel. I think it is clearer if we create a new folder with the date than piling up files with just a version number
- ouputs should be named:
- LimitHistograms.VHbb.*x*Lep.13TeV.mc16*y*.*z*.root
- with: x=0,1,2 for your channel
          y=a,d,e,ad,ade depending on the period
          z="yourInstitute.version" (version=r32-15+addOnIfNeeded)
- a README file containing all setup information for your channel (for example detail which samples are truthtagged)
- ideally you can put the configuration file (framework-read or logfile) of the CxAOD reader in the folder
