# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat Part 2 ##

## Last Edited: 30-08-2019
--------------------------------------------------------------------------

Once a basic familiarity with WSMaker has been established, and a newer set of inputs has been created. These will need to be tested.  

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
(mv WSMaker_VHbb WSMaker_VHbb_WZjets) #Temprorarily move this such that the git clone works.
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git


mv WSMaker_VHbb WSMaker_VHbb_New0Linputs (&& mv WSMaker_VHbb_WZjets WSMaker_VHbb)
cd WSMaker_VHbb_New0Linputs
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..
~~~

Now we have to switch to a specific branch
~~~
git branch --all | grep kalkhour
git checkout -b master-kalkhour-Adapt_WS-to_NewRegions origin/master-kalkhour-Adapt_WS-to_NewRegions --track
~~~

The next step is to split the new inputs, normally you would do this yourself but if they have already been split then you can just copy forward.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/WSMaker_VHbb_New0Linputs
setupATLAS && lsetup git 
cd inputs
cp -r /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/AfterWSMakerSplit/2019-08-20/ZeroLepton-mc16ad .
~~~
