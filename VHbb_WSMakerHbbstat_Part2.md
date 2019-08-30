# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## "VHbb WSMaker and Hbb Stat Part 2" ##
===============================================================================
## Last Edited: 30-08-2019

Once a basic familiarity with WSMaker has been established, and a newer set of inputs has been created. These will need to be tested.  
~~~
1) MC16ad new + new regions Global Conditional
2) MC16ad new + new regions Asimov
3) MC16ad new + new regions Asimov fitting only CRs
4) MC16ad new + new regions Global Conditional fitting only CRs

- Pull plots 1 vs 2
- Pull plots 1 vs MC16ad old
- Pull plots 1 vs 3
- Pull Plots 1+2 vs 4

The first thing though, is to check out a new version of the WSMaker. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git 
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git
mv WSMaker_VHbb WSMaker_VHbb_New0Linputs && cd WSMaker_VHbb_New0Linput
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
~~~
The next step is to split the new inputs 
~~~
vim setup.sh
~~~
>  CHANGE blinding to be on. 
>   >     IS_BLINDED = 1 (L6)
~~~
source setup.sh
cd build && cmake ..
make -j10
cd ..
SplitInputs -r Run2 -v SMVHVZ_Summer18_MVA_mc16ad_v07_fixed2
~~~
One the inputs have been split, open the job launcher and ensure that  that these settings are there
~~~
vim scripts/launch_default_jobs.py
~~~
