# This will be aimed at my the second part of my qualification task which is to join the efforts surrounding the study of fake rates. The first aim is to try and replicate Otilia's plots. # 

## R21 vs R20.7 in Data and R21 MC16 vs R20.7 MC15  ##
===============================================================================
Last Edited: 13-11-2017
-------------------------------------------------------------------------------
Indeed, the note (https://cds.cern.ch/record/2110140/files/ATL-PHYS-PUB-2015-051.pdf) explains what we need to measure and presents the methodology. It takes a bit of time to understand the existing framework and run the code for the first time. For more context on why this is being done see p97-p100 of PhD Book 1. The commands used to run the code in several ways can be found here.

>/afs/cern.ch/work/o/oducu/public/ZeroBias_FakeTracks/MyVertexAnalysis/run/command_grid


One of the commands looks like this 
> python MyVertexAnalysis/scripts/Run.py --submitDir 302393  --inputDS data16_13TeV:data16_13TeV.00302393.physics_ZeroBias.merge.AOD.f711_m1620 --driver grid --suffix _Rel2op7v2

So it makes sense to dig around the Run.py file to see what it is doing. The first thing that it required is that a copy of the ZeroBias_FakeTracks folder is needed somewhere in your work directory.

> [old] cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks  
> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy

> vim Run.py  

See PhD Book 1 p96 for details of whats in Run.py. For context on why this is happening see p97-p100 of PhD Book 1

> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks  
> setupATLAS && lsetup panda  
> rcSetup Base,2.4.31  

If you are going to try and do a local run. There no need to set up for DQ2,
Panda and Grid.

# Submission Options in the Run.py file #
>  ---------------
> | --submitDir             | Directory to store the output , default="submit_dir"
> | --inputDS               | input dataset from DQ2, default="none"
> | --input                 | Input directory or file (use with --driver=direct)"
> | --driver                | select where to run, choices=("direct", "LSF", "prooflite","grid"), default="direct" (local)]  
> | --nevents               | type=int, Number of events to process for all the datasets
> | --skip-events           | type=int, Skips the first n events]  
> | --overwrite or -w,      | action='store_true', Overwrites the previous submitDir ,default=False 
> | --ntuplein              | type=str, Reads in an ntuple and make histograms
> | --filelist              | type=str, Reads in a list of files
> | --suffix                | type=str, Suffix to distinguish different grid   submissions apart default="v1"
> | --z0cut                 | type=float, Cut on track z0 on the histograms, default=999.
> | --ptcut                 | type=float, Cut on track pT on the histograms (GeV),default=0.
> | --dataScaleFactor       | type=float,Scale Factor to Actual Mu,default=1.
>  ---------------


Now to submit some of the Rel21 (Rel20.7) runs that Otilia used to the grid

> [Data Samples]
>  ---------------
> | data15_13TeV.00276689.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.r7562_p2634 |  
> | data15_13TeV.00276731.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.r7600_p2634 |  
> | data15_13TeV.00276778.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.f620_m1480  |  
>  ---------------                                         |                       |  
> | data16_13TeV.00297730.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.f694_m1583  |  
> | data16_13TeV.00299288.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.f698_m1594  |  
> | data16_13TeV.00299584.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.f703_m1600  |  
> | data16_13TeV.00307259.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.f739_m1667  |  
> | data16_13TeV.00310574.physics_ZeroBias.recon.AOD.r9264 | merge.AOD.f756_m1704  |  
>  ---------------

> [MC Samples]
>  ---------------
> |   T = 13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711  
>  ---------------
> | Rel21: mc16_T_s3126_r9425_r9315  
> | Rel20.7: mc15_T_s2576_s2132_r8205_r7676  
>  ---------------


Select from the data grid submission commands at the bottom of this document those that you want to submit to the grid and paste them into a .sh file that you will make. The MC ones will be done by hand for now, using the commands listed at the bottom of this document as well.

> setupATLAS  
> vim gridDataJobSubmit.sh  

