# This will be aimed at my third task which was to port everything on the branch of 21.0 on git into svn. # 

## "R21" in Data and "R21 MC16c" ##

Last Edited: 09-10-2017
-------------------------------------------------------------------------------

The next part of the task is to compare the code on SVN to the most recent 
version of the git master branch (your changes should not be there yet)
Set up a new directory to compare ATHENA trunks. What you want to do is
git clone the latest git release, then use the svnpull.py script to pull the 
svn trunk, into this git clone, and then you can git diff -R the two
directories to see the differences between the two.

~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development
(rm -rf COMP_ATHENA)
mkdir ATHENA_4_20-20 && cd ATHENA_4_20-20
setupATLAS
lsetup git python # python is needed for svnpull.py
git clone https://:@gitlab.cern.ch:8443/dspiteri/athena.git
cd athena
git remote add upstream https://:@gitlab.cern.ch:8443/atlas/athena.git
git fetch upstream
git checkout -b 21.0_Update-Git upstream/21.0 --no-track
(git merge --ff-only upstream/master is the default) 

git log --oneline InnerDetector/InDetValidation/InDetPhysValMonitoring
~~~
Find the last commit that updated from svn ends in (InDetPhysValMonitoring-XX-XX-XX-XX), then create new branch on the node with the last commit and pull svn changes here.
~~~
git checkout -b 21.0_Last-Svn [commithash at start that accompanies the svn tag]
svnpull.py InDetPhysValMonitoring 
~~~
The root that this command pulls from defaults to svn+ssh://svn.cern.ch/reps/atlasoff so only the package is needed.
You only need the package name (not it's path from the root InnerDetector/InDetValidation/) as it will search through the repository for a directory named as you typed with a CMakeLists.txt file.

Decide whether you need to get rid of 'untracked' files or add them. Then commit the changes to that branch. For a sanity check, check the svn code browser comparing the svn trunk to the tag in the commithash of the last svn commit (https://svnweb.cern.ch/trac/atlasoff/....)
~~~
git status
(e.g - rm -rf InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/)
git commit -a -m  [commit message about untracked files]
git add [FILE1 FILE2 ...]
git commit -a -m [add commit message here for tracked files]
~~~
Switch to the branch on the head and merge. 
~~~
git checkout 21.0_Update-Git
git merge --hours 21.0_Last-Svn ("--hours" to prioritise git changes over svn changes)
~~~
Resolve conflicts if there are any
~~~
git mergetool
(vim )
~~~
Setup new test environment 
~~~
mkdir ../build && cd ../build
setupATLAS
asetup 21.0,r28,Athena,gcc62
~~~
Since the entirety of Athena is cloned, Add InDetPhysValMonitoring to  the list of packages that you want to build (see justGitThings.md)
~~~
cp ../athena/Projects/WorkDir/package_filters_example.txt ../package_filters.txt
vim ../package_filters.txt
~~~
> -ADD- 
>    >    + InnerDetector/InDetValidation/InDetPhysValMonitoring
~~~
cmake -DATLAS_PACKAGE_FILTER_FILE=../package_filters.txt ../athena/Projects/WorkDir
make -j8

mkdir ../run && cd ../run
setupATLAS && lsetup rucio 
voms-proxy-init -voms atlas
rucio download --nrandom=1 mc16_valid:mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00
ln -s mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00/<*TAB*> AOD.pool.root
~~~

Run local tests (and the ones shown previously in TrackSelec[...]Task.md)
~~~
source ../build/x86_64-slc6-gcc62-opt/setup.sh 
athena ../athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/PhysVal_jobOptions.py

setupATLAS
asetup 21.0.20,Athena,gcc62
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateLooseTracks.set_Value_and_Lock(True); InDetPhysValFlags.doValidateTightPrimaryTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.LooseandTightP_21_000012.pool.root.1" > LooseandTightP.log 2>&1
~~~

If this set of directory commands work then the release is good for release 21. Now test with release 20
~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_4_20-20
mkdir source-20-20-8-9 && cd source-20-20-8-9
setupATLAS && asetup 20.20.8.9,here
pkgco.py -A InDetPhysValMonitoring # to get the svn trunk
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
cmt config (the pkgco.py command effectively does this command)
make 
~~~
However this will only obtain the release on the trunk. I have to copy all the changes by hand from the hybrid directory to test them before I can push them to the trunk to easily test them. 
DO SO LATER - AT THIS STAGE WE JUST CHECK THE TRUNK.
~~~
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
cmt config
make
cd -

Reco_tf.py  --steering doRAWtoALL  --checkEventCount  False --ignoreErrors True --maxEvents 500 --valid True --inputRDOFile root://eosatlas.cern.ch//eos/atlas/atlasgroupdisk/perf-idtracking/dq2/rucio/mc15_13TeV/d6/dc/RDO.08543088._000001.pool.root.1 --outputNTUP_PHYSVALFile physval.root --outputAODFile physval.AOD.root --validationFlags doInDet --preExec "from InDetRecExample.InDetJobProperties import InDetFlags;InDetFlags.doSlimming.set_Value_and_Lock(False);rec.doTrigger.set_Value_and_Lock(False); from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateTightPrimaryTracks.set_Value_and_Lock(True);InDetPhysValFlags.doValidateTracksInJets.set_Value_and_Lock(True);InDetPhysValFlags.doValidateGSFTracks.set_Value_and_Lock(False);rec.doDumpProperties=True;rec.doCalo=False;rec.doEgamma=False;rec.doForwardDet=False;rec.doInDet=True;rec.doJetMissingETTag=False;rec.doLArg=False;rec.doLucid=False;rec.doMuon=False;rec.doMuonCombined=False;rec.doSemiDetailedPerfMon=True;rec.doTau=False;rec.doTile=False;"
~~~
The above command does two things at once. Firstly it turns the input RDO file into an AOD and runs the decorators upon finishing and secondly it runs the validation framework on the resulting AOD. Check the log files log.PhysicsValidation and log.RAWtoALL 

To get a quick way of seeing what has been changed between version run
~~~
cd ../athena
git diff 21.0_Last-Svn 21.0_Update-Git -- > InnerDetector/InDetValidation/InDetPhysValMonitoring
~~~
Either these changes can be added in by hand, or each change is given it's own patch and implimented seperately. Goetz says that there is a commit (7ad3b85d8d) that makes a change not compatible with Release 20 and needs to be removed by hand.
~~~
git show 7ad3b85d8d
~~~
In the I.../I.../I.../python/InDetPhysValDecoration.py file the variable checkDeadElementsOnTrack is not defined in r20.20. I have to put a try: except: to only run the changes in this commit if the resulting release that has been run by the user is 21.0 or above. Asking for forgiveness is easier that asking for permission. The aim is to see what kind of exceptions that running the code in 20.20 comes up with and try to account for them all in an except.
~~~
vim InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValDecoration.py
~~~
> ADD
>   >     try:
>   >        [Current code block]
>   >        pass
>   >     except [exceptions to be worked out]:
>   >        [Edited code block with non-R20-friendly bits taken out]
>   >        pass
        
This will be repeated several times. Each time when finished run
~~~
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
cmt config
make
cd -
rm * # To remove all the files generated by the last try
Reco_tf.py  --steering doRAWtoALL  --checkEventCount  False --ignoreErrors True --maxEvents 5 --valid True --inputRDOFile root://eosatlas.cern.ch//eos/atlas/atlasgroupdisk/perf-idtracking/dq2/rucio/mc15_13TeV/d6/dc/RDO.08543088._000001.pool.root.1 --outputNTUP_PHYSVALFile updated.physval.root --outputAODFile update.physval.AOD.root --validationFlags doInDet --preExec "from InDetRecExample.InDetJobProperties import InDetFlags;InDetFlags.doSlimming.set_Value_and_Lock(False);rec.doTrigger.set_Value_and_Lock(False); from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateTightPrimaryTracks.set_Value_and_Lock(True);InDetPhysValFlags.doValidateTracksInJets.set_Value_and_Lock(True);InDetPhysValFlags.doValidateGSFTracks.set_Value_and_Lock(False);rec.doDumpProperties=True;rec.doCalo=False;rec.doEgamma=False;rec.doForwardDet=False;rec.doInDet=True;rec.doJetMissingETTag=False;rec.doLArg=False;rec.doLucid=False;rec.doMuon=False;rec.doMuonCombined=False;rec.doSemiDetailedPerfMon=True;rec.doTau=False;rec.doTile=False;"
~~~
Catalogue the error and look at the log files to see if they help identify the source of the error to add to the exception list.
Once this compiles it needs to be added into svn. Move changes back into svn removing copywrite statements that were put in by
svnpull.py?
~~~
svn commit -m"Porting compatible Release 21.0 changes into 20.20" 
~~~
Once the code it on the trunk. It needs to be tested. Ensure that the  directory is actually able to run over an ITk job. First run a control sample on nothing
~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_4_20-20/source-20-20-8-9
setupATLAS
mkdir Control && cd Control
asetup 20.20.8.9,here
svn co $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk -r810776 InnerDetector/InDetValidation/InDetPhysValMonitoring
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
echo "InDetPhysValMonitoring-r810776" > version.cmt
cmt config
source setup.sh
gmake
cd ../run
athena PhysValITk_jobOptions.py
cp MyPhysVal.root MyPhysValRef.root
~~~

The svn co $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk -r810776 line above will check out the Revision 810776 of the code. In https://svnweb.cern.ch/trac/atlasoff/log/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk you can see that the 810776 is the snapshot just before your last sets of changes.
~~~
cd ../../../../..
mkdir GitPortTest && cd GitPortTest
asetup 20.20.8.9, here
pkgco.py -A InDetPhysValMonitoring PhysValMonitoring
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
cmt config
source setup.sh
gmake
cd ../run/
athena PhysValITk_jobOptions.py
cp MyPhysVal.root MyPhysValTest.root 
~~~
To ensure that both of the files are the same
~~~
cd ../../../../../../..
mkdir UsefulStuff && cd UsefulStuff
setupATLAS
~~~
If you don't already have a root_helpers_shared directory 
~~~
> lsetup git
> git clone https://:@gitlab.cern.ch:8443/lmijovic/root_helpers_shared.git
> cd root_helpers_shared
> mkdir bin obj
~~~
Otherwise go straight to here
~~~
cd root_helpers_shared
rm test.root ref.root
asetup AtlasProduction,20.20.8.9,here
ln -s ../../ATHENA_4_20-20/source-20-20-8-9/Control/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/MyPhysValRef.root ref.root
ln -s ../../ATHENA_4_20-20/source-20-20-8-9/GitPortTest/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/MyPhysValTest.root test.root
make
~~~
Don't worry if at this point you get "make: Nothing to be done for 'all'." errors at this point. Make works on the base of time stamps. If you do not change anything in the source file, then compiler has nothing to do with your project. Make does not work like gcc to compile every time whether new build is needed or not. This is one of many advantages of using make for your project.
~~~
./bin/rooth_compare_files > Gittosvn.out
~~~
If the Gittosvn.out file shows (1's) then the two histograms are identical to within the rounding rounding precision (double prec) given in rooth_compare_files.cxx 
~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_4_20-20/source-20-20-8-9
setupATLAS
rm -rf Control GitPortTest
setupATLAS
asetup AtlasProduction,20.20.8.9,here
svn cp $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk -r 810807 $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/tags/InDetPhysValMonitoring-00-04-60/ -m "your message, eg synchornization with git"
~~~
The above will give you the tag InDetPhysValMonitoring-00-04-60. To look at the edited files.
~~~
asetup 20.20.10.5, here
pkgco.py InDetPhysValMonitoring-00-04-60
~~~
Nb, if you  will find something you want to change  some files, you should not edit the files of the tag  directly. Instead, you should check out the trunk (pkgco.py -A InDetPhysValMonitoring ), make your edits there, and re-commit.

