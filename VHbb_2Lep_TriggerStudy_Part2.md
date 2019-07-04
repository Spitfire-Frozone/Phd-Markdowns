# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about the testing new trigger regimes for the 2 Lepton Analysis for VHbb. #

## VHbb 2 Lepton Trigger Study Part 2 ##

Last Edited: 02-07-2019
-------------------------------------------------------------------------------

# Setup Script
Search https://gitlab.cern.ch/CxAODFramework/FrameworkSub to see what the latest released version is.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
source getMaster.sh origin/master CxAODFramework_master 1 1
> source getMaster.sh r31-10 CxAODFramework_tag_r31-10 1 1 [for a tag]
~~~

# Editing MET Trigger code to also work with 1L
There are 6 steps
- Update Variables in submitReader
- Proliferate new Variables to config files
- Move TriggerTool_VHbb2Lep code to TriggerTool_VHbb
- Change function calls in AnalysisReader_VHQQ2Lep.cxx and 1L equivalent
-  
~~~
vim 
~~~



When you think you have finished then you can refresh your local changes and re-build. Make sure that you go into each directory in /source/ and do git pull to make sure that each sub-repository is up to date. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_master/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt" 
release=`cat source/CxAODBootstrap_VHbb/bootstrap/release.txt` && echo "release=$release"
asetup $release,AnalysisBase
cd build && rm -rf *
cmake ../source
make -j10
source x86_64-centos7-gcc62-opt/setup.sh
lsetup 'lcgenv -p LCG_91 x86_64-centos7-gcc62-opt numpy'
~~~
