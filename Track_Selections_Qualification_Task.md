# This will be aimed at my second task which was to add track selections to   the InDetPhysValMonitoring Tool#
=================================================
Last Edited: 09-10-2017
-------------------------------------------------

--------------------------From Goetz' email-------------------------
At the moment, there is the hack "addExtraMonitoring"
ttps://gitlab.cern.ch/atlas/athena/blob/21.0/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValDecoration.py#L359)
which I put in place to avoid having to change the job options in
PhysValMonitoring. But in principle it would make sense to move this also to
the new hook getInDetPhysValMonitoringTools (if it is introduced).

The job options to set up the IDPVM tools need to be changed to something like
this :

> from InDetPhysValMonitoring.InDetPhysValMonitoringTool import
> InDetPhysValMonitoringTool
> 
> indet_mon_tool = InDetPhysValMonitoringTool.InDetPhysValMonitoringTool()
> indet_mon_tool_tight =
> InDetPhysValMonitoringTool.InDetPhysValMonitoringToolTight()
> 
> ToolSvc += indet_mon_tool_tight
> 
> monMan.AthenaMonTools += [indet_mon_tool,indet_mon_tool_tight]

Eventually it would make sense to introduce flags or a detail level to allow
to switch on the monitoring of the various track selections by preExec
statements.

Some flags are already defined here:
https://gitlab.cern.ch/atlas/athena/blob/21.0/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValJobProperties.py)

Not sure what the better solution is:

a) one flag for each track selection.
b) a numeric level of detail which would then switch on an increasing number
   of monitoring tools with increasing level.
c) a list of strings e.g. ["tight","robust_tight","loose","all"]

Then the configurations of the monitoring tools for the various track
selections have to be added like it is done for TightPrimary here:
https://gitlab.cern.ch/atlas/athena/blob/21.0/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/InDetPhysValMonitoringTool.py#L88)

