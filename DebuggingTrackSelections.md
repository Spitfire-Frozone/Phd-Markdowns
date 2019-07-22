# The main cause for most errors is a cluttering of environment variables. The best thing to do in most cases is to open a new terminal/exit and start afresh OR obtain a new sparse checkout of athena.

## Debugging Git and Software Errors in Track_Selection_Qualification_Task.md ##
===============================================================================
## Last Edited: 12-07-2017 ##


First check if your packages run as normal
~~~
cd ~/Code_Development #or whatever directory you want to create your athena area
mkdir ATHENA-TEST && cd ATHENA-TEST
setupATLAS
lsetup git
git atlas init-workdir https://:@gitlab.cern.ch:8443/atlas/athena.git
~~~
If you have already done some work 
~~~
cd athena
git atlas addpkg InDetPhysValMonitoring PhysValMonitoring
git fetch upstream 
git checkout -b master_Track-Selections-QT origin/master --no-track

asetup master,r28,Athena,gcc62
ls -ltr Projects/WorkDir #make sure this has the CMakeLists.txt file in it
mkdir ../build && cd ../build
cmake ../athena/Projects/WorkDir #This folder needs to have CMakeLists.txt
make -j6
source ../build/x86_64-slc6-gcc62-opt/setup.sh 
mkdir ../run && cd ../run
lsetup rucio
voms-proxy-init -voms atlas
rucio download --nrandom=1 mc16_valid:mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00
ln -s mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00/#<NAMEOFFILE># AOD.pool.root
~~~
After these steps we are now ready to do some debugging tests. Make sure you check all of the log files after they are created for errors. Typically they report errors either around line 78 or line 1060.

- 1) Test your athena download works
~~~
athena ../athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/PhysVal_jobOptions.py
~~~
- 2) Test the packages you have compile and build correctly
~~~
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507_000101.pool.root.1" > Trial.log 2>&1
~~~
- 3) Test the infrastructure of --preExec
~~~
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateTightPrimaryTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.TightPrimary_000101.pool.root.1" > TightPrimary.log 2>&1
~~~
Add your files to where they should go from the non-fresh ATHENA and to the their respective direcories directory. This is unique to the layout of ATHENA  present in the ATHENA.tar also sent to you.
~~~
cd ../athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/
cp ../../../../../../ATHENA/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValMonitoringTool.py .
cp ../../../../../../ATHENA/athena/Projects/WorkDir/InDetPhysValJobProperties.py . 
cp ../../../../../../ATHENA/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/TrackSelectionTool.py .
cd ../../../../PhysicsAnalysis/PhysicsValidation/PhysValMonitoring/share/
cp ../../../../../../ATHENA/athena/PhysicsAnalysis/PhysicsValidation/PhysValMonitoring/share/PhysValInDet_jobOptions.py .
~~~
Files of importance 
- InDetPhysValMonitoringTool.py - Added class InDetPhysValMonitoringToolLoose
- InDetPhysValJobProperties.py - Added class doValidateLooseTracks and to list
- TrackSelectionTool.py - Added class InDetTrackSelectionToolLoose
- PhysValInDet_jobOptions.py - Added indet_mon_Loose

- 4) Test the changed infrastructure of --preExec
~~~
cd ../../../run/
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateLooseTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.Loose_000101.pool.root.1" > Loose.log 2>&1
~~~
>  --------------------ERROR MESSAGE-----------------
>  89 PyJobTransforms.transform.execute 2017-05-22 23:17:43,505 WARNING Transform
   now exiting early with exit code 65 (Non-zero return code from PhysicsValidation (8); Logfile error in log.PhysicsValidation: "AttributeError: JobPropertyContainer:: JobProperties.InDetPhysValJobProperties does not have property doValidateLooseTracks. Did you mean 'doValidateGSFTracks'?")

