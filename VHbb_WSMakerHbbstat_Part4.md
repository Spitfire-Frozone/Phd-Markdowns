# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 28-11-2019
-------------------------------------------------------------------------------
## Milestone 2 Fits on latest inputs

Now we shall test the 0L standalone fit for new and shiny inputs. 
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-physics/higgs/hbb/WSMaker_VHbb.git

mv WSMaker_VHbb WSMaker_VHbb_Milestone2
cd WSMaker_VHbb_Milestone2
source setup.sh
mkdir inputs
cd build
cmake ..
make -j8
cd ..
~~~
The inputs can be found here.
>    0-lepton :/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root
>    1-lepton : /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/OneLep/r32-15_postMS2_20191128/LimitHistograms.VHbb.1Lep.13TeV.mc16ade.UCL.32-15.root
>    2-lepton : /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/TwoLep/r32-15_postMS2_20191124/v1/fullSysts/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.Kyoto.r32-15_postMS2_20191124.v1.root

The first thing to do is to check the yields against the previous inputs used. 
~~~
vim scripts/InputsCheck_VHbb.py
~~~
>    CHANGE Inputs to compare and the common area where they are kept (~L28-30)
>   >  eos_dir='/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/FullRunII2019/statArea/inputs/'                             
>   >  IC.input_ref  = eos_dir+'ZeroLep/r32-15_PCBT_20191113/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford-newExt-PCBT.32-15.root'
>   >  IC.input_test = eos_dir+'ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root'

>    CHANGE the Lepton channel to 0 lepton.
>   >  IC.lep = 'L0'
~~~
python scripts/InputsCheck_VHbb.py > Comp32-15_PCBT-20191113_postMS2_20191127.txt
tail -n 7 Comp32-15_PCBT-20191113_postMS2_20191127.txt
~~~
The next step is to split these new inputs
~~~
cd inputConfigs
vim SMVHVZ_2019_MVA_mc16ade_v03_STXS.txt 
~~~
>    ADD new samples
>   >  ZeroLepton ZeroLep/r32-15_postMS2_20191127/LimitHistograms.VHbb.0Lep.13TeV.mc16ade.Oxford.r32-15.root mva,MET,mBB,mvadiboson                                                                                 
>   >  OneLepton OneLep/r32-15_postMS2_20191128/LimitHistograms.VHbb.1Lep.13TeV.mc16ade.UCL.32-15.root mvaFullRun2Default,pTV,mBB,mvadiboson
>   >  TwoLepton TwoLep/r32-15_postMS2_20191124/v1/fullSysts/LimitHistograms.VHbb.2Lep.13TeV.mc16ade.Kyoto.r32-15_postMS2_20191124.v1.root rootmvaPolVars,pTV,mBB,mvadiboson
~~~
cd ..
SplitInputs -r Run2 -v SMVHVZ_2019_MVA_mc16ade_v03_STXS
~~~
