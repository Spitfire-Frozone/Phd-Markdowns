# This will be aimed at my the second part of my qualification task which is to join the efforts surrounding the study of fake rates. The second aim is to try and redo Otilia's study with new data and MC. 

## "R21" in Data and "R21 MC16c" ##
===============================================================================
Last Edited: 9-11-2017
-------------------------------------------------------------------------------
### I highly recommend you read Fake_Rate_Qualification_Task.md if you think some steps are skipped without explanation. This will hit the ground running so to speak. ###

See p104-105 of PhD Research I of how to select the data you want to use from a particular year. To get the data from the 2017 runs you want to use. Go to 
ami.in2p3.fr. Select 'Dataset Browser' in the top banner and then select the dataset specifics you desire.   
- Real Data  
- data17  

After this you add further discriminators to the data  
- projectName 'data17_13TeV'  
- prodStep 'merge'  
- dataType 'AOD'  
- streamName 'physics_ZeroBias'  
- runNumber min: 327341 max: 330471  

Then repeat for the MC data  
- Simulated Data   
- mc16  
- projectName 'mc16_13TeV'  
- prodStep 'merge'  
- dataType 'AOD'  
- subcampaign 'MC16c'  
- logicalDatasetName '*_nu_*'  

Then CTRL-F to find the run numbers that you want.  

> |[Data Samples] |  
>  ---------------  
> | data17_13TeV.00327342.physics_ZeroBias.merge.AOD.f832_m1812  | USED|  
> | data17_13TeV.00327764.physics_ZeroBias.merge.AOD.f836_m1824  | USED|  
> | data17_13TeV.00330470.physics_ZeroBias.merge.AOD.f843_m1824  | USED|  
> | data17_13TeV.00337404.physics_ZeroBias.merge.AOD.f871_m1879  | USED|  
> | data17_13TeV.00337491.physics_ZeroBias.merge.AOD.f873_m1879  | USED|  
> | data17_13TeV.00339849.physics_ZeroBias.merge.AOD.f889_m1897  | USED|  
>  ---------------  

> |[MC Samples] |
>  ---------------
> |   T = 13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711  |
> | Rel21: mc16_T_s3126_r9781_r9778  |
>  ---------------

The MC16c sample used here has 8,000,000 events, 800 files and 3.16TB size.  
The MC16a sample used before has 7,900,000 events, 792 files and 8.67TB size.  

> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/  
> cp -r ZeroBias_FakeTracks/ ZeroBias_FakeTracksR21/

Then either go to Release 20.7 or Release 21 instructions.

# Release 20.7 #

## Submitting your jobs to the grid ##
Try to build the 2017 data in an older release just to get a decent look at it.

> cd ZeroBias_FakeTracks  
> setupATLAS && lsetup panda && rcSetup Base,2.5.0  
> vim gridJobSubmit.sh  

>  >   ADD 
>  >   > export USER_GRID=dspiteri  
>  >   > source rcSetup.sh  
>  >   > rc find_packages && rc compile  
>  >   >
>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00327342  --inputDS data17_13TeV.00327342.physics_ZeroBias.merge.AOD.f832_m1812 --driver grid --suffix _Rel21_data17

>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00327764  --inputDS data17_13TeV.00327764.physics_ZeroBias.merge.AOD.f836_m1824 --driver grid --suffix _Rel21_data17
 
>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00330470  --inputDS data17_13TeV.00330470.physics_ZeroBias.merge.AOD.f843_m1824 --driver grid --suffix _Rel21_data17

>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00337404  --inputDS data17_13TeV.00337404.physics_ZeroBias.merge.AOD.f871_m1879 --driver grid --suffix _Rel21_data17

>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00337491  --inputDS data17_13TeV.00337491.physics_ZeroBias.merge.AOD.f873_m1879 --driver grid --suffix _Rel21_data17

>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00339849  --inputDS data17_13TeV.00339849.physics_ZeroBias.merge.AOD.f889_m1897 --driver grid --suffix _Rel21_data17

>  >   > python MyVertexAnalysis/scripts/Run.py --submitDir MC16c_R21_00159000_3 --inputDS mc16_13TeV:mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9781_r9778 --driver grid --suffix _Rel21_MC16c


> chmod u+x gridJobSubmit.sh

