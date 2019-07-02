# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about learning how the VHbb CxAOD's Framework operates. #

## "VHbb 2 Lepton Reader" ##
===============================================================================
Last Edited: 25-09-2018
-------------------------------------------------------------------------------
### Up to the subheading 'Running' the instructions here are exactly identical to those found in VHbb_CxAOD_production.md
###

# Setup (now obselete)
## Initial Setup

Such that asetup settings are saved between sessions open up the .asetup file in your home directory (the one that you enter when you log in) in your favorite text editor.
~~~
vim .asetup
~~~
>   ADD under [defaults]
>   > briefprint  = True
>   > autorestore = True

Now we want to create the folder substructure and check out the code.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS &&lsetup git
mkdir CxAODFramework
cd CxAODFramework
mkdir source build run
cd source
~~~
The FrameworkSub contains a script that will check out other needed packages - check out it's master branch
~~~
git clone ssh://git@gitlab.cern.ch:7999/CxAODFramework/FrameworkSub.git
cd FrameworkSub/
git checkout origin/master -b master-${USER} [in keeping with the groups convention]
cd ..
~~~
In the FrameworkSub/bootstrap/ subdirectory there is a new set of scripts for continuing the process of checking out the release-21 development branches of all the framework code. The list of packages to be checked out can be found and edited from the file FrameworkSub/bootstrap/packages_VHbb_git.txt.
~~~
cat ./FrameworkSub/bootstrap/packages_VHbb_git.txt
~~~
To launch the script that checks out the packages run
> source ./FrameworkSub/bootstrap/setup.sh [branch] branch
> source ./FrameworkSub/bootstrap/setup.sh [tag] tag
~~~
source ./FrameworkSub/bootstrap/setup.sh origin/master branch
~~~
Now we need to:
- Choose the analysis base,
- > Found in FrameworkSub/bootstrap/release.txt and can be changed
> cd ../build
> setupATLAS
> lsetup asetup
> For branch or tag
> release=`cat ../source/FrameworkSub/bootstrap/release.txt`
> echo "release=$release"
> asetup AnalysisBase,$release,here
> cp CMakeLists.txt ../source