>  > ADD   
>  >   > export USER_GRID=dspiteri   
>  >   > source rcSetup.sh   
>  >   > rc find_packages && rc compile    
>  >     
>  > ADD     
>  >   > python MyVertexAnalysis/scripts/Run.py [...]    
    
> chmod u+x gridDataJobSubmit.sh  
> ./gridDataJobSubmit.sh  

Do the MC jobs here as well. The commands are at the bottom of this document

Once all the jobs have been sumbitted, move all the directories created upon submission to a single folder and create a new folder for the grid jobs to download. The MC jobs will take longer than the data jobs.

> mkdir jobSubmissionFolders && cd jobSubmissionFolders   
> mkdir 07-08-17 && cd ../..   
> mv D15* D16* MC15* MC16* jobSubmissionFolders/07-08-17    
>   > These folders contain root samples that you can supposedly run over, but they have always been empty for me, so there is no point in keeping them.  

Make a .txt file with all of the output containers that the grid has made for your jobs that have not fucked up. The best place to find all your jobs is http://bigpanda.cern.ch/tasks/?username=dwayne%20spiteri

You may find it easier to open multiple terminals at this point. In one of them you should go to a place where you have lots of storage be it /eos/user/ or some server from your institution. I did not but I ended up moving them over later.

> mkdir gridOutput && cd gridOutput  
> setupATLAS && lsetup rucio  
> voms-proxy-init -voms atlas  
> vim gridOutputContainers.txt  

     > ADD user.dspiteri.00307259.physics_ZeroBias._Rel20p7_data16_myOutput.root/  
     > ADD user.dspiteri.00299288.physics_ZeroBias._Rel20p7_data16_myOutput.root/  
     > ADD user.dspiteri.00299584.physics_ZeroBias._Rel20p7_data16_myOutput.root/  
     > [...]  

> rucio download `cat gridOutputContainers.txt`  

Once this is done it may be a good idea to organise the output better  

> mkdir 07-08-17 && mv user* gridOutputContainers.txt 07-08-17   
> cd 07-08-17  
> mkdir Rel20p7 Rel21  
> mv *_Rel20p7* Rel20p7 && mv *_Rel21* Rel21  

Check if the output files are useful and if not just delete them, but keep the folders to remind you what samples were used

> cd D15_R20p7_00276731 && rm -rf * && cd ..  
> cd D15_R20p7_00276778 && rm -rf * && cd ..  
> cd D15_R21_00276778 && rm -rf * && cd ..  

...e.t.c for all the folders in this subdirectory
   
# For both R20.7 and R21:
hadd the ntuples before turning them into histograms (saves future issues) with TProfile histrgram hadd-ing.Do this for all sub-folders

> cd Rel21  
> cd user.dspiteri.00276689.physics_ZeroBias._Rel21_data15_myOutput.root  
> hadd run276689.root *dspiteri*  
> rm *dspiteri*  
> mv run276689.root ..  
> cd ../user.dspiteri[...]  

Then once that is done. Remove all of the empty folders 
> rm -rf user*

hadd these to get one Release21 dataset. 
> hadd R21_data-276689-276731-276778-299288-299584-307259-310574.root run*

Move the data to a place where you have more space or get rid of the data used to make this
> ssh -Y glasvr02  
> cd /data/dspiteri  
> mkdir FakeStudy && cd FakeStudy  
> mkdir R21 R20p7 MC_R20p7  
> exit  
> mv run* glasvr02:/data/dspiteri/FakeStudy/R21  
    
Now for the MC. This is a big file so you will need to do this in an area that has lots of space. You probably want to keep all the files there. If the files add up to larger that 100GB then you will have to read in a list of files.

