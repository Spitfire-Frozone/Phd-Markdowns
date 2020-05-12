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