- Get access to numpy (if you don't already have it)
> lsetup 'lcgenv -p LCG_91 x86_64-slc6-gcc62-opt numpy'

- Compile the repository to create the  "hsg5framework" executable
> cmake ../source
> cmake --build
>   > make -j4   [ if you change the package but DO NOT add a new file. ]

- Set up the compiled environment.
> source x86_64-slc6-gcc62-opt/setup.sh

## Setup Script
Once you are happy with all of these steps there is a script by Adrian Buzatu that runs all of the above, but the commads depend on the release so search https://gitlab.cern.ch/CxAODFramework/FrameworkSub to see what the latest released version is.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
setupATLAS && lsetup git
cp /afs/cern.ch/user/a/abuzatu/work/public/BuzatuAll/BuzatuATLAS/CxAODFramework/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_branch_21.2.23 1 1
> source getMaster.sh r30-02 CxAODFramework_tag_r30-02 1 1 [for a tag]
> source getMaster.sh r31-07 CxAODFramework_tag_r31-07 1 1 [for a tag]
~~~
Then you can test the checkout with the submission of some trial jobs by doing
~~~
source ../source/FrameworkExe/scripts/testLocallyAsInCIPipeline.sh
-   OR   -
source ../source/CxAODOperations_VHbb/scripts/testLocallyAsInCIPipeline.sh
~~~
Then once you get this to work, everytime you log in you can do.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23
source source/FrameworkExe/scripts/setupLocal.sh
~~~
To submit to the grid, from this point it's best to close the terminal used to get the branch, open a new one, cd to the folder that contains the build, source and run directories, and submit in one go with
~~~
source source/FrameworkExe/scripts/submitToGrid.sh
~~~
which will tell you the different arguments it wants.

# Code Architecture
Lets dial it back a bit. We've set up the base and we can now run, but what is it really have we set up? How do we actually DO anything? What are we doing? What did we just run? These are all good questions I currently only partially have an answer to. The answer to these is to start digging and to see what we can find within the things we have installed.

Turning the CxAOD's produced by the maker into analysis plots is the job of the reader.
The C++ code that is run when you type the command hsg5frameworkReadCxAOD in the terminal is hsg5frameworkReadCxAOD.cxx and can be found in source/FrameworkExe/util/hsg5frameworkReadCxAOD.cxx

## hsg5frameworkReadCxAOD.cxx
> std::string submitDir = "submitDir";
This is the name of the target directory created to store output files.

> std::string configPath = "data/FrameworkExe/framework-read.cfg";
This is the string that contains the configuration files that certain variables will be extracted from.

> std::string dataset_dir = config->get<std::string>("dataset_dir");
One of the most important strings obtained from the config file is the directory that contains the data/MC to run over. The config file also supplies the input files, the boolean which controls yield file creation, the analysis channel to run as, and the number of files to run per job.

## framework-read.cfg
The file is 317 lines long and most of it is commented out, which is nice.
-Top Level Settings (L5-L150)
> Contains number of maximum events
> Changes to analysisType variable  [0lep, 1lep 2lep, WBF0ph, WBF1ph, vvqq lvqq]
> Initialisation of switches specific to your analysisType.
> dataset_dir [HIGG5D1 = 0 leptons, HIGG5D2 = 1 lepton, HIGG2D4 = 2 leptons]
> boolean for generating the yields file locally.
> nFilesPerJob and jobSizeLimit
- CxAODReader Settings (L150-L317)
> Booleans for general settings for writing histograms and trees.
> Tagging options, energy corrections and container names
> b-tagging configuration and PU re-weighting
> GRL and advice on what datasets-MC comparisons are sensible
> Systematics
> Turning on/off use of shallow copies of inputs
> Sample names to run over.

## AnalysisReaderVHbb
This file is 1050 lines long and is the part of the reader that you will interact with the most.
It is at this point that I must point out the Reader Structure is more complicated. The framework-read files are configurations of the Reader, and sometimes you will have to go into the readers to understand thing, but there are three levels of the reader.
- AnalysisReader
>   - AnalysisReaderVHbb
>   >   - AnalysisReaderVHbbNLep

The framework-reader executes the AnalysisReaderVHbbNLep.cxx based in source/CxAODReader_VHbb/Root/ . The object that stores histograms is m_histSvc and is filled using ->BookFillHist. You can see this ~L1040. Currently only mvadiboson, mva and mBBMVA objects are being booked into the histogram.
HistSvc.h is where you can find details of this BookFillHist member function of the histogram service. (L45-48)
> void BookFillHist(const string& name, int nbinsx, float xlow, float xup, float value, float weight=1, doSysts ds = doSysts::yes);  //TH1 (1D)

Which all seems reasonable but now you need to find what the {float value} argument actually is so that it can be filled. These are filled from EasyTree which is filled not so far before this so just look there for what you want, and the naming is rather simple. Again once you feel confident you can add your own variables here.

# Trial Running the Framework for 2 Lepton

First thing is to edit the configuration file such that it will work for a 2 lepton analysis run.
~~~
vim ../source/FrameworkExe/data/framework-read.cfg
~~~
>   EDIT the analysisType string
>   > string analysisType           = 0lep -> 2lep.
> CHANGE the directory you point to in search of a dataset.
>   > string dataset_dir = /eos/atlas/atlascerngroupdisk/phys-exotics/CxAOD/CxAOD_30/HIGG5D1_13TeV/CxAOD_00-30-02_a ->
>   > string dataset_dir = /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02/HIGG2D4_13TeV/CxAOD_00-30-02_c
This directory is 416.556 GB in size so moving is rather expensive in terms of space. The directory structure inside, where each subdirectory is either 'data' or a specific MC sample  is what the code looks for to run over. For 2 lepton, HIGG2D4 is the input directory where CxAOD's are stored.
The letter on the end indicates the period of MC run over.
> CHANGE the luminosity which is dependent on the MC type used
>   > float luminosity = 33.26 (MC16c only)
>   CHANGE the number of files you run over per job and the MC period.
>   > int nFilesPerJob              = -1 -> 20
>   > string mcPeriod = mc16c
>   CHECK you have the right GRL
>   COMMENT in the right Syst List & Corrs and Syst List if applicable
>   COMMENT in the samples strings (the test is only ttbar_nonallhad)

## Running
To test that it works out of the box, try the following command from the run directory, with the results in a directory called submitDir. You will want to change this so that you can keep your runs
~~~
cd ../run
hsg5frameworkReadCxAOD
mv submitDir test_NoCuts
~~~

## Auto-Running
The process of finding the correct things to edit can be combersome and you may miss important things that you need to do when you submit a job, but lucky for you this process has been automated by Adrian Buzatu! All of the variables that you need to change have variables names that take input.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23
~~~
To calibrate this though, you need to look over the file submitReader.sh.
~~~
vim source/FrameworkExe/scripts/submitReasher.sh
~~~
The top of the file tells you how to use the file and what inputs you need to insert, and the 6 arguments go like this:
> INPUT_FOLDER=$1
> OUTPUT_FOLDER=$2
> CHANNELs=$3
> MCTYPEs=$4
> VTAGs=$5
> DO_EXECUTE=$6

Lets, for example, run the framework on the latest tag with only the 2017 data and compare it to MC16c. The up to date framework-read.cfg in FrameworkExe/data has the repositories so you just have to select the correct one. Alternatively you can check here (https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/HiggsbbCxAODproduction) and this should be up to date

>   > string dataset_dir = /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02/HIGG2D4_13TeV/CxAOD_00-30-02_c

In this example dataset directory, the repository path names after "VH" contains the tag, then the channel with the CoM energy, then the MC subcampaign (a = MC16a, c = MC16c, d = MC16d). If you look into the folder you see a load of kinds of MC16c samples (and a data17 directory). You will run over all of these folders named in the vector<string> SAMPLES. To test if this is working only run over one or two small samples (lets try data and SR).

> COMMENT OUT
>   > SAMPLES="data15 data16 data17 qqZvvHbbJ_PwPy8MINLO ggZvvHbb_PwPy8   qqZllHbbJ_PwPy8MINLO ggZllHbb_PwPy8 qqWlvHbbJ_PwPy8MINLO WZ_Sh221 ttbar_nonallhad_PwPy8"
> ADD IN
>   > SAMPLES="data17 ggZH125"

To get a better idea of what folders you should run over in general, use Andrew Bell's talk
https://indico.cern.ch/event/713324/contributions/2930684/attachments/1617793/2572691/MC_Sample_Overview.pdf or talk to people working on your channel.

~~~
vim source/FrameworkExe/data/framework-read-automatic.cfg
~~~
The next thing to do is to edit the vector<string> TMVATrainingTool.InputVarNames to match those used in the 2 Lepton analysis and I will re-arrange the order so that they appear in the same order they are listed as variables in the makePlots.cxx file. You can make your own but they have to edit the makePlots2Lepton.cxx accordingly.

>   CHANGE (~L80)
>   > dEtaWH -> dEtaVBB
>   > dPhiVBB -> dPhiVBB
>   > dPhiLBmin -> mLL
>   > Mtop -> mBBJ
>   > pTV mBB pTB1 pTB2 mTW MET dRBB dPhiVBB mLL MBBJ dEtaVBB -> pTV mLL MET dRBB mBB pTB1 pTB2 pTJ3 mBBJ dPhiVBB dEtaVBB

> COMMENT IN + CHANGE
>   > string bQueue              = 1nh-> 8nh #for LSF driver ~L91
> CHANGE  CxAODReader settings
>   > bool fillCr = true -> false L139
>   >

Next ensure that the variables that point to relative paths in the directory ${WorkDir_DIR} and ${CONFIG_INITIAL_STEM} are pointing to the right place/exists at all.

You also have the option of running both locally with "direct" or on the grid with "condor" (now defunct) or "LSF". You will probably want to first test that the jobs successfully run locally first with options like
NUMBEROFEVENTS="1000000" (signal region is small)
DRIVER="direct"
### ALWAYS TEST YOUR JOBS LOCALLY FIRST.

~~~
vim source/CxAODReader_VHbb/Root/AnalysisReader_VHbb2Lep.cxx
~~~
So we need the following histograms filled:  pTV mLL MET dRBB mBB pTB1 pTB2 pTJ3 mBBJ dPhiVBB dEtaVBB

Now you need to know some physics because you have to estimate the fine-ness of your binning and acceptable ranges for these variables. (makePlots2Lepton.cxx can help with the ranges.)

>   ADD ~(L1053) more histograms to plot INSIDE if(m_histNameSvc->get_description() != ""){}.
>   > m_histSvc->BookFillHist("pTV", 20, 0, 500, m_MVAInputs2l->pTV, m_weight);
>   > m_histSvc->BookFillHist("mLL", 20, 80, 100, m_MVAInputs2l->mLL, m_weight);
>   > m_histSvc->BookFillHist("MET", 25,0, 500, m_MVAInputs2l->MET, m_weight);
>   > m_histSvc->BookFillHist("dRBB", 20, 0, 5, m_MVAInputs2l->dRBB, m_weight);
>   > m_histSvc->BookFillHist("mBB", 25, 0., 500., m_MVAInputs2l->mBB, m_weight);
>   > m_histSvc->BookFillHist("pTB1", 50, 0., 500., m_MVAInputs2l->pTB1, m_weight);
>   > m_histSvc->BookFillHist("pTB2", 50, 0., 500., m_MVAInputs2l->pTB2, m_weight);
>   > m_histSvc->BookFillHist("pTJ3", 50, 0., 500., m_MVAInputs2l->pTJ3, m_weight);
>   > m_histSvc->BookFillHist("mBBJ", 15, 0, 750, m_MVAInputs2l->mBBJ, m_weight);
>   > m_histSvc->BookFillHist("dPhiVBB", 15, 1.5, 3.15, m_MVAInputs2l->dPhiVBB, m_weight);
>   > m_histSvc->BookFillHist("dEtaVBB", 20, 0, 5, m_MVAInputs2l->dEtaVBB, m_weight);

Every time you edit this reader file you have to go into your build directory and re-make the framework
~~~
cd build
setupATLAS
make -j6
~~~
Now it's time to run over this configuration. In order to run this then I would type this into the command like from where I am (the run directory).
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23
source source/FrameworkExe/scripts/setupLocal.sh
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02 TEST 2L c 00-30-02 1
~~~
If this small local test works then we can run on the grid. To do so we run the same commands above but we change the following flags in the submitReader.sh file.
NUMBEROFEVENTS="-1"
DRIVER="LSF"
To check on the jobs that you have submitted on the grid, write in the terminal
~~~
bjobs
~~~
Where you should see this type of output
JOBID     USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
155042852 dspiter RUN   8nh        lxplus015.c b6b828bb99  *mit/run 0 Mar 26 18:44
155042853 dspiter RUN   8nh        lxplus015.c b6b828bb99  *mit/run 1 Mar 26 18:44

When this is done the output logs are found in submit/LSFJOB_JOBID/STDOUT, and the output root files are found in the fetch directory. The data directory is one of the largest samples. If you don't get an output root file then it may be because you are using too much memory on a single node. If this is the case you can try a few things

1) Setting the number for NRFILESPERJOB in sumbitReader.sh instead of  JOBSIZELIMITMB,  but this means that you have to set  JOBSIZELIMITMB  to "-1". The first one of these to be written JOBSIZELIMITMB overwrite all successive initialisations of the other variable.
2) Trying a different driver.

