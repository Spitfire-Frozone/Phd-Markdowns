## Running PhysVal_make_web_display ##
===============================================================================
Last Edited: 10-01-2017
-------------------------------------------------------------------------------

```Bash

Get a particular NTUP_PHYSVAL file from an ESD, or anywhere.
> rucio download valid2:NTUP_PHYSVAL.09826102._000001.pool.root.1

cd to your web area where you want to make the display
Set up ATLAS infrastructure, set up Rucio and Athena, set up ATLAS VO and change to the Athena directory. Build $PYTHONPATH correctly

> setupATLAS 
> lsetup asetup
> lsetup rucio
> voms-proxy-init -voms atlas
> asetup 21.0.14,here
> mkdir firstWorkingTest
> cd firstWorkingTest
> physval_make_web_display.py --reffile ${refname}://${reffile} --outdir=$PWD --title "${testname}" ${testfile} --startpath=IDPerformanceMon

refname and testname are user inputs that can be anything. reffile and testfile are #names of the files you want to compare and you need the complete file path for this to work

> e.g physval_make_web_display.py --reffile ref2:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDDCubeTutorial/ source/valid2/NTUP_PHYSVAL.09826102._000001.pool.root.1 --outdir=$PWD --title "test101" /afs/cern.ch/user/d/dspiteri/PhysVal/ESDDCubeTutorial/source/valid2/NTUP_PHYSVAL.09826102._000001.pool.root.1 --startpath=IDPerformanceMon
```
