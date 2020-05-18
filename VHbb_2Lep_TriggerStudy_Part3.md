# This is aimed at providing those who read it with an note of what I did when asked to repeat my MET study on a different set of samples designed for run 3. #

## VHbb 2 Lepton Trigger Study Part 3 ##

Last Edited: 15-05-2020
-------------------------------------------------------------------------------

# Aims
The aims of exercise this time is to create perform the MET Trigger Study in both the 32-15 smaples used in the latest paper and the new run3 samples with different MET trigger configurations. On the analysis side of things the code used to run over them should be identical, but the set of samples to be run is different, and a different data period has been simulated. 

# Setting up the repository
## NOW FEATURING SUBMODULES!
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
git clone --recursive ssh://git@gitlab.cern.ch:7999/CxAODFramework/CxAODReaderCore.git

mv CxAODReaderCore CxAODReaderCore_May2020
cd CxAODReaderCore_May2020
git submodule update --init --recursive
cd ..
~~~
In this directory there are two folders. Core, which contains the CxAODOperation, CxAODReader and CxAODTools packages, and VHbb, which contains the analysis specific packages: CorrsAndSysts, CxAODBootstrap_VHbb, CxAODOperations_VHbb, CxAODReader_VHbb, CxAODTools_VHbb and KinematicFit.
Now we need to check that this repository can build. Standard practise is to make a build directory outside your repository
~~~
mkdir build run && cd build
setupATLAS
asetup AnalysisBase,21.2.110,here
cmake ../CxAODReaderCore_May2020
cmake --build .
source x86_64-centos7-gcc8-opt/setup.sh
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

The second set is the same as the above but for the Resolved Analysis. Since there are no new systematics that are introduced, this set of runs can be done under the nominal only. 

## (1) Old Samples MET Trigger Regime.
Currently the easiest one to do as it requires the least amount of changes to the code, if any at all. For the current Boosted Analysis this is the default.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
>  CHANGE Analysis Strategy (~L109)                                                                                           
>   >   ANASTRATEGY="Merged" # Resolved, Merged, ...                                                                           

>  CHANGE Driver because the files are very small (~L133)                                                                     
>   >   DRIVER="direct"                                                                                                       

>  CHECK that the systematic variations are not being run (~L217, ~L236)                                                       
>   >   NOMINALONLY = "true" flag                                                                                            

>  CHECK the right combinations of trigger option and analysis (~L302-L307)                                                   
>   >   if [[ ${ANALYSIS} == "VHbb" ]] && [[ ${ANASTRATEGY} == "Merged" ]]; then #boosted VHbb                                 
>   >       DO2LMETTRIGGER="true" # This replaces the Muon trigger with the MET one at a given PtZ value for the 2L analysis   
>   >   else #Allow this flag to be turned off for the Resolved VHbb and VHcc analyses.                                       
>   >       DO2LMETTRIGGER="false"                                                                                             
>   >   fi                                                                                                                     