If this works then you are ready to run the reader over everything
- 1) Obtain correct sample directory from https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/HiggsbbCxAODproduction.
[/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02/HIGG2D4_13TeV/]
[/eos/atlas/atlascerngroupdisk/phys-exotics/CxAOD/CxAOD_31/HIGG2D4_13TeV/]
- 2) Pick the data and MC types you want to run over
[Data17 - MC16c (MC16d in later versions)]

~~~
vim /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/source/FrameworkExe/data/submitReader.sh
~~~
> CHANGE
>   > NUMBEROFEVENTS="-1"
>   > DRIVER="LSF" ~L52
>   > SAMPLES="data17 ggZvvHbb_PwPy8 ggZllHbb_PwPy8 ggWW_Sh222 ggZZ_Sh222 qqZvvHbbJ_PwPy8MINLO qqZllHbbJ_PwPy8MINLO qqWlvHbbJ_PwPy8MINLO stops_PwPy stopWt_PwPy ttbar_nonallhad_PwPy8 WenuB_Sh221 WenuC_Sh221 WenuL_Sh221 Wenu_Sh221 WmunuB_Sh221 WmunuC_Sh221 WmunuL_Sh221 Wmunu_Sh221 WtaunuB_Sh221WtaunuC_Sh221 WtaunuL_Sh221 Wtaunu_Sh221 WW_Sh221 WZ_Sh221 ZZ_Sh221 ZmumuB_Sh221 ZmumuC_Sh221 ZmumuL_Sh221 Zmumu_Sh221 ZnunuB_Sh221 ZnunuC_Sh221 ZnunuL_Sh221 Znunu_Sh221 ZtautauB_Sh221 ZtautauC_Sh221 ZtautauL_Sh221 Ztautau_Sh221" ~L69