> rucio download user.dspiteri.159000.ParticleGenerator_nu_E50._Rel20p7_MC15_myOutput.root  
> mv user.dspiteri.159000.ParticleGenerator_nu_E50._Rel20p7_MC15_myOutput.root MC15_Rel20p7   
> mv MC15_Rel20p7 07-08-17   
> cd MC15_Rel20p7  
> hadd 159000.ParGen_nu_E50.root *dspiteri*  
> mv run* glasvr02:/data/dspiteri/FakeStudy/MC_R20p7  
   
The next phase is to take the (hadd-ed) ntuples and produce histograms of them 
for each data(MC)set. From here on out you want to ensure that you're working somewhere with space (/eos/ mounted home directory, remote institution server e.t.c) -- glasvr02 for me. 

> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/  
> mkdir Plots  

> setupATLAS  
> source rcSetup.sh  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574  --ntuplein gridOutput/07-08-17/Rel21/R21_data-276689-276731-276778-299288-299584-307259-310574.root  

> rcSetup -u  
> rcSetup Base,2.4.31  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/B-_-R20p7_data-276689-276731-276778-299288-299584-307259-310574  --ntuplein gridOutput/07-08-17/Rel20p7/R20p7_data-276689-276731-276778-299288-299584-307259-310574.root  

> rcSetup -u  
> rcSetup Base,2.4.31  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/C-_-R20p7_mc15-159000.ParGen_nu_E50.root  --ntuplein gridOutput/07-08-17/MC15_Rel20p7/159000.ParGen_nu_E50.root  

> rcSetup -u   
> rcSetup Base,2.5.1  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root  --ntuplein /data/dspiteri/FakeStudy/MC_R21/159000.ParGen_nu_E50.root.part  

If you had any problem with these commands or generating the hadded ntuples in the first place then you will repeat these commands with a list of files (or smaller hadd-ed objects that have the form [].root) instead. I edited the Run.py to add the parser options --filelistarea.   

So instead of hadding the files you run over them in a file. The file needs to be in the same area as the files you select to run over. The full path needs to be specified within the text file. I could try to use the --filelistarea to change such that the sampleholder appends the information to the file but I couldn't get it to work.

> rcSetup -u  
> source rcSetup.sh  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574  --filelist R21HaddDataFileList.txt --filelistarea /data/dspiteri/FakeStudy/R21  

> rcSetup -u  
> rcSetup Base,2.4.31  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/B-_-R20p7_data-276689-276731-276778-299288-299584-307259-310574  --filelist R20p7IndivDataFileList.txt --filelistarea /data/dspiteri/FakeStudy/R20p7/  
 
> rcSetup -u 
> rcSetup Base,2.4.31  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/C-_-R20p7_mc15-159000.ParGen_nu_E50.root  --filelist 20p7MCFileList.txt --filelistarea /data/dspiteri/FakeStudy/MC_R20p7/  

> rcSetup -u  
> rcSetup Base,2.5.1  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root --filelist MC16_R21FileList.txt --filelistarea /data/dspiteri/FakeStudy/MC_R21/  

Once you have generated plots from MC15, MC16, and R20p7 and R21 data. You then need to produce pretty plots using NTracksVsMu_datamc_fakerate_v2.C

> void NTracksVsMu_datamc_fakerate_v2(int fitType = 1,
>   > TString data   = "[PATH TO DATA]",  
>   > TString mc     = "[PATH TO MC]",  
>   > TString outdir = "Output", (needs to already exist)  
>   > TString outname= "test",   
>   > Int isDataData=0,   
>   >   > ( 0 - R21 vs MC16)   
>   >   > ( 1 - R20.7 vs MC16)   
>   >   > ( 2 - R21 vs R20p7)   
>   >   > ( 3 - MC15 vs MC16)   

>   > double fitmin=9,  
>   > double fitmax=17) {   
nominal: 9-16;  (9-15, 9-17, 9-18, 8-16, 7-16)

> lsetup root  
> cd Plots  
> mkdir _A-D_ _B-C_ _A-B_ _C-D_  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-D_","R21vMC16",0)'  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/B-_-R20p7_data-276689-276731-276778-299288-299584-307259-310574/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/C-_-R20p7_mc15-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_B-C_","R20p7vsMC15",1)'  
 
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/B-_-R20p7_data-276689-276731-276778-299288-299584-307259-310574/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-B_","R21vR20p7",2)'  

> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/C-_-R20p7_mc15-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_C-D_","MC15vMC16",3)'  

If this works you can either check the results in gimp or move the files to a web folder to view them.

> cp -r Plots/_A-D_/ ~/www/FakeratePlots/PreRec/  
> cp -r Plots/_B-C_/ ~/www/FakeratePlots/PreRec/  
> cp -r Plots/_A-B_/ ~/www/FakeratePlots/PreRec/  
> cp -r Plots/_C-D_/ ~/www/FakeratePlots/PreRec/  
> mv ~/www/FakeratePlots/PreRec/_A-D_/ ~/www/FakeratePlots/PreRec/R21vsMC16/   
> mv ~/www/FakeratePlots/PreRec/_B-C_/ ~/www/FakeratePlots/PreRec/R20p7vsMC15/  
> mv ~/www/FakeratePlots/PreRec/_A-B_/ ~/www/FakeratePlots/PreRec/R21vsR20p7/  
> mv ~/www/FakeratePlots/PreRec/_C-D_/ ~/www/FakeratePlots/PreRec/MC16vsMC15/  

Further edits to the NTracksVsMu_datamc_fakerate.C may have to be made, such like the tidying up of a ratio segment to the fakerate with pad plot. 

# Troubleshooting: Aside

If you are still getting problems at this point then go back to the drawing board. Try one type of file at a time and see if you can actually produce plots. In the ZeroBias_FakeTracks directory

> mkdir Plots/Testplots  
> source rcSetup.sh  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/Testplots/A-_-R21_data  --ntuplein ./A.21.root  
> 
> rcSetup -u  
> rcSetup Base,2.4.31  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/Testplots/B-_-R20p7_data --ntuplein ./B.20p7.root  
> 
> rcSetup -u  
> rcSetup Base,2.4.31  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/Testplots/C-_-R20p7_mc15  --ntuplein ./C.MC15.root  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/Testplots/C-_-R20p7_mc15_Large --ntuplein/data/dspiteri/FakeStudy/MC_R20p7/20p7LargeSample.root  
>     
> rcSetup -u  
> rcSetup Base,2.5.1  
> rc find_packages && rc compile  
> python MyVertexAnalysis/scripts/Run.py --submitDir Plots/Testplots/D-_-R21_mc16  --ntuplein ./D.MC16.root  

> lsetup root  
root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/Testplots/A-_-R21_data/hist-..root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/Testplots/D-_-R21_mc16/hist-..root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/Testplots/","test",0)'                      


# Reproducing Recommendations [OLD WAY]

Need to produce the same plots under different ranges of the linear fit to reproduce Otilias recommendations.

> mkdir Plots/R21MuRangeVariations  
>   
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-D_","R21vMC16",0,9,15)'  
> mv Plots/_A-D_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/R21MuRangeVariations/Fakerate9to15.pdf  
> 
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-D_","R21vMC16",0,9,17)'  
> mv Plots/_A-D_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/R21MuRangeVariations/Fakerate9to17.pdf  
> 
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-D_","R21vMC16",0,9,18)'  
> mv Plots/_A-D_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/R21MuRangeVariations/Fakerate9to18.pdf  
> 
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-D_","R21vMC16",0,8,16)'  
> mv Plots/_A-D_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/R21MuRangeVariations/Fakerate8to16.pdf  
> 
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/A-_-R21_data-276689-276731-276778-299288-299584-307259-310574/hist-Rel21.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/D-_-R21_mc16-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/Plots/_A-D_","R21vMC16",0,7,16)'  
> mv Plots/_A-D_/NTracksVsMu_datamc_fakerate_withPad.pdf Plots/R21MuRangeVariations/Fakerate7to16.pdf  
> 
> cp -r Plots/R21MuRangeVariations/ ~/www/FakeratePlots/PreRec/  

