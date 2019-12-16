# This documents how to move the changes for the Track selections into svn and how to compare them to the git version. #

Last Edited: 09-10-2017

~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development
mkdir ATHENA_4-20-20 && cd SVN_ATHENA_4-20-20
setupATLAS
asetup 20.20.10.5,here
svn co svn+ssh://svn.cern.ch/reps/atlasoff/InnerDetector/InDetValidation/InDetPhysValMonitoring
~~~
One-by-one swap in (or edit) the modified files (TrackSelectionTool.py, InDetPhysValMonitoringTool.py, InDetPhysValDecoration.py and InDetPhysValJobProperties.py) in to the filesystem under trunk, cd to the directory where they are and then do:
~~~
cd InDetPhysValMonitoring/trunk/python
cp ../../../../../ATHENA_Loose/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/TrackSelectionTool.py .
cp ../../../../../ATHENA_Loose/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValMonitoringTool.py .
cp ../../../../../ATHENA_Loose/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValJobProperties.py .
cp ../../../../../ATHENA_Loose/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValDecoration.py .
~~~
OR create a patch file of the differences between the files on the svn trunk 
and the files with the Loose selection added. 
~~~ 
cd /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_Loose/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python

diff InDetPhysValDecoration.py ../../../../../../ATHENA_4_20-20/source_20-20-10-5/InDetPhysValMonitoring/trunk/python/[a,b,c,d]
~~~
> [a] TrackSelectionTool.py > TrackSelectionTool.patch
> [b] InDetPhysValJobProperties.py > InDetPhysValJobProperties.patch
> [c] InDetPhysValDecoration.py > InDetPhysValDecoration.patch
> [d] InDetPhysValMonitoringTool.py > InDetPhysValMonitoringTool.patch
~~~
patch TrackSelectionTool.py TrackSelectionTool.patch
patch InDetPhysValJobProperties.py InDetPhysValJobProperties.patch
patch InDetPhysValDecoration.py InDetPhysValDecoration.patch
patch InDetPhysValMonitoringTool.py InDetPhysValMonitoringTool.patch
rm (*).patch
svn commit -m "Addition of Loose Selection in IDPVM package" TrackSelectionTool.py InDetPhysValMonitoringTool.py InDetPhysValJobProperties.py InDetPhysValDecoration.py
~~~
> ------------------------------------
> Sending        InDetPhysValJobProperties.py
> Sending        InDetPhysValMonitoringTool.py
> Sending        TrackSelectionTool.py
> Transmitting file data ...
> Committed revision 808440.
> 
> Warning: post-commit hook failed (exit code 1) with output:
> 
> WARNING: Releases 21.0 and 22.0 have now migrated to git based development.
> Therefore no SVN tags can now be added to those releases.
> Please see https://atlassoftwaredocs.web.cern.ch/gittutorial/
-----------------------------------

Repeat the above with the jobOptions File.
~~~
cd ../../..
svn co svn+ssh://svn.cern.ch/reps/atlasoff/PhysicsAnalysis/PhysicsValidation/PhysValMonitoring
cd PhysValMonitoring/trunk/share
cp ../../../../../ATHENA_Loose/athena/PhysicsAnalysis/PhysicsValidation/PhysValMonitoring/share/PhysValInDet_jobOptions.py .
svn commit -m"job Options edit to acommodate a Loose Selection" PhysValInDet_jobOptions.py
~~~
> -------------------------------------
> Sending        PhysValInDet_jobOptions.py
> Transmitting file data .svn: Commit failed (details follow):
> svn: Access denied
> -------------------------------------
> 
> Solution: ASK FOR PREMISSION AND TRY AGAIN.

Once the code it on the trunk. It needs to be tested. Ensure that the  directory is actually able to run over an ITk job. First run a control sample on nothing
~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_4_20-20/source_20-20-10-5
setupATLAS
mkdir Control && cd Control
asetup 20.20.10.5,here
svn co $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk -r810776 InnerDetector/InDetValidation/InDetPhysValMonitoring
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
echo "InDetPhysValMonitoring-r810776" > version.cmt
cmt config
source setup.sh
gmake
cd ../run
athena PhysValITk_jobOptions.py
cp MyPhysVal.root MyPhysValRef.root
mv MyPhysValRef.root /afs/cern.ch/work/d/dspiteri/public 
~~~
The svn co
$SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk
-r810776 line above will check out the Revision 810776 of the code. In
https://svnweb.cern.ch/trac/atlasoff/log/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk
you can see that the 810776 is the snapshot just before your last sets
of changes.
~~~
cd ../../../../..
mkdir LooseTest && cd LooseTest
asetup 20.20.10.5, here
pkgco.py -A InDetPhysValMonitoring PhysValMonitoring
cd InnerDetector/InDetValidation/InDetPhysValMonitoring/cmt/
source setup.sh
gmake
cd ../run/
athena PhysValITk_jobOptions.py
cp MyPhysVal.root MyPhysValTest.root
mv MyPhysValTest.root /afs/cern.ch/work/d/dspiteri/public 
~~~
To ensure that both of the files are the same
~~~
cd ../../../../../../..
mkdir UsefulStuff && cd UsefulStuff
setupATLAS && lsetup git
git clone https://:@gitlab.cern.ch:8443/lmijovic/root_helpers_shared.git
cd root_helpers_shared
asetup AtlasProduction,20.20.8.9,here
mkdir bin obj
ln -s ../../ATHENA_4_20-20/source_20-20-10-5/Control/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/MyPhysVal.root ref.root
ln -s ../../ATHENA_4_20-20/source_20-20-10-5/LooseTest/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/MyPhysVal.root test.root
or
> ln -s /afs/cern.ch/work/d/dspiteri/public/MyPhysValRef.root ref.root
> ln -s /afs/cern.ch/work/d/dspiteri/public/MyPhysValTest.root test.root
make
./bin/rooth_compare_files > LooseSelection.out
~~~
If the LooseSelection.out file shows (1's) then the two histograms are identical to within the rounding rounding precision (double prec) given in rooth_compare_files.cxx.

Once satisfied with this then it's time to patch the revised version of the trunk into a tag and to clean up the folder in terms of space. 
~~~
cd /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_4_20-20/source_20-20-10-5
setupATLAS
rm -rf Control LooseTest
setupATLAS
asetup AtlasProduction,20.20.8.9,here
svn cp $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/trunk -r 810807 $SVNROOT/InnerDetector/InDetValidation/InDetPhysValMonitoring/tags/InDetPhysValMonitoring-00-04-60/ -m "Git to SVN port of the Release-20-compatible Release 21 amendments to the IDPVM and the PVM packages"
~~~