Then assuming that you are all set up, you run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02 R30-02 2L c 00-30-02 1

OR

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23_NEW/run
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-exotics/CxAOD/CxAOD_31 R31-01-2 2L d 31-01 1
~~~
### WARNING
Some errors come about due to the use of old versions of the reader. Make sure that you are relatively up to date when you run these commands.

When you do update you have to ensure that the inputs in the following files are correct (relative to source/FrameworkExe )
- scripts/submitReader.sh
- data/framework-read-automatic.cfg
- ../CXAODReader_VHbb/Root/AnalysisReader_VHbb2Lep.cxx

Lots of tools auto-hadd the plots but a lot of them run into problems. So it might be best to do it yourself.
~~~
/afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/R30-02/Reader_2L_MVA_00-30-02_c/fetch

hadd DATA17.root hist-data17-*
hadd GGZH125.root hist-ggZH125-*
hadd STOP_WT.root hist-singletop_Wt-*
hadd TTBAR_1PLUSLEP.root hist-ttbar_nonallhad_A14-*
hadd TTH.root hist-ttH-*
hadd WENUB.root hist-WenuB_v221-*
hadd WENUC.root hist-WenuC_v221-*
hadd WENUL.root hist-WenuL_v221-*
hadd WMUNUB.root hist-WmunuB_v221-*
hadd WMUNUC.root hist-WmunuC_v221-*
hadd WMUNUL.root hist-WmunuL_v221-*
hadd WTAUNUB.root hist-WtaunuB_v221-*
hadd WTAUNUC.root hist-WtaunuC_v221-*
hadd WTAUNUL.root hist-WtaunuL_v221-*
hadd WW.root  hist-WW-*
hadd WZ.root  hist-WZ-*
hadd ZZ.root  hist-WZ-*
hadd ZEEB.root  hist-ZeeB_v221-*
hadd ZEEC.root  hist-ZeeC_v221-*
hadd ZEEL.root  hist-ZeeL_v221-*
hadd ZMUMUB.root  hist-ZmumuB_v221-*
hadd ZMUMUC.root  hist-ZmumuC_v221-*
hadd ZMUMUL.root  hist-ZmumuL_v221-*
hadd ZTAUMTAUB.root  hist-ZtautauB_v221-*
hadd ZTAUMTAUC.root  hist-ZtautauC_v221-*
hadd ZTAUMTAUL.root  hist-ZtautauL_v221-*
hadd ZNUNUB.root  hist-ZnunuB_v221-*
hadd ZNUNUC.root  hist-ZnunuC_v221-*
hadd ZNUNUL.root  hist-ZnunuL_v221-*
~~~
Now just in case this does not work it's best to also hadd all these into one file.
~~~
hadd 2LEPALL.root DATA17.root GGZH125.root STOP_WT.root TTBAR_1PLUSLEP.root TTH.root WENUB.root WENUC.root WENUL.root WMUNUB.root WMUNUC.root WMUNUL.root WTAUNUB.root WTAUNUC.root WTAUNUL.root WW.root WZ.root ZZ.root ZEEB.root ZEEC.root ZEEL.root ZMUMUB.root ZMUMUC.root ZMUMUL.root ZTAUMTAUB.root ZTAUMTAUC.root ZTAUMTAUL.root ZNUNUB.root ZNUNUC.root ZNUNUL.root
~~~