For the Boosted (Merged) case:
~~~
setupATLAS && lsetup "root 6.18.04-x86_64-centos7-gcc8-opt"
cd build
release=`cat ../CxAODReaderCore_May2020/VHbb/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cmake ../CxAODReaderCore_May2020
cmake --build .
source x86_64-centos7-gcc8-opt/setup.sh
lsetup 'lcgenv -p LCG_96b x86_64-centos7-gcc8-opt numpy'
cd ../run
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_METTrigger 2L e VHbb CUT D1 32-15 qqZllHbbJ_PwPy8MINLO none 1
~~~
For the Resolved case:
~~~
setupATLAS && lsetup "root 6.18.04-x86_64-centos7-gcc8-opt"
cd build
release=`cat ../CxAODReaderCore_May2020/VHbb/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cmake ../CxAODReaderCore_May2020
cmake --build .
source x86_64-centos7-gcc8-opt/setup.sh
lsetup 'lcgenv -p LCG_96b x86_64-centos7-gcc8-opt numpy'
cd ../run
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalResolved_METTrigger 2L e VHbb MVA D1 32-15 qqZllHbbJ_PwPy8MINLO none 1
~~~
_____
### Troubleshooting
>   If running either command you get the error                                                                               
>   >   hsg5frameworkReadCxAOD: /cvmfs/sft.cern.ch/lcg/releases/gcc/6.2.0-b9934/x86_64-centos7/lib64/libstdc++.so.6: version `CXXABI_1.3.11' not found (required by ... )                                                                                  

>   Then most likely GCC 6.2 has introduced a newer C++ ABI version than your system libstdc++ has, so you need to tell the library loader where the newer version of the library is by adding that path to LD_LIBRARY_PATH. In this case, check firstly that you have set up the wrong version of AnalysisBase or you have forgotten to source your current setup. The mostly cause is that the line "lsetup 'lcgenv -p LCG_96b x86_64-centos7-gcc8-opt numpy' " has depreciated. All of these types of commands should be run with their latest versions.

>  If you are unsure whether your output files are correct the first thing you should note is that output files of the megabyte-gigabyte size are largely correct, but ones of the kilobyte size usually have failed in some large capacity. 

>  If instead you get the error /afs/cern.ch/work/d/dspiteri/VHbb/build/x86_64-centos7-gcc8-opt/data/CxAODReader_VHbb/2017-21-13TeV-MC16-CDI-2019-07-30_v1_CustomMaps.root does not exist, as a first port of call find someone with these files, put them in the right place in the directory and try again.   
____
Next one should check that all the inputs were fine. Resubmitting failed ones as necessary. From the run directory that you submitted the files in.
~~~
cd SignalBoosted_METTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D1
~~~
## (2) Old Samples Single Lepton Trigger Regime.
I implimented a switch, so it can always just be turned off again! For the Resolved Analysis this is the default. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
>  CHANGE Analysis Strategy (~L109)                                                                                           
>   >   ANASTRATEGY="Merged" # Resolved, Merged, ...                                                                           

>  CHANGE Driver because the files are very small (~L133)                                                                     
>   >   DRIVER="direct"                                                                                                       

>  CHECK the right combinations of trigger option and analysis (~L302-L307)                                                   
>   >   if [[ ${ANALYSIS} == "VHbb" ]] && [[ ${ANASTRATEGY} == "Merged" ]]; then #boosted VHbb                                 
>   >       DO2LMETTRIGGER="true" # This replaces the Muon trigger with the MET one at a given PtZ value for the 2L analysis   
>   >   else #Allow this flag to be turned off for the Resolved VHbb and VHcc analyses.                                       
>   >       DO2LMETTRIGGER="false"                                                                                             
>   >   fi                                                                                                                     

For the Boosted (Merged) case:
~~~
setupATLAS && lsetup "root 6.18.04-x86_64-centos7-gcc8-opt"
cd build
release=`cat ../CxAODReaderCore_May2020/VHbb/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cmake ../CxAODReaderCore_May2020
cmake --build .
source x86_64-centos7-gcc8-opt/setup.sh
lsetup 'lcgenv -p LCG_96b x86_64-centos7-gcc8-opt numpy'
cd ../run
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalBoosted_SLTrigger 2L e VHbb CUT D1 32-15 qqZllHbbJ_PwPy8MINLO none 1
~~~
For the Resolved case:
~~~
setupATLAS && lsetup "root 6.18.04-x86_64-centos7-gcc8-opt"
cd build
release=`cat ../CxAODReaderCore_May2020/VHbb/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cmake ../CxAODReaderCore_May2020
cmake --build .
source x86_64-centos7-gcc8-opt/setup.sh
lsetup 'lcgenv -p LCG_96b x86_64-centos7-gcc8-opt numpy'
cd ../run
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalResolved_SLTrigger 2L e VHbb MVA H 32-15 qqZllHbbJ_PwPy8MINLO none 1
~~~

## (3) New Samples MET Trigger Regime.
The next easiest one to do, only requires a couple of line changes and those are to do with the location of the samples.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODMakerCore_May2020
vim VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh


~~~