Before submitting the GRL in Run.py needs to be replaced as it is configured to run over the 2016 dataset. Need to add in the 2017 dataset ones.The GRL'S for the 2017 dataset can be found here: http://atlas.web.cern.ch/Atlas/GROUPS/DATABASE/GroupData/GoodRunsLists/data17_13TeV/20171019/ and is mirrored here:
/cvmfs/atlas.cern.ch/repo/sw/database/GroupData/GoodRunsLists/data17_13TeV/20171019/.

Physics analyses are advised to use 1.7e34 primaries by default. These are loosest unprescaled triggers in use throughout this GRL and have collected the maximum luminosity. They will not be in the RootCoreBin as this is not up to date so add in by hand the correct .xml file
"data17_13TeV.periodAllYear_DetStatus-v95-pro21-09_Unknown_PHYS_StandardGRL_All_Good_25ns_Triggerno17e33prim.xml"

> cp /cvmfs/atlas.cern.ch/repo/sw/database/GroupData/GoodRunsLists/data17_13TeV/20171019/data17_13TeV.periodAllYear_DetStatus-v95-pro21-09_Unknown_PHYS_StandardGRL_All_Good_25ns_Triggerno17e33prim.xml RootCoreBin/data/MyVertexAnalysis/GRL
 
> vim MyVertexAnalysis/scripts/Run.py  
>  > COMMENT OUT 
>  >   > alg.goodRunList = "..." (L222)   
>  >   > alg.blackRunLists = "..." (L230)  
>  > ADD 
>  >   > alg.goodRunList = "$ROOTCOREBIN/data/MyVertexAnalysis/GRL/data17_13TeV.periodAllYear_DetStatus-v95-pro21-09_Unknown_PHYS_StandardGRL_All_Good_25ns_Triggerno17e33prim.xml"

The GRL'S for the 2017 dataset can be found here: http://atlas.web.cern.ch/Atlas/GROUPS/DATABASE/GroupData/GoodRunsLists/data17_13TeV/20171019/. 
Physics analyses are advised to use 1.7e34 primaries by default. These are loosest unprescaled triggers in use throughout this GRL and have collected the maximum luminosity.

If you want to perform a local test now to ensure everything is fine you'll want to run over a sample.

> mkdir TestSamples && cd TestSamples  
> lsetup rucio && voms-proxy-init -voms atlas  
> rucio download --nrandom 1 data17_13TeV.00327342.physics_ZeroBias.merge.AOD.f832_m1812  
> cd ..  
> export USER_GRID=dspiteri  
> source rcSetup.sh  
> rc clean  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00327342  --input TestSamples/data17_13TeV.00327342.physics_ZeroBias.merge.AOD.f832_m1812/data17_13TeV.00327342.physics_ZeroBias.merge.AOD.f832_m1812._lb0874-lb0883._0001.1 --suffix _Rel21_data17 --overwrite  

Else
> ./gridJobSubmit.sh  


Take a break. Have a KitKat, or a Snickers or something while you are waiting for the jobs to finish. The next time you log in to your working area, open up two terminals. The best place to find all your jobs is http://bigpanda.cern.ch/tasks/?username=dwayne%20spiteri or to follow the link in the email you receive at the resolution of your job.

## Generating Comparison plots ##
> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks

Now to clean up the directory
> mkdir jobSubmissionFolders/16-09-17  
> mv D17* MC16* jobSubmissionFolders/16-09-17  
>   > There is no need to keep these, so you can just delete them instead

## (1) ##
> ssh -Y glasvr02  
> cd /data/dspiteri/FakeStudy/  
> setupATLAS && lsetup rucio  
> voms-proxy-init -voms atlas  
> vim gridOutputContainers2.txt  

>   > ADD  
>   >   > user.dspiteri.159000.ParticleGenerator_nu_E50._Rel21_MC16c_myOutput.root/  
>   >   > user.dspiteri.00327342.physics_ZeroBias._Rel21_data17_myOutput.root/  
>   >   > user.dspiteri.00327764.physics_ZeroBias._Rel21_data17_myOutput.root/  
>   >   > user.dspiteri.00330470.physics_ZeroBias._Rel21_data17_myOutput.root/  
>   >   > user.dspiteri.00337404.physics_ZeroBias._Rel21_data17_myOutput.root/    
>   >   > user.dspiteri.00337491.physics_ZeroBias._Rel21_data17_myOutput.root.161648137/  
>   >   > user.dspiteri.00339849.physics_ZeroBias._Rel21_data17_myOutput.root/
  