# Plotting
In order to see what you have run, the root files created using the reader need to be turned into pretty plots.
There are many ways of doing this but I shall go through two.

## PlottingTool
This is the official package recommended by the framework. First you need to get it if you don't already have it.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/source/FrameworkExe/macros
setupATLAS && lsetup git
git clone ssh://git@gitlab.cern.ch:7999/CxAODFramework/PlottingTool.git
git clone ssh://git@gitlab.cern.ch:7999/CxAODFramework/TransformTool.git
rm -f TransformTool/CMakeLists.txt
cd PlottingTool && lsetup root
~~~
The plotting tool works with some scripts already present within the framework. The script makePlots2Lepton.cxx (INSIDE THE PLOTTING TOOL) is the code to produce generic plots. Then you need to check the run instructions in the source/FrameworkExe/macros/runCxAODPlots.cxx file. And you shouldn't need to compile, the root execution command for that file does that. Lets open the file and change some things so that it fits the 2-lepton stuff and only worry about running over one file for now.
~~~
vim runCxAODPlots.cxx
~~~
> ADD
>   >  TString libdir="core/"; (L17)
>   > TString LocalAreaPath="/afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run"; (L20)

> CHANGE
>   > InputPath = LocalAreaPath+"run/TEST/Reader_2L_MVA_00-30-02_c/fetch"; (L20)
>   > InputFileName = "hist-ggZH125-0.root"; (L21)
>   > Output -> Input (L40 & L42)
>   > ...Framework/macros/makePlots0Lepton.cxx.... -> ...makePlots2Lepton.cxx... (L41)


> COMMENT OUT
>   > if(do_hadd) loop (L36-38)

So this script would usually hadd the root files within a target directory and then put this through makePlots2Lepton.cxx, but we have temporarily disabled the feature to see if we can get the tool to run.

The most common output error from the plotting tool come from errors in the jetBins, tagBins and pTVBins vector collections. You will need to open up a sample root file and see what tag,jet and pTV collections are stored.
> ggZvvH125_0tag2jet_75_150ptv_SR_mBBJ
~~~
root -l TEST/Reader_2L_MVA_00-30-02_c/fetch/hist-ggZH125-0.root
.ls *0tag2jet*
~~~
### WARNING
Release 30-02 had an issue with the trigger matching.

