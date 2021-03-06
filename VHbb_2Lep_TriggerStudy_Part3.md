# This is aimed at providing those who read it with an note of what I did when asked to repeat my MET study on a different set of samples designed for run 3. #

## VHbb 2 Lepton Trigger Study Part 3 ##

Last Edited: 26-05-2020
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
source ./copydatafromafs.sh
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
cd /afs/cern.ch/work/d/dspiteri/VHbb/
vim CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
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

>  COMMENT OUT samples from SAMPLES2 to ensure faster running (~L348-L351)                                                   
>   >   #SAMPLESCOMMON2+=" WqqWlv_Sh221 " # WW (default)                                                                      
>   >   #SAMPLESCOMMON2+=" ggWqqWlv_Sh222" ## ggWW                                                                            
>   >   #SAMPLESCOMMON2+=" WenuC_Sh221 WenuL_Sh221 WmunuC_Sh221 WmunuL_Sh221 WtaunuC_Sh221 WtaunuL_Sh221" # W+jets             
>   >   #SAMPLESCOMMON2+=" ZeeC_Sh221 ZeeL_Sh221 ZmumuC_Sh221 ZmumuL_Sh221 ZtautauC_Sh221 ZtautauL_Sh221" # Z+jets except Znunu+jets
 

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
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 SignalResolved_METTrigger 2L e VHbb MVA H 32-15 qqZllHbbJ_PwPy8MINLO none 1
~~~
_____
### Troubleshooting
>   - If running either command you get the error                                                                               
>   >   hsg5frameworkReadCxAOD: /cvmfs/sft.cern.ch/lcg/releases/gcc/6.2.0-b9934/x86_64-centos7/lib64/libstdc++.so.6: version `CXXABI_1.3.11' not found (required by ... )                                                                                  

>   Then most likely GCC 6.2 has introduced a newer C++ ABI version than your system libstdc++ has, so you need to tell the library loader where the newer version of the library is by adding that path to LD_LIBRARY_PATH. In this case, check firstly that you have set up the wrong version of AnalysisBase or you have forgotten to source your current setup. The mostly cause is that the line "lsetup 'lcgenv -p LCG_96b x86_64-centos7-gcc8-opt numpy' " has depreciated. All of these types of commands should be run with their latest versions.

>  - If you are unsure whether your output files are correct the first thing you should note is that output files of the megabyte-gigabyte size are largely correct, but ones of the kilobyte size usually have failed in some large capacity. 

>  - If instead you get the error 
>   >   /afs/cern.ch/work/d/dspiteri/VHbb/build/x86_64-centos7-gcc8-opt/data/CxAODReader_VHbb/2017-21-13TeV-MC16-CDI-2019-07-30_v1_CustomMaps.root does not exist, as a first port of call find someone with these files, put them in the right place in the directory and try again. Most of this time this is because the command `source ./copydatafromafs.sh' was not run and the files that are stored on afs rather than the repo have not been incorporated into the framework.

>   - If the run is taking a long time, it may be that there are other samlpes being run alongside the one you want. In this case try setting both SAMPLESFINAL2 to something such that it doesn't pick up samples we are not interested in (for me this was the case for the resolved analysis and not the boosted one). 
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

>  COMMENT OUT SAMPLES2 samples such that they are not run over (~L348-351)                                                   
>   >   #SAMPLESCOMMON2+=" WqqWlv_Sh221 " # WW (default)
>   >   #SAMPLESCOMMON2+=" ggWqqWlv_Sh222" ## ggWW
>   >   #SAMPLESCOMMON2+=" WenuC_Sh221 WenuL_Sh221 WmunuC_Sh221 WmunuL_Sh221 WtaunuC_Sh221 WtaunuL_Sh221" # W+jets
>   >   #SAMPLESCOMMON2+=" ZeeC_Sh221 ZeeL_Sh221 ZmumuC_Sh221 ZmumuL_Sh221 ZtautauC_Sh221 ZtautauL_Sh221" # Z+jets except Znunu+jets

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
After running these four, its best to check that none of the jobs failed. This is done by using the script checkReaderFails.py 
~~~
cd SignalBoosted_METTrigger       
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D1

cd ../SignalResolved_METTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_MVA_H

cd ../SignalBoosted_SLTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_CUT_D1

cd ../SignalResolved_SLTrigger
python /afs/cern.ch/work/d/dspiteri/VHbb/CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts/checkReaderFails.py Reader_2L_32-15_e_MVA_H
~~~
If there are failed jobs you can submit them by searching the failed jobs in the segments directory and submitting the job according to the numerical job number identified as failed
~~~
vim Reader_2L_32-15_e_CUT_D1/submit/segments
./submit/run 24
~~~

## (3) New Samples MET Trigger Regime and (4) New Samples SL Trigger Regime.
Also an easy one to do, as the commands will be exactly the same as for the previous two sections, but you point to a different location of the samples.
**Make sure that the submitReader is submitting the right analysis and that the trigger switch has been flipped**. 