> rucio download `cat gridOutputContainers2.txt`  
> mkdir R21 MC16   
>   > From 01/11/17 Instances of R21 were replaced with 2017Data  
> mv user.dspiteri.00* 2017Data    
> mv user.dspiteri.1* MC16  


Obtain a filelist from the downloaded output folders. For example if we take the 2017Data directory...

> cd 2017Data  
> mv user.dspiteri.003*/user.dspiteri.1* .  
> rm -rf user.dspiteri.00*  
> pwd  
> vim R21DataFileList.txt  
>  > ADD
>  >   > [pwd/file1]  
>  >   > [pwd/file2]  
>  >   > ...  
>  >   > [pwd/filen]  

> cd ../MC16/  
> mv user.dspiteri*/user.dspiteri.1* .  
> rm -rf user.dspiteri.159000.ParticleGenerator_nu_E50._Rel21_MC16c_myOutput.root/  
> pwd  
> vim MC16cFileList.txt  
>  > ADD
>  >   > [pwd/file1]  
>  >   > [pwd/file2]  
>  >   > ...  
>  >   > [pwd/filen]  

## (2) ##  
> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks  
> setupATLAS   
> (rcSetup -u)  
> rcSetup Base,2.5.0  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/G-_-R21_data-327342-327764-330470-337404-337491 --filelist 2017DataFileList.txt --filelistarea /eos/user/d/dspiteri/FakeStudy/Data2017/

## (1) ##  
> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks  
> setupATLAS  
> (rcSetup -u)  
> rcSetup Base,2.5.0  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root --filelist MC16cFileList.txt --filelistarea /data/dspiteri/FakeStudy/MC16/  

## (2) ##  
While the MC jobs are running and the Data jobs have finished you can do

> lsetup root  
> mkdir Plots/_F-G_  
> (vim ../NTracksVsMu_datamc_fakerate_v2.C )  

When the MC16c jobs have finished you will most likely have to resetup the release and environment variables. Either way check both directories in the Plots folder you just created have hist-sample.root file so you can run

> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks  
> setupATLAS  
> (lsetup rcSetup)  
> (rcSetup - u)  
> rcSetup Base,2.5.0  
> rc find_packages && rc compile  

It reads in the numerator first and then the denominator, so ensure that the data is the second argument, and the MC is the third.

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_F-G_","2017Data2vMC16c",0)'  

Copy the new directory to a www folder for wasy viewing 
> cp -r Plots/_F-G_/ ~/www/FakeratePlots/PreRec/  
> mv ~/www/FakeratePlots/PreRec/_F-G_/   ~/www/FakeratePlots/PreRec/Data17v3vsMC16c/  

Web access can be found here 
http://dspiteri.web.cern.ch/dspiteri/FakeratePlots/PreRec/Data17v3vsMC16c/[.....]
- NTracksVsMu_datamc_fakerate.pdf
- NTracksVsMu_datamc_fakerate_withPad.pdf
- NTracksVsMu_datamc.pdf

As a comparison you can also run the 2017 and 2016 data on the same plot
> cp ../NTracksVsMu_datamc_fakerate_v2.C ../NTracksVsMu_datamc_fakerate_v3.C  
> mkdir Plots/_A-G_  
> vim ../NTracksVsMu_datamc_fakerate_v3.C  
>   > CHANGE  
>   >   > ...fakerate_v2 -> ...fakerate_v3 (L66)  
>   >   > Data Rel21 -> Data15+Data16 (L191, 192)  
>   >   > Data Rel20p7 -> Data17 (L193, 194)  
>   >   > Rel21/Rel20p7 -> (Data15+Data16)/Data17 (L478, 844)  
>   >   > 45 -> 80 (L314, L320)  
>   >   > 50 -> 80 (L362, L369, L380, L396, L479)           
>   >   > 50.5 -> 80.5 (L426, L505, L542, L580, L653, L769, L781, L865)  

> root -b -l -q '../NTracksVsMu_datamc_fake5ate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-G_","2017Datav2vs2016Data",2)'
  