The next thing to do is to open up the makePlots2Lepton.cxx file and include the variables, background and signal MC samples and the data samples that you want to run over; Edit the output path you put the plots into; and calibrate the types of binning you want. The best way to ensure that you get the correct things on your first try is open the output root files in a TBrowser and see what is in the file.
~~~
vim makePlots2Lepton.cxx
~~~

So the first 60 or so lines are the code, are general configuration and at the moment doesn't need to be changed. So the main things in this file that need to be edited are the samples, variables and regions that get run over to create histograms.

> CHANGE
>   > "PlottingTool/Config.h" -> "core/Config.h" (L1)
>   > "PlottingTool/PlotMaker.h" -> "core/PlotMaker.h" (L2)
> ADD signal sample to
>   > config.addSignalSample("ggZvvH125",  "VH 125", kRed); (L71)
>   > config.addSignalSample("ggZllH125",  "VH 125", kRed); (L72)

> CHANGE jet,tag and pTV details (to what you saw in the ROOT file)
>   > ("4jet","4 jet")) -> ("4pjet","4+ jet")); (L201)
>   >   jetBins.push_back(make_pair("5pjet","5+ jet")); -> DELETE (L202)
>   >   jetBins.push_back(make_pair("0pjet", "0+ jet")); -> ADD (L199)
>   > "_0_150ptv_" -> "_0_75ptv_" (L211)
>   > "_150_200ptv_" -> "_75_150ptv_" (L212)
>   > "_200_500ptv_" -> "_150ptv_" (L213)
>   > pTVBins.push_back("_500ptv_"); -> DELETE (L214)
CHANGE titles of merged PtV bins
>   > string pTVBinName -> string pTVBinNameIncl (L216)
>   > string pTVBinTitle -> string pTVBinTitleIncl (L217)
> CHANGE pTVBinName -> pTVBinNameIncl & pTVBinTitle -> pTVBinTitleIncl
>   > (L243 , L246, L248, L260)

> COMMENT OUT/IN
>   > all the background samples and the data sample (for now)
>   > all the BDTInput Variables that do not feature in the vector<string> TMVATrainingTool.InputVarNames found in the framework-reader(-automatic).cfg (~L125-L185)
>   CHANGE for dPhiVBB and dEtaVBB
>   > dPhiVH -> dPhiVBB  (L160 - all instances of H -> BB)
>   > dEtaVH -> dEtaVBB (L161 - all instances of H -> BB)

Since you may want to delete and reinstall the PlottingTool and TransformTool packages, you may want make these changes into the files of the same names in the 'macros' folder and then copy them into the tool such that they replace the ones in the package.

Lets try running this now. The code does not have to be pre-compiled, it was designed to be used with ROOT interactively. To run the tool, in the main PlottingTool package directory simply type .
~~~
cp ../runCxAODPlots.cxx .
cp ../makePlots2Lepton.cxx .
root -b -q runCxAODPlots.cxx > dryrun.log
~~~
### Debugging
Most errors that come from the plotting tool are down to the fact that the histogram you are expected to run over are not in the output ROOT file. This means that somewhere either the histogram naming service has slipped up, or you aren't looking in the right regions in the makePlots2Lepton.cxx

The first thing to do is to open up the ROOT file and check that, the name of the Root file a) follows the correct convention, and b) is all there. If either of these is the case, then you have to follow up in the region of the AnalysisReader_VHbb2Lep.cxx which sets that part of the name.

e.g.1 One of the first errors that I has was the region part of the plots were missing. I went to the region section of the reader, but a few cout statements and found that the DiLeptonCuts::Trigger in the common cuts were failing, which meant that no regions were being labelled. ASked around, and it turned out this was a problem with the 00-30-02 tag that I was using.

>   > if ( passSpecificCuts(eventFlag, cuts_common_resolved) ) std::cout<<"COMMON CUTS PASSED"<<endl;
>   > else std::cout<<"COMMON CUTS FAILED"<<endl;
>   > if ( !passSpecificCuts(eventFlag, cuts_common_resolved) ) {
>   > if (passSpecificCuts(eventFlag, {DiLeptonCuts::LeadLepPt27}) ) { std::cout<<"CUT 1/6 PASSED"<<endl; }
>   > if (passSpecificCuts(eventFlag, {DiLeptonCuts::Trigger}) ) { std::cout<<"CUT 2/6 PASSED"<<endl; }
>   > if (passSpecificCuts(eventFlag, {DiLeptonCuts::OSLeptons}) ) { std::cout<<"CUT 3/6 PASSED"<<endl; }
>   > if (passSpecificCuts(eventFlag, {DiLeptonCuts::AtLeast2Jets}) ) { std::cout<<"CUT 4/6 PASSED"<<endl; }
>   > if (passSpecificCuts(eventFlag, {DiLeptonCuts::AtLeast2SigJets}) ) { std::cout<<"CUT 5/6 PASSED"<<endl; }
>   > if (passSpecificCuts(eventFlag, {DiLeptonCuts::PtB145}) ) { std::cout<<"CUT 6/6 PASSED"<<endl; }
>   > }

