# This is aimed at providing those who read it with an note of what I did when asked to repeat my MET study on a different set of samples designed for run three. #

## VHbb 2 Lepton Trigger Study Part 3 ##

Last Edited: 11-05-2020
-------------------------------------------------------------------------------

# Setup Script
## NOW FEATURING SUBMODULES!
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
cp /afs/cern.ch/user/v/vhbbframework/public/CxAODBootstrap_VHbb/scripts/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_master_july2019 1 1
~~~
>   source getMaster.sh r31-10 CxAODFramework_tag_r31-10 1 1 [for a tag]