The configurations for the track selection tools can be added following the
example of InDetTrackSelectionToolTightPrimary here:
(https://gitlab.cern.ch/atlas/athena/blob/21.0/InnerDetector/InDetValidation/InDetPhysValMonitoring/python/TrackSelectionTool.py#L13)
---------------------------------------------------------------------

Local Cloning of Athena - Sparce Checkout Packages Required 

> git atlas init-workdir https://:@gitlab.cern.ch:8443/atlas/athena.git
> cd athena
> git atlas addpkg InDetPhysValMonitoring PhysValMonitoring

Topic Branch Name

> git fetch upstream 
> git checkout -b master_Track-Selections-QT origin/master --no-track
> asetup master,r28,Athena,gcc62

No need to actually define a Loose cut. Found these on gitlab
 https://gitlab.cern.ch/atlas/athena/blob/21.0/InnerDetector/InDetRecTools/InDetTrackSelectionTool/Root/InDetTrackSelectionTool.cxx 

case CutLevel::Loose :
    setCutLevelPrivate(CutLevel::NoCut, overwrite); // if hard overwrite, reset all cuts first. will do nothing if !overwrite
    // change the cuts if a hard overwrite is asked for or if the cuts are unset
	    if (overwrite || m_maxAbsEta >= LOCAL_MAX_DOUBLE) m_maxAbsEta = 2.5;
	    if (overwrite || m_minNSiHits < 0) m_minNSiHits = 7;
	    m_maxOneSharedModule = true;
	    if (overwrite || m_maxNSiHoles >= LOCAL_MAX_INT) m_maxNSiHoles = 2;
	    if (overwrite || m_maxNPixelHoles >= LOCAL_MAX_INT) m_maxNPixelHoles = 1;
	    break;

case CutLevel::LoosePrimary :
    setCutLevelPrivate(CutLevel::NoCut, overwrite); // implement loose cuts first
	    if (overwrite || m_maxAbsEta >= LOCAL_MAX_DOUBLE) m_maxAbsEta = 2.5;
	    if (overwrite || m_minNSiHits < 0) m_minNSiHits = 7;
	    m_maxOneSharedModule = true;
	    if (overwrite || m_maxNSiHoles >= LOCAL_MAX_INT) m_maxNSiHoles = 2;
	    if (overwrite || m_maxNPixelHoles >= LOCAL_MAX_INT) m_maxNPixelHoles = 1;
	    if (overwrite || m_minNSiHitsIfSiSharedHits < 0) m_minNSiHitsIfSiSharedHits = 10;
	    break;


- Add InDetPhysValMonitoringToolLoose class to InDetPhysValMonitoringTool.py
- Move the file PhysValInDet_jobOptions.py to getInDetPhysValMonitoring.py
- Move contents of ExtraMonitoring to getInDetPhysValMonitoring.py and edit PhysValInDet_jobOptions.py accordingly
- Add doValidateLooseTracks class and container item to InDetPhysValJobProperties.py
- Add InDetTrackSelectionToolLoose to TrackSelectionTool.py

Once done, the changes need to be built. Make sure you have run the asetup
command

> cd <athena root path>
> mkdir ../build && cd ../build
> cmake ../athena/Projects/WorkDir #This folder needs to have CMakeLists.txt
> make -j
> source x86_64-slc6-gcc62-opt/setup.sh 

To test these changes realise the the IDPVM selection is in the p-tag.
Download a file to test and find the name of the root file.
<SEE POINT 6 IN justGitthings>

mkdir ../run && cd ../run 

> lsetup rucio
> voms-proxy-init -voms atlas
> rucio download --nrandom=1 mc16_valid:mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00
> ln -s mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00/<NAMEOFFILE> AOD.pool.root


> mkdir ATHENA-TEST && cd ATHENA-TEST
> setupATLAS
> lsetup git
> git atlas init-workdir https://:@gitlab.cern.ch:8443/atlas/athena.git
> cd athena
> git atlas addpkg InDetPhysValMonitoring PhysValMonitoring
> git fetch upstream 
> git checkout -b master_Track-Selections-QT upstream/master --no-track
> asetup master,r28,Athena,gcc62 
> ls -ltr Projects/WorkDir #make sure this has the CMakeLists.txt file in it
> mkdir ../build && cd ../build
> cmake ../athena/Projects/WorkDir #This folder needs to have CMakeLists.txt
> make -j
> source ../build/x86_64-slc6-gcc62-opt/setup.sh 
> mkdir ../run && cd ../run
> lsetup rucio
> voms-proxy-init -voms atlas
> rucio download --nrandom=1 mc16_valid:mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00
> ln -s mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00/#<NAMEOFFILE># AOD.pool.root

After these steps we are now ready to do some debugging tests

# 1) Test your athena download works
> athena ../athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/PhysVal_jobOptions.py

# 2) Test the packages you have compile and build correctly
> Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507_000101.pool.root.1" > Trial.log 2>&1

# 3) Test the infrastructure of --preExec
> Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateTightPrimaryTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.TightPrimary_000101.pool.root.1" > TightPrimary.log 2>&1

# 4) Test the changed infrastructure of --preExec
> Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateLooseTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.Loose_000101.pool.root.1" > Loose.log 2>&1

Once the code runs. Do a quick run over one or two files, make the IDPVMs,
create the webpages and check the plots to see it (a) doesn't crash and (b)
does what is intended?  (i.e. look at one or two plots of variables where
the default/loose/tightprimary selection explicitly differs). For a reminder 
how to do this, see ESD-NTUP_PhysVal comparison.md
For this you will want to make suer that you used the same file you
downloaded for both the Loose and the TightPrimary Reco.py tests (if you 
have been following this to the letter then this should be the case). 

In the run directory you should have thee different NTUP_PHYSVAL files. One
with _Loose, one with _TightPrimary and one with nothing. To avoid confusion 
change the number(s) between 'NTUP_PHYSVAL.' and '.pool.root.1' to match
that of the downloaded file is that is not the case already. 

Now to go to your web area and create the test

> cd ~/www/physval_make_web_display/
> mkdir LooseTest && cd LooseTest
> setupATLAS
> asetup 21.0.20,here
> physval_make_web_display.py --reffile TightPrimary:///afs/cern.ch/user/d/dspiteri/Code_Development/ATHENA/run/NTUP_PHYSVAL.11027507.TightPrimary_000006.pool.root.1 --outdir=$PWD --title LooseTest /afs/cern.ch/user/d/dspiteri/Code_Development/ATHENA/run/NTUP_PHYSVAL.11027507.Loose_000006.pool.root.1 --startpath=IDPerformanceMon

Web access can be found at:
https://dspiteri.web.cern.ch/dspiteri/physVal_make_web_display/LooseTest/
In such a test the Loose catagory should appear, but it is undefined. 

# 5) Create a file with both Loose and TightPrimary tags

> cd ~/Code_Development/ATHENA/build/
> source x86_64-slc6-gcc62-opt/setup.sh
> cd ../run
> setupATLAS
> asetup master,r28,Athena,gcc62 #(or asetup 21.0,r28,Athena,gcc62)
> Reco_tf.py --inputAODFile="AOD.pool.root" --maxEvents="10" --preExec "all:from InDetPhysValMonitoring.InDetPhysValJobProperties import InDetPhysValFlags; InDetPhysValFlags.doValidateLooseTracks.set_Value_and_Lock(True); InDetPhysValFlags.doValidateTightPrimaryTracks.set_Value_and_Lock(True);" --skipEvents="0" --valid="True" --runNumber="410000" --digiSeedOffset1="3" --digiSeedOffset2="3" --jobNumber="1" --validationFlags doInDet --outputNTUP_PHYSVALFile="NTUP_PHYSVAL.11027507.LooseandTightP_000006.pool.root.1" > LooseandTightP.log 2>&1

Refer to LooseandTightP.log to ensure that everything behaved as expected. 
Now to go to your web area and create the second test

> cd ~/www/physVal_make_web_display/
> mkdir LooseTest2 && cd LooseTest2
> setupATLAS
> asetup 21.0.20,here
> physval_make_web_display.py --reffile TightPrimary:///afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_Loose/run/NTUP_PHYSVAL.11027507.LooseandTightP_000006.pool.root.1 --outdir=$PWD --title Loose_TightTest /afs/cern.ch/work/d/dspiteri/Code_Development/ATHENA_Loose/run/NTUP_PHYSVAL.11027507.LooseandTightP_000006.pool.root.1 --startpath=IDPerformanceMon 

Web access can be found at:
https://dspiteri.web.cern.ch/dspiteri/physVal_make_web_display/LooseTest2/
In such a test, all catagories that appeared in the previous catagory should
appear (Tracks, Loose, TightPrimary). 

Once you are satisfied with this these changes need to go into git
and eventually will be merged. See from step 9 justGitthings.md on how to do
this. 

     