“ if (passSpecificCuts(eventFlag, {DiLeptonCuts::Trigger}) )” returning false includes both the trigger firing AND trigger matching. This calls AnalysisReader_VHbb2Lep::pass2LepTrigger, which calls TriggerTool::getDecisionAndScaleFactor, which calls TriggerTool::getPassedTriggers which contains the trigger matching. This is a general error so the file that needs to be looked at is in general tools
/source/CxAODTools/CxAODTools/TriggerTool.h. L39 is where you disable the trigger matching.

e.g.2 After I had repaired the SR region. The topemucr region was also not showing up. I thought this was due to the same issue. i.e that the regions were failing the cuts. However there were histograms showing up with empty regions and were being filled. I looked near the end of the 2 Lepton reader and I realised that there were histograms being booked regardless of whether the region was being filled. Once these histograms were only being created if the region was declares, histograms with the topemucr tag were showing up and being filled.

## DynPlottingTool
Plotting tool is not robust in the sense that you have to open up root files and ensure that what is being plotted is only what is in the file or it breaks. DynPlottingTool solves this by just plotting what is in the output root file and not having to search through and edit files every time.

DynPlottingTool is best when initialised OUTSIDE the directory, such that it can be independent of releases. PlottingTool can't do this and has to be present within the file structure of the framework, meaning it needs to be freshly obtained everytime you get a new copy/version of the directory.

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git
git clone ssh://git@gitlab.cern.ch:7999/sjiggins/DynPlottingTool.git
~~~
Before use, the files need a spucing up in terms of the numbers that are hard-coded to be shown on the final plots
~~~
vim DynPlottingTool/src/Plotmaker.cxx
~~~
> CHANGE the luminosity
36.1 -> 43.8 (L301)
>   CHANGE the channel
WH #rightarrow l#nubb, -> ZH #rightarrow llbb (L554)

Now make the package
~~~
cd DynPlottingTool
lsetup root
make clean
make -j6
~~~
The input of the DynPlottingTool are the output root files created by the reader
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/TEST/Reader_2L_MVA_00-30-02_c
vim lofinputs.txt
~~~
> ADD
>   > /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/TEST/Reader_2L_MVA_00-30-02_c/hist-ggZH125.root
>   > /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/TEST/Reader_2L_MVA_00-30-02_c/hist-data17.root

The DynPlottingTool is then run using one command but it takes a series of inputs, the order of which doesn't matter. Below is a list of some of the more important ones.
~~~
./PlotMaker -h (for help)
~~~
>   -x systematics on or off
>   -l listoffiles
>   -w wsmaker format (obselete)
>   -f signal scale factor (normalise cross section to 1pb because you don't know if it's real - this f is the cross section of the process as predicted by the standard model.)
>   -a/-q (obselete)
>   -s signal (currently can only put 1 sample)
>   -e signal(s) present in file but you don't want to plot
>   -o output directory
>   -v variable, can give comma seperated list.
>    > "/" after it is a re-binning factor
>    > "-" changes the x-axis range


Here is an example running command.
~~~
./PlotMaker -x false -l /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/TEST/Reader_2L_MVA_00-30-02_c/lofinputs.txt -a SM -q SM -d data -s ggZHll125 -w 1 -f 1.0 -v mBB/4-400 -o VHbb2Lep_ggZHll125_data17-TEST -b 1 -e ggZvvH125 2>&1 | tee VHbb2Lep_DYN-TEST.log
~~~

Whenever you want to run the plotting tool ensure that:

- You edit the/a lofinputs.txt file to contain the root files you want to run over
- You have the correct axis labels
- Separate different plot titles using commas in the convention "name[/X][-Y]"
    /X rebins by a factor X and -Y makes Y the new maximum
~~~
./PlotMaker -x false -l /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/R30-02/Reader_2L_MVA_00-30-02_c/lofinputs.txt -a SM -q SM -d data -s ggZH125 -w 1 -f 1.0 -v mBB/4-400,pTV/1-500,mLL/1-110,MET,dRBB/4,pTB1/2-400,pTB2/2-400,pTJ3/4,mBBJ/4,dPhiVBB,dEtaVBB -o VHbb2Lep_ggZH125_data17 -b 1 -e ggZvvH125 2>&1 | tee VHbb2Lep_DYN.log
~~~