Once you are happy with all this and the plots work delete the contents of the
gridOutput directory to save space. Also remove any directories that have the raw files from the grid.

> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/ZeroBias_FakeTracks/  
> rm -rf gridOutput  


# GRID SUBMISSION COMMANDS for Otilia's work.
-------------------------------------------------------------------------------
> [DATA_15 R21]  
> python MyVertexAnalysis/scripts/Run.py --submitDir D15_R21_00276689  --inputDS data15_13TeV:data15_13TeV.00276689.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data15
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D15_R21_00276731  --inputDS data15_13TeV:data15_13TeV.00276731.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data15
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D15_R21_00276778  --inputDS data15_13TeV:data15_13TeV.00276778.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data15
> 
> [DATA_16 R21]   
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R21_00297730 --inputDS data16_13TeV:data16_13TeV.00297730.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R21_00299288 --inputDS data16_13TeV:data16_13TeV.00299288.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R21_00299584 --inputDS data16_13TeV:data16_13TeV.00299584.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R21_00307259 --inputDS data16_13TeV:data16_13TeV.00307259.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R21_00310574 --inputDS data16_13TeV:data16_13TeV.00310574.physics_ZeroBias.recon.AOD.r9264 --driver grid --suffix _Rel21_data16
> 
> [DATA_15 R20.7]  
> python MyVertexAnalysis/scripts/Run.py --submitDir D15_R20p7_00276689 --inputDS data15_13TeV:data15_13TeV.00276689.physics_ZeroBias.merge.AOD.r7562_p2634 --driver grid --suffix _Rel20p7_data15
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D15_R20p7_00276731 --inputDS data15_13TeV:data15_13TeV.00276731.physics_ZeroBias.merge.AOD.r7600_p2634 --driver grid --suffix _Rel20p7_data15
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D15_R20p7_00276778 --inputDS data15_13TeV:data15_13TeV.00276778.physics_ZeroBias.merge.AOD.f620_m1480 --driver grid --suffix _Rel20p7_data15
> 
> 
> [DATA_16 R20.7]   
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R20p7_00297730 --inputDS data16_13TeV:data16_13TeV.00297730.physics_ZeroBias.merge.AOD.f694_m1583 --driver grid --suffix _Rel20p7_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R20p7_00299288 --inputDS data16_13TeV:data16_13TeV.00299288.physics_ZeroBias.merge.AOD.f698_m1594 --driver grid --suffix _Rel20p7_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R20p7_00299584 --inputDS data16_13TeV:data16_13TeV.00299584.physics_ZeroBias.merge.AOD.f703_m1600 --driver grid --suffix _Rel20p7_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R20p7_00307259 --inputDS data16_13TeV:data16_13TeV.00307259.physics_ZeroBias.merge.AOD.f739_m1667 --driver grid --suffix _Rel20p7_data16
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir D16_R20p7_00310574 --inputDS data16_13TeV:data16_13TeV.00310574.physics_ZeroBias.merge.AOD.f756_m1704 --driver grid --suffix _Rel20p7_data16 
> 
> [MC_15 R20.7]   
> export USER_GRID=dspiteri  
> rcSetup -u  
> rcSetup Base,2.4.36  
> rc find_packages && rc compile  
> rc clean  
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir MC15_R20p7_00159000 --inputDS mc15_13TeV:mc15_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s2576_s2132_r8205_r7676 --driver grid --suffix _Rel20p7_MC15  
> 
> [MC_16 R21]   
> export USER_GRID=dspiteri  
> (rcSetup -u) # if another setup has been done before  
> rcSetup Base,2.5.1  
> rc find_packages && rc compile  
> rc clean   
> 
> python MyVertexAnalysis/scripts/Run.py --submitDir MC16_R21_00159000_3 --inputDS mc16_13TeV:mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315 --driver grid --suffix _Rel21_MC16_3  

