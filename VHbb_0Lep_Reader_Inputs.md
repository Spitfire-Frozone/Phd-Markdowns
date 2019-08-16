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
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/
~~~
## Setup:
- CxAOD production r32-15
- latest CxAODReader, CxAODReader_VHbb, CxAODTools, CorrsAndSysts, CxAODOperations_VHbb
~~~
vim source/CxAODOperations_VHbb/scripts/submitReader.sh
~~~
- only fit inputs: DOONLYINPUTS="true" (~L194)
- all systematics: NOMINALONLY="false" (~L204)
- start with non-STXS, we will need STXS_stage=2 for the final configuration: GENERATESTXSSIGNALS="false" (~L211)
>  rerunning only the signal in STXS mode after the rest of the inputs are fully done
- harmonized CBA/MVA: DONEWREGIONS="true" (L274)
>   should  consider a "false" production, but I propose to launch it only once this one is done
- Latest custom CDI: BTAGGINGCDIFILE="/afs/cern.ch/user/i/iluise/public/2019-13TeV_Feb19_CDI-2019-02-18_v1_Continuous.root" (~L560)
- b-tagging: BTAGGINGCONFIGS="MV2c10 70 AntiKt4EMTopoJets FixedCut" (L571)
- split histograms at ptv=250: DOPTVSPLITTING250GEV="true" (L661)
- truth-tagging: use full hybrid, mode H
- periods: files for a, d, e, ad, ade
- include medium pTV (75-150 GeV)

## Running 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/
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

## Outputs:
The outputs from the running should be stored here:
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/XLep/r32-15_customCDI_DATE/
with XLep the folder of your channel. Thomas Calvet thinks it is clearer if we create a new folder with the date than piling up files with just a version number.

Ouputs should have the following naming convention:
- LimitHistograms.VHbb.*x*Lep.13TeV.mc16*y*.*z*.root
>  x=0,1,2 for your channel                                                                                    
>  y=a,d,e,ad,ade depending on the period                                                                      
>  z="yourInstitute.version" (version=r32-15+addOnIfNeeded)   

- A README file containing all setup information for your channel (for example detail which samples are truthtagged)
- Ideally the configuration file (framework-read or logfile) of the CxAOD reader in the folder