> cp -r Plots/_A-G_/ ~/www/FakeratePlots/PreRec/  
> mv ~/www/FakeratePlots/PreRec/_A-G_/  ~/www/FakeratePlots/PreRec/Data15+Data16vsData17v2/  


Web access can be found here 
http://dspiteri.web.cern.ch/dspiteri/FakeratePlots/PreRec/Data15+Data16vsData17v2/[plot you want]
- NTracksVsMu_datamc_fakerate.pdf  
- NTracksVsMu_datamc_fakerate_withPad.pdf  
- NTracksVsMu_datamc.pdf  




## Recomendations - OLD WAY  ##
Need to produce the same plots under different ranges of the linear fit to produce recommendations. Otila's way of reproducing recommendations is to repeat the plot generating code with different linear fit ranges and estimate the uncertainty in the variation in the resulting ratio plots.

> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks  
> mkdir Plots/2017MuRangeVariations  
> setupATLAS  
> lsetup root  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/E-_-R21_data-327342-327764-330470/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_E-F_","2017DatavMC16c",0,9,15)'  
> mv Plots/_E-F_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/2017MuRangeVariations/Fakerate9to15.pdf  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/E-_-R21_data-327342-327764-330470/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_E-F_","2017DatavMC16c",0,9,17)'  
> mv Plots/_E-F_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/2017MuRangeVariations/Fakerate9to17.pdf  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/E-_-R21_data-327342-327764-330470/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_E-F_","2017DatavMC16c",0,9,18)'  
> mv Plots/_E-F_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/2017MuRangeVariations/Fakerate9to18.pdf  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/E-_-R21_data-327342-327764-330470/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_E-F_","2017DatavMC16c",0,8,16)'  
> mv Plots/_E-F_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/2017MuRangeVariations/Fakerate8to16.pdf  
 
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/E-_-R21_data-327342-327764-330470/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_E-F_","2017DatavMC16c",0,7,16)'  

> mv Plots/_E-F_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/2017MuRangeVariations/Fakerate7to16.pdf  

> cp -r Plots/2017MuRangeVariations/ ~/www/FakeratePlots/PreRec/   



# Release 21 - Needs Updating #

## Not Required as of yet ##
The first thing that is different between the versions is that version control and code development is done in git. To create a project in git where you will put your analysis code go to
https://gitlab.cern.ch/<your cern username>

and click on "New project". Then need name the project and set a visibility level but it's best if this is kept at the default value: "Private" for now.

For more information go to 
https://twiki.cern.ch/twiki/bin/viewauth/AtlasComputing/SoftwareTutorialAnalysisInGitReleases


> cd ZeroBias_FakeTracksR21  
> mv ZeroBias_FakeTracks/* . && rm -rf ZeroBias_FakeTracks  
> setupATLAS && lsetup panda  

Remove any other build in this area 
> rcSetup -u 

Create new build directory (required for "out of source building")
> mkdir build run

Use https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/AnalysisRelease to ensure that you have the most up to date release and then try to find it.

> howVersions athena | grep AnalysisBase  
> asetup AnalysisBase,21.2.4  
> vim run/gridDataJobSubmit2.sh  

> ADD 
>   > export USER_GRID=dspiteri # at the top of the file
>   >
>   > python ../MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00327342  --inputDS ddata17_13TeV.00327342.physics_ZeroBias.merge.AOD.f832_m1812 --driver grid --suffix _Rel21_data17  
>   > python ../MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00327764  --inputDS data17_13TeV.00327764.physics_ZeroBias.merge.AOD.f836_m1824 --driver grid --suffix _Rel21_data15   
>   > python ../MyVertexAnalysis/scripts/Run.py --submitDir D17_R21_00330470  --inputDS data17_13TeV.00330470.physics_ZeroBias.merge.AOD.f843_m1824 --driver grid --suffix _Rel21_data15  

> chmod u+x run/gridDataJobSubmit2.sh
Use https://twiki.cern.ch/twiki/bin/view/AtlasComputing/SoftwareTutorialAnalysisInGitReleases to create a CMakeLists.txt file and save it in this directory. I actually copied the one that accompanies the athena directory
> cd build  
> cmake ..  
> ../gridDataJobSubmit2.sh  