If this doesn't work (and I strongly suspect it will not) and you get the above error message try rebuilding before retrying the command above
~~~
cd ../build
(rm -rf ../build && mkdir ../build && cd ../build #instead if you want safety)
cmake ../athena/Projects/WorkDir
make -j6
source ../build/x86_64-slc6-gcc62-opt/setup.sh #Don't forget this!
cd ../run
~~~
You don't need to rerun athena [PATHTO]/jobOptions.py test if you have not edited the jobOptions file.

Looking in the x86_64-slc6-gcc62-opt/setup.sh file shows that it takes the local definition of PYTHONPATH. If you are in your work, home or data directory you will need to append the x86_64-slc6-gcc62-opt/python to the variable PYTHONPATH.
~~~
echo $PYTHONPATH
~~~
Try going into the build directory and sourcing the setup.sh file again.
~~~
cd ../build
source x86_64-slc6-gcc62-opt/setup.sh 
echo $PYTHONPATH
~~~
If this doesn't produce <PWD/x86_64-slc6-gcc62-opt/python:[BLAH]> as PYTHONPATH seek help 
~~~
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateLooseTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.Loose_000101.pool.root.1" > Loose.log 2>&1
~~~
>  --------------------ERROR MESSAGE-----------------
>  1060 PyJobTransforms.transform.execute 2017-05-22 23:28:03,267 WARNING Transform
     now exiting early with exit code 65 (Non-zero return code from PhysicsValidation (8); Logfile error in log.PhysicsValidation: "TypeError: Can not append InDetPhysValMonitoringToolTightPrimary (Private AlgTool) to a PublicToolHandleArray
     
Looking in the log.PhysicsValidation file now in the run directory to the last lines

>   >   1014  File                               "/cvmfs/atlas.cern.ch/repo/sw/software/21.0/GAUDI/21.0.20/InstallArea/x86_64-slc6-gcc62-opt/python/GaudiKernel/GaudiHandles.py", line 173, in append
>   >   1015 (value.__class__.__name__, pop, value.getGaudiType(), self.__class__.__name__) )
>   >   1016 TypeError: Can not append InDetPhysValMonitoringToolTightPrimary (Private AlgTool) to a PublicToolHandleArray
>   >   1017 Py:Athena            INFO leaving with code 8: "an unknown exception occurred"

Try re-arranging the order of the selections in the packages. Swap all objects that mention Loose and TightPrimary in all edited files in the work directory and re-insert them into the appropriate place in their respective packages.

I logged out and back in again at this point
~~~
cd ~/Code_Development/ATHENA-TEST
setupATLAS
asetup 21.0.20,Athena,gcc62
cd build
cmake ../athena/Projects/WorkDir
make -j6
source ../build/x86_64-slc6-gcc62-opt/setup.sh #Don't forget this!
cd ../run
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateLooseTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.Loose_000101.pool.root.1" > Loose.log 2>&1
~~~
>  --------------------ERROR MESSAGE-----------------
>  1060 PyJobTransforms.transform.execute 2017-05-22 23:28:03,267 WARNING Transform
     now exiting early with exit code 65 (Non-zero return code from PhysicsValidation (8); Logfile error in log.PhysicsValidation: "TypeError: Can not append InDetPhysValMonitoringToolLoose (Private AlgTool) to a PublicToolHandleArray


Looking in the log.PhysicsValidation file now in the run directory to the last lines. 

>   >   1014  File                               "/cvmfs/atlas.cern.ch/repo/sw/software/21.0/GAUDI/21.0.20/InstallArea/x86_64-slc6-gcc62-opt/python/GaudiKernel/GaudiHandles.py", line 173, in append
>   >   1015 (value.__class__.__name__, pop, value.getGaudiType(), self.__class__.__name__) )
>   >   1016 TypeError: Can not append InDetPhysValMonitoringToolLoose (Private AlgTool) to a PublicToolHandleArray
>   >   1017 Py:Athena            INFO leaving with code 8: "an unknown exception occurred"

#All instances of TightPrimary in these errors are now labelled Loose.

Goetz says that these errors could be caused by improper additions of the toolto a list. He says that the tool service classes need to be added like mon_manager.AthenaMonTools += [toolFactory(InDetPhysValMonitoringTool.InDetPhysValMonitoringToolTightPrimary) ] in keeping in the way they are added in the Decoration.py file AND edited in the decoration file unlike suggested previously. So undo the name order swap in InDetPhyValMonitoringTool.py, TrackSelectionTool.py and InDetPhysValJobProperties.py in InnerDetector/InDetValidation/InDetPhysValMonitoring/python and PhysValInDet_jobOptions.py in PhysicsAnalysis/PhysicsValidation/PhysValMonitoring/share


- 5) Correct tool additions in the PhysValInDet_jobOptions.py file 
> ADD
>  >    mon_manager.AthenaMonTools += [toolFactory(indet_mon_tool,...)]

~~~
cd (PATH TO athena root)    
rm -rf ../build && mkdir ../build && cd ../build #instead if you want safety
cmake ../athena/Projects/WorkDir
make -j6
source ../build/x86_64-slc6-gcc62-opt/setup.sh #Don't forget this!
cd ../run
athena ../athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/PhysVal_jobOptions.py
Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507_000101.pool.root.1" > Trial.log 2>&1
~~~
>  --------------------ERROR MESSAGE-----------------
>  1051 PyJobTransforms.transform.execute 2017-07-13 14:31:48,277 WARNING       
     Transform now exiting early with exit code 65 (Non-zero return code from PhysicsValidation (8); Logfile error in log.PhysicsValidation: "NameError: name 'toolFactory' is not defined")

-Solution-
>   ADD Near the top add the line
>   >    from ConfigUtils import toolFactory #save and repeat from 6).

>  --------------------ERROR MESSAGE-----------------
>  1042 PyJobTransforms.transform.execute 2017-07-14 11:48:18,188 WARNING 
     Transform now exiting early with exit code 65 (Non-zero return code from PhysicsValidation (3); Logfile error in log.PhysicsValidation: "ImportError: No module named ConfigUtils")

-Solution-
>   ADD before the line inserted previously add
>   >    import InDetPhysValMonitoring.ConfigUtils #save and repeat instructions from 6)

>  --------------------ERROR MESSAGE-----------------
>  1042 PyJobTransforms.transform.execute 2017-07-14 11:48:18,188 WARNING 
     Transform now exiting early with exit code 65 (Non-zero return code from PhysicsValidation (3); Logfile error in log.PhysicsValidation: "ImportError: No module named ConfigUtils")

Delete all lines to do with toolFactory and put all of the tools into the tool
service

- 6) Correct tool additions in the PhysValInDet_jobOptions.py file 

> CHANGE
>    >   ToolSvc += indet_mon_tool -> ToolSvc += [indet_mon_tool,...]

>    >   mon_manager.AthenaMonTools += [toolFactory(indet_mon_tool,...)] -> mon_manager.AthenaMonTools += [indet_mon_tool,...]  
    