>  /afs/cern.ch/work/g/gcallea/public/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/qqZllHbbJ_PwPy8MINLO
Now create a new directory in the folder where the samples live and put the new samples there. Ensure that you make a unique name such that nobody else can run over it by accident.
~~~
cd /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/
mkdir qqZllHbbJ_PwPy8MINLOr3
cp /afs/cern.ch/work/g/gcallea/public/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15/HIGG2D4_13TeV/CxAOD_32-15_e/qqZllHbbJ_PwPy8MINLO/* qqZllHbbJ_PwPy8MINLOr3
~~~
If you cannot access this directory, or the files contained within, you might want to get Giuseppe to give you cernbox access, so you can download it and upload it to the server yourself, such you have the right permissions. This is only a temporary solution for small files. 

### New Samples Boosted MET Trigger
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
vim CxAODMakerCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
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
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 NewSignalBoosted_METTrigger 2L e VHbb CUT D1 32-15 qqZllHbbJ_PwPy8MINLOr3 none 1
~~~

### New Samples Boosted SL Trigger
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
vim CxAODMakerCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
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
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 NewSignalBoosted_SLTrigger 2L e VHbb CUT D1 32-15 qqZllHbbJ_PwPy8MINLOr3 none 1
~~~

### New Samples Resolved MET Trigger
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
vim CxAODMakerCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
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
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 NewSignalResolved_METTrigger 2L e VHbb MVA H 32-15 qqZllHbbJ_PwPy8MINLOr3 none 1
~~~

### New Samples Resolved SL Trigger
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
vim CxAODMakerCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh
~~~
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
../CxAODReaderCore_May2020/VHbb/CxAODOperations_VHbb/scripts_CxAODReader/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r32-15 NewSignalResolved_SLTrigger 2L e VHbb MVA H 32-15 qqZllHbbJ_PwPy8MINLOr3 none 1
~~~
# Making Plots
For the original study I made a macro called TriggerStudyPlots.cxx which I an make available upon request. However it really works under the old version of the analysis. I have made a newer version of this plotting script called TriggerStudyPlotting.cxx designed to exist in your home area in parallel to the analysis repo, the build and run directories.

The inputs to the file are as follows                                                                                       
| void TriggerStudyPlots(  |            |                                                                                     
|:--------------------:|------------------------|
| TString homeArea       = "/afs/cern.ch/work/d/dspiteri/VHbb/", |   Your home area                                         | 
| TString sampletype     = "Signal",                             |   First part of the output folder. i.e 'NewSignalBoosted'|
| TString triggertype1   = "old",                                |   Type of triggers (new, newer, newest, old, SL, MET)    |
| TString triggertype2   = "new"                                 |                                                          |
| TString inputFileName  = "SIGNAL.root",                        |   Input file name                                        | 
| TString channel        = "2L",                                 |   Analysis Lepton Channel interested in                  |
| TString CxAODSampVer   = "31-10",                              |   Sample Version                                         | 
| TString mcVer          = "a",                                  |   MC Simulation Version                                  |
| TString modelType      = "MVA"                                 |   MVA = Resolved, CBA = Boosted                          |
| TString bTagType       = "D1",                                 |   B-tagging Type in output folder                        | 
| TString region         = "SR",                                 |   Signal or Control Region                               |
| TString suffix1        = "",                                   |   Suffixes you added to seperate samples                 |
| TString suffix2        = "" )                                  |                                                          |
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup "root 6.18.04-x86_64-centos7-gcc8-opt"
vim TriggerStudyPlotting.cxx 
~~~
> CHANGE vector of samples to the signal ones (L111)
>   >     std::vector samples = {"qqZllH125"}; //Giuseppe's Trial Samples

> Ensure that the legend displays the right type of label (L528)
>   >      legend.AddEntry(TProfileInputPlot2, Form("Signal (%) Gain = %f",improvement), "-"); 

~~~
cd run
root -b -l -q '../TriggerStudyPlotting.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/","SignalBoosted","SL","MET","hist-qqZllHbbJ_PwPy8MINLO.root","2L","32-15","e","CUT","D1","SR")'
mv SignalBoosted-SLandMET_TriggerPlots SignalBoosted-SLandMET_e_TriggerPlots
cd SignalBoosted-SLandMET_e_TriggerPlots
imgcat *.pdf
~~~
Once this works, and everything looks ok, we can produce the other plots. 
~~~
cd ..
root -b -l -q '../TriggerStudyPlotting.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/","NewSignalBoosted","SL","MET","hist-qqZllHbbJ_PwPy8MINLOr3.root","2L","32-15","e","CUT","D1","SR")'
mv NewSignalBoosted-SLandMET_TriggerPlots NewSignalBoosted-SLandMET_e_TriggerPlots

root -b -l -q '../TriggerStudyPlotting.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/","SignalResolved","SL","MET","hist-qqZllHbbJ_PwPy8MINLO.root","2L","32-15","e","MVA","H","SR")'
mv SignalResolved-SLandMET_TriggerPlots SignalResolved-SLandMET_e_TriggerPlots

root -b -l -q '../TriggerStudyPlotting.cxx("/afs/cern.ch/work/d/dspiteri/VHbb/","NewSignalResolved","SL","MET","hist-qqZllHbbJ_PwPy8MINLOr3.root","2L","32-15","e","MVA","H","SR")'
mv NewSignalResolved-SLandMET_TriggerPlots NewSignalResolved-SLandMET_e_TriggerPlots
~~~
