# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document concerns the addition of plots into the group internal note. # 

## Adding Plots to the Internal Note ##

## Last Edited: 25-09-2019

-------------------------------------------------------------------------------
There will come a time where you will have produced plots and want to put them in a paper. The best way to make that paper
is in a Late$\chi$ environment, but if several people will want to add and edit things then the best thing to do is to create
a git repo such that changes and edits can be stored. You download the repo, add your stuff, build to make sure your changes don't break things and upload. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb
setupATLAS && lsetup git

git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics-office/HIGG/ANA-HIGG-2018-51/ANA-HIGG-2018-51-INT1.git
cd ANA-HIGG-2018-51-INT1

~~~