Utils.cxx and Plotmaker.cxx can be used to edit the plot. ~L2384 is where the formatting for the 2tag2jet_75_150ptv_SR_mBB.png is done but since this is a git repository don't forget to create a new branch.
~~~
git checkout -b master_plotPretty upstream/master --no-track
~~~

### Adding to Plotmaker.cxx in DynPlottingTool
In Plotting.cxx, there may be be the option you want in the framework. If you want to add your own option to this you need to do three structural things
1) Add a printout statement to what your option will do in int Usage()
-  std::cout << "--VHbbBlinding   (-b)  : Adds VHbb blinding Scheme to plot outputs" << std::endl; (L89)

2) Add an variable for this printout (initialised if needed)
- bool VHbbBlinding = false; (L124)

3) Add option to option struct
- { "VHbbBlinding" , required_argument , 0 , 'b'}, (L134)

4) Add the flag letter to the case string "l:d:s:v:f:b:C:c:F:D:w:B:S:R:k:e:x:T:p:P:n:o:h"
- "l:d:s:v:f:C:c:F:D:w:B:S:R:k:e:x:T:p:P:n:o:h" -> "l:d:s:v:f:b:C:c:F:D:w:B:S:R:k:e:x:T:p:P:n:o:h" (L157)

5) Add overwriting of (initialised) flag variable is the user calls it in the command line execution for the tool
- case 'b':
- VHbbBlinding = ( std::atoi(optarg) != 0 ) ; (for a boolean)
- std::cout << "<PlotMaker()>:: VHbbBlinding = " << VHbbBlinding << std::endl;
- break;

6) Now you just need to code whatever your the addition of this flag is meant to do.
In this case it's to design a void VHbb2018BlindingScheme(std::string & variable, std::string & region, TH1 *data, TH1 *signal, TH1 *bkg) in Utils.cxx, and declare it in Utils.h, and call it in the makePlots function within Plotmaker.cxx.

7) NEW SUBMISSION
~~~
./PlotMaker -x false -l /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/TEST/Reader_2L_MVA_00-30-02_c/lofinputs.txt -B SM -S SM -d data -s ggZHll125 -w 1 -f 1.0 -v mBB/4-400 -o VHbb2Lep_ggZHll125_data17-TEST -D 1 -e ggZvvH125 2>&1 -b true | tee VHbb2Lep_DYN-TEST.log
~~~

# Cutflow Challenge

One of the things under the jurisdiction of VHbb analysis NTuples activities is to check the number of events in the cutflow of the analysis (See https://twiki.cern.ch/twiki/bin/view/AtlasProtected/HiggsbbCxAODproduction) . Basically you run the reader over a standard sample.

For 2 lepton R30, the directories to run over are found on the spreadsheets that circulate within the group.
~~~
/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02/HIGG2D4_13TeV/CxAOD_00-30-02_a/ZH125J_MINLO/group.phys-higgs.mc16_13TeV.345055.PwPy8EG_NNPDF3_AZNLO_ZH125J_MINLO_llbb_VpT.r9364.HIGG2D4.30-02_CxAOD.root

/eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02/HIGG2D4_13TeV/CxAOD_00-30-02_a/data16/group.phys-higgs.data16_13TeV.00300800.physics_Main.HIGG2D4.30-02_CxAOD.root
~~~
Edit the file that steers the Reader
~~~
vim /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/source/FrameworkExe/data/submitReader.sh
~~~
> CHANGE variables
>   > NUMBEROFEVENTS="-1" ~L48
>   > DRIVER="LSF" ~L52
>   > NRFILESPERJOB="50" ~L56
>   > SAMPLES="data16 ZH125J_MINLO" ~L68

Some metadata for the analysis. It is performed on MC16a, using the tag 30-02. Now running the below commands should successfully run the cutflow over the grid.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23
source source/FrameworkExe/scripts/setupLocal.sh
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/180116_r30-02/ Cutflow 2L a 00-30-02 1
~~~
If this has successfully run. There should be histgrams in the Cutflow/Reader_2L_MVA_00-30-02_a/fetch/ directory. The next time you want to then all that is needed to be done is for these to be hadd-ed. Then once the hadd-ed files are complete they can be opened in the TBrowser and the numbers in the Cutflow-> Nominal -> CutsMVANoWeight are the cutflow results.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.23/run/Cutflow/Reader_2L_MVA_00-30-02_a/fetch/
setupATLAS && lsetup root
hadd ZH125J_MINLO-CUTFLOW.root hist-ZH125J*
hadd data16-CUTFLOW.root hist-data16*
root -l ZH125J_MINLO-CUTFLOW.root
root -l data16-CUTFLOW.root
 -> TBrowser a
~~~

