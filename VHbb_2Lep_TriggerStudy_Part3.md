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
Now we need to check that this repository can build.
~~~
mkdir build && cd build
setupATLAS
asetup AnalysisBase,21.2.XXX,here
cmake ../CxAODMakerCore_May2020
cmake --build .
source x*setup.sh
~~~
# Samples
The new samples based off of MC16e samples and are currently based here: 
> /afs/cern.ch/work/g/gcallea/public/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/

Currently the only sample that is present is: qqZllHbbJ_PwPy8MINLO

The 32-15 samples to compare this against are currently based here:
> /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/

# Running
To perform this study there need to be two set of four sets of runs done
The first set is for the Boosted Analysis
1) Old Samples - Current Trigger Regime (MET triggers)
2) Old Samples - Previous Trigger Regime (Single Lepton triggers)
3) New Samples - Current Trigger Regime (MET triggers)
4) New Samples - Previous Trigger Regime (Single Lepton triggers)

The second set is the same as the above but for the Resolved Analysis

## (1) Old Samples Current Trigger Regime.
Currently the easiest one to do as it requires the least amount of changes to the code, if any at all
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODMakerCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
>  CHANGE Analysis Strategy (~L109)
>   >   ANASTRATEGY="Resolved" # Resolved, Merged, ...

>  CHECK the right combinations of trigger option and analysis (~L302-L307)
>   >   if [[ ${ANALYSIS} == "VHbb" ]] && [[ ${ANASTRATEGY} == "Merged" ]]; then #boosted VHbb
>   >       DO2LMETTRIGGER="true" # This replaces the Muon trigger with the MET one at a given PtZ value for the 2L analysis
>   >   else #Allow this flag to be turned off for the Resolved VHbb and VHcc analyses.
>   >       DO2LMETTRIGGER="false"
>   >   fi

~~~
## (3) New Samples Current Trigger Regime.
The next easiest one to do, only requires a couple of line changes and those are to do with the location of the samples.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODMakerCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh


~~~
