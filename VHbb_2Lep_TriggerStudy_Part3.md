# This is aimed at providing those who read it with an note of what I did when asked to repeat my MET study on a different set of samples designed for run 3. #

## VHbb 2 Lepton Trigger Study Part 3 ##

Last Edited: 11-05-2020
-------------------------------------------------------------------------------

# Aims
The aims of exercise this time is to create perform the MET Trigger Study in both the 32-15 smaples used in the latest paper and the new run3 samples with different MET trigger configurations. On the analysis side of things the code used to run over them should be identical, but the set of samples to be run is different, and a different data period has been simulated. 

# Setting up the repository
## NOW FEATURING SUBMODULES!
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
git clone --recursive ssh://git@gitlab.cern.ch:7999/CxAODFramework/CxAODMakerCore.git

mv CxAODMakerCore CxAODMakerCore_May2020
cd CxAODMakerCore_May2020
~~~
In this directory there are two folders. Core, which contains the CxAODMaker and CxAODTools packages, and VHbb, which contains the analysis specific packages: CxAODBootstrap_VHbb, CxAODMaker_VHbb, CxAODOperations_VHbb and CxAODTools_VHbb.

# Samples
The new samples based off of MC16e samples and are currently based here: 
> /afs/cern.ch/work/g/gcallea/public/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/

The 32-15 samples to compare this against are currently based here:
> /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/

# Running
To perform this study there need to be four sets of runs done
1) Old Samples - Current Trigger Regime (MET triggers)
2) Old Samples - Previous Trigger Regime (Single Lepton triggers)
3) New Samples - Current Trigger Regime (MET triggers)
4) New Samples - Previous Trigger Regime (Single Lepton triggers)

## (1) Old Samples Current Trigger Regime.
Currently the easiest one to do as it requires the least amount of changes to the code, if any at all
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODMakerCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh


~~~

## (3) New Samples Current Trigger Regime.
The next easiest one to do, only requires a couple of line changes and those are to do with the location of the samples.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODMakerCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh


~~~
