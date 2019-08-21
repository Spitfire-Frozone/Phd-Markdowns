# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about generating and creating ntuples from VHbb CxAOD's and testing them. #

## "VHbb 0 Lepton Reader Inputs" ##

Last Edited: 19-08-2019
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
source x86_64-centos7-gcc8-opt/setup.sh
cd ../run
~~~
Ouputs should have the following naming convention:
- LimitHistograms.VHbb.*x*Lep.13TeV.mc16*y*.*z*.root
>  x=0,1,2 for your channel                                                                                    
>  y=a,d,e,ad,ade depending on the period                                                                      
>  z="yourInstitute.version" (version=r32-15+addOnIfNeeded)   
~~~
../source/CxAODOperations_VHbb/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow 0L e VHbb MVA H 32-15 none none 1

~~~
## Outputs:

The inputs need to be checked for errors that occurred during the run. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 

cd run/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow.root/Reader_0L_32-15_e_MVA_H/
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/source/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_0L_32-15_e_MVA_H/
~~~
Once you are happy that all the files are all present and correct. You can hadd the output.
~~~
cd run/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow/Reader_0L_32-15_e_MVA_H/fetch

hadd DATA.root hist-data*
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

hadd LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow.root GGZH.root GGZZ.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root  TTBAR_1PLUSLEP.root  WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root  ZEEB.root  ZEE.root  ZMUMUB.root ZMUMU.root ZTAUTAUB.root ZTAUTAU.root  

rm DATA.root GGZH.root GGZZ.root QQWH.root QQZH.root STOPs.root TTBAR_DILEP.root TTBAR_1PLUSLEP.root WENUB.root WENU.root WMUNUB.root WMUNU.root WTAUNUB.root WTAUNU.root WZ.root ZZ.root ZEEB.root ZEE.root ZMUMUB.root ZMUMU.root ZTAUTAUB.root ZTAUTAU.root 
~~~
The outputs from the running should be stored here:
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/XLep/r32-15_customCDI_DATE/
>  /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_customCDI_20190820/

- A README file containing all setup information for your channel (for example detail which samples are truthtagged)
- Ideally the configuration file (framework-read or logfile) of the CxAOD reader in the folder
~~~
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_customCDI_20190820/
cp /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/run/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow/Reader_0L_32-15_e_MVA_H/fetch/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow.root .
cp /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/run/LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow/Reader_0L_32-15_e_MVA_H/submit/run.log .
mv run.log run_e.log
cp /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master_0Linputs2019/source/CxAODReader_VHbb/data/framework-read-automatic.cfg .
mv framework-read-automatic.cfg framework-read-e-automatic.cfg

vim README.v01.r32-15_customCDI.txt
~~~
> 
> LimitHistograms.VHbb.0Lep.13TeV.mc16e.Glasgow.root
> 
> -2019/08/20
> 
> * 32-15 CxAOD files
> * Used August 15th 2019 checkout of CxAODReader master
> * Catagories split at ptV=250GeV
> * CDI file = /afs/cern.ch/user/i/iluise/public/2019-13TeV_Feb19_CDI-2019-02-18_v1_Continuous.root
> * Ran with the 'H' tag. All samples hybrid tagged
> * STXS signals: none 
> * New Regions = none
