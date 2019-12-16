#  VHbb Tutorial Notes - November 13th 2018
## By Adrian Buzatu

General Page
https://gitlab.cern.ch/CxAODFramework/CxAODBootstrap_VHbb
General Tutorial Page
https://gitlab.cern.ch/CxAODFramework/CxAODOperations_VHbb (/blob/master/README_Tutorial.md)


To get git to work you need to create a new ssh key for git for your local machine.

https://gitlab.cern.ch/profile/keys
https://gitlab.cern.ch/help/ssh/README#generating-a-new-ssh-key-pair
~~~
ssh-keygen -t rsa -C "d.spiteri.1@research.gla.ac.uk" -b 4096
~~~

Once you have completed the instructions on there then we can try to run on the home machine.
~~~
cd /home/ppe/d/dspiteri/ATLAS/VHbb
setupATLAS && lsetup git
source /afs/cern.ch/user/a/abuzatu/work/public/BuzatuAll/BuzatuATLAS/CxAODFramework/getMaster.sh origin/master          CxAODFramework_branch_master_21.2.39_1    1           1
~~~

> VHbb -> ICHEP

> VHbb0925 -> Latest


## Testing
~~~
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh
~~~

check logs
cd logs

Check if you are happy and then remove all of these files and restore your configuration settings
~~~
./clean.sh
source ../source/CxAODOperations_VHbb/scripts/submitMaker.sh
~~~
inputs

mcdata-test-none (sample list, none is a single local file)
tags
Higgs-Exotics-User
0 - run with no tcc jets
-1 running over all events
none - (path to file, if none build from first argument)
none - log file (grid, no log file - locally, everything)
last number 0 (builds config without executing)

Try a local test
~~~
source ../source/CxAODOperations_VHbb/scripts/submitMaker.sh 0L       d       VHbb0925 none               none          user    0      200     /afs/cern.ch/work/v/vhbbframework/public/data/DxAOD/fromGoetz/180925/mc16_13TeV.345056.PowhegPythia8EvtGen_NNPDF3_AZNLO_ZH125J_MINLO_vvbb_VpT.deriv.DAOD_HIGG5D1.e5706_e5984_s3126_r10201_r10210_p3641_tid15477888_00 none 1
~~~

data-CxAOD: The CxOAD produced by this job
hist: cutflow
output-CxAOD: useless trash

Testing ROOT output
CollectionTree->Print("AntiKt*")
CollectionTree->Draw("AntiKt10LCTopoTrimmedPtFrac5SmallR20ExCoM2SubJets___NominalAuxDyn.m")

## Grid Submission
~~~
cd /home/ppe/d/dspiteri/ATLAS/VHbb
setupATLAS && lsetup rucio
voms-proxy-init -voms atlas

cd /home/ppe/d/dspiteri/ATLAS/VHbb/CxAODFramework_branch_master_21.2.39_1/run
source ../source/CxAODOperations_VHbb/scripts/submitMaker.sh 2L   d  VHbb0925  test  32-181113-dspiteri  user   0  -1   none  none  1
~~~

>  hsg5framework: error while loading shared libraries: libfastjet.so.0: cannot open shared object file: No such file or directory

>  ATLASPhysics->HiggsWorkingGroup->Higgsbb->HSG5RunIIAnalysisFramework->CxAODFramework->Release21Migration

## RunMaker

Setting up the download
~~~
C=/home/ppe/d/dspiteri/ATLAS/VHbb/CxAODFramework_branch_master_21.2.39_1
echo $C
source $C/source/CxAODOperations_VHbb/scripts/getMaker.sh 0L,1L,2L d VHbb0925 test Higgs 32-01 $C 1
~~~
Download.
~~~
cd prepare_2L_d_32-01
python2 rucio_get_jobs.py out_sample_list_sample_grid.13TeV_25ns.test_d.HIGG2D4.txt
~~~
