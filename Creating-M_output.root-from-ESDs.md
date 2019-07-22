# This is aimed at the first part of qualification task of my PhD: Running and setting up a web interface to run and see plots more efficiently. This document is about editing old framework to be replaced. # 

## RUNNING DCUBE ##
<https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/InDetPhysValMonitoring>
===============================================================================
## Last Edited: 05-12-2016 ##

Set up ATLAS infrastructure, set up Rucio and Athena, set up ATLAS VO and change to the Athena directory. Build $PYTHONPATH correctly
~~~
setupATLAS 
lsetup rucio
voms-proxy-init -voms atlas
asetup 21.0.6,here
source ../build/x86_64-slc6-gcc49-opt/setup.sh
cd "${TestArea}"
~~~
Get a particular file from a ttbar sample.
~~~
rucio download valid3:ESD.05297574._000080.pool.root.1
~~~
Delete included sample files and create some symbolic links to this downloaded file to be consistent with the file path defined in the Job Options.
~~~
rm "${TestArea}"/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/AOD.pool.root
rm "${TestArea}"/../run/AOD.pool.root
test_file=""${PWD}"/valid3/ESD.05297574._000080.pool.root.1"
ln -s "${test_file}" "${TestArea}"/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/AOD.pool.root
ln -s "${test_file}" "${TestArea}"/../run/AOD.pool.root
~~~
Remove junk XML files.
~~~
rm "${TestArea}"/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/*xml
~~~
Change to the run directory and run the example Job Options piping the display cout and cerr into a file called run.log (2.&1 pipes cerr into cout).(& less run.log) opens the output in less parallelly for easy debugging 
~~~
cd "${TestArea}"/../run
athena "${TestArea}"/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/PhysVal_jobOptions.py > run.log 2>&1 & less run.log
~~~
