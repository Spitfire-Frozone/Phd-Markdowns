# This document walks the user through the steps required to run one (and multiple) ESD file(s) on the grid on lxplus.# 

## "Running ESD jobs on the grid with lxplus" ##
===============================================================================
Last Edited: 07-04-2017
-------------------------------------------------------------------------------

# Getting a single ESD running.

Open two windows in the same lxplus node (i.e. lxplus027@cern.ch) to stop 
possibilities of conflicts between the python version of hte athena release and
with the one in dq2 or panda

###(1)###
Create new directory to store created variables SEPERATE to the DCube one

> mkdir ESDforGrid
> cd ESDforGrid

Set up ATLAS infrastructure, set up Rucio and Athena, set up ATLAS VO and change to the Athena directory. Build $PYTHONPATH correctly

> setupATLAS 
> lsetup rucio
> voms-proxy-init -voms atlas
> asetup 21.0.14,here #[MAKE SURE THIS IS A RELEVANT SETUP]
>  > From 07/04/17 you can do this by typing 
>  >  > cd validation
>  >  > source ~/Generic/dwayneGenericSetup.sh 

Download an ESD file of your choosing. A finished job from http://bigpanda.cern.ch/datasetInfo/?datasetname=valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.ESD.e4993_s2958_r8727_tid09826097_00&jeditaskid=9826097 -> Show files in dataset

You will also need the athena version that set up this job. At the top of the first link, note the r tag on the dataset (r8727), then type this in to the search box at https://ami.in2p3.fr/app?subapp=amiTags_show and find the athena release -SWReleaseCache- which in this case is AtlasOffline_21.0.7

Rename the downloaded folder to the name of the dataset
> rucio download valid2:ESD.09826097._000011.pool.root.1
> mv valid2 valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.ESD.e4993_s2958_r8727_tid09826097_00

Pull out Bill Murrays public directory to your $TestArea and rename it

> cp -R /afs/cern.ch/user/m/murray/public/validation-ref/ .
> mv /validation-ref /validation

The source.sh files and checkout.sh files need to be edited for local runs
From 07/04/17 the following steps are unnessecary if this is not your first time. For repeats the file to check, edit and source is  'dwayneValidationSetup.sh'

> cd validation
> vim xats.sh
    > CHANGE AtlasProduction,20.1.3.2,gcc48 ->AtlasOffline21.0.7,gcc49 #(or whatever the athena release was on ami)(line 33)
    
See how many processors are available to you

> nproc

Checkout.sh seems to want CMakeLists.txt in it's own source folder, which it  won't be if you have completed this as this is generated in your TestArea which should be the validation folder.

> vim checkout.sh
>  >  ADD cp ../../CMakeLists.txt . #below the line 'cd source' (line 7)
>  >  CHANGE cd build -> cd ../build #(line15) build is not in that folder
>  >  CHANGE make -> make -j[result of nproc] (line16)

###(2)###
Go into the validation folder and source the xats.sh and checkout.sh.

> cd ~/LOCALPATHTOEDSGRID/ESDforGrid/validation
> source xats.sh #This just seems to setup a particular version of athena
> source checkout.sh #checks out the tracking validation software and compiles it

If you are not doing this for the first time and checkout.sh repeatedly fails, it's probably because something in build directory is giving you trouble. If you delete the folder and checkout checkout.sh again, it should work


###(1)###
making a local test with validation_21.py but a symbolic link is needed to be created to the downloaded ESD file in order to allow athena to load the metadata. 
From 11/04/17 the following steps are unnessecary if this is not your first time. For repeats the file to check, edit and source is  'dwayneValidationLocalTest.sh "${1}" '

> ln -s ../valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.ESD.e4993_s2958_r8727_tid09826097_00/*.root.* ESD.pool.root
athena Validation_21.py

###(2)###
Need to set up cmake which is done in checkout.sh
Prepare the jobs to go on the grid by setting up panda 

From 12/04/17 the rest of the text to the end of the document has now 
been superceded if this is not your first time
>  > source dwayneGridSubmission.sh "${1}" "${2}" "${3}" "${4}" "${5}"

> source gridSetup.sh

If you run this and you get the error message: User's request for VOMS
attributes could not be fulfilled, or other errors.

> vim gridSetup.sh
>    
>  >    CHANGE localSetupPandaClient -> lsetup panda (L4)
>  >    CHANGE atlas:/atlas/perf-idtracking/Role=production -> atlas (L5)
>  >  > This will not be need to be done if on this page https://lcg-voms2.cern.ch:8443/voms/atlas/user/home.action under 'Your groups and roles' you are in the group /atlas/perf-idtracking and have the 'production' role. If you do not have these, then these need to be requested. Else do the change as instructed to above.
>  >    CHANGE source/ -> build/ (L10)
    
###(1)###
Open up the run_21 folder and make edits, saving it under a different name You submit the jobs with the Role perf-idtracking/Role=production in order  to have high priority in the samples, but this priveledge may not have been  awarded yet. The dataset-name of the output must start with: group.perf-idtrackig.yourlabel.XXXX. The dataset-name of the input could be either the whole container if available or a  particular tid

> vim run_21
    
>   >  hange /tmp/${USER} -> ~/LOCALPATHTOESDGRID/ESDforGrid #local path to downloaded file (L15-16)
>   >  ELETE --official # after pathena - creates an offial dataset (L36)
>   >  HANGE atlas:/atlas/perf-idtracking/Role=production -> atlas 
>   >   > Same as before, the change is only needed if you don't have membership in #particular groups on the VOMS page
>   >  HANGE group.perf-idtracking.010916.t0.NopTcut -> user.dspiteri.[DDMMYY].test (e.g 180117.test -naming convension upon submission )
>   >  DD domain=`echo $1 |awk -F"." '{print$1}'` #(insert under L32)
>   >  DD echo $domain #(insert under L34)
>   >  HANGE --inDS=$1 -> --inDS=$domain:$1
>   >  Save as run_21_Something
  
Turn your newly created tun file to an executable  
  
> chmod +x run_21_Something    
###(2)###
Now you are ready to submit the job to the grid

> ./run_21_Something valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.ESD.e4993_s2958_r8727_tid09826097_00 

If this was submitted successfully you will get an email with the task number of the  job whether it succeeds or fails. If the job is a success, it should take about 5-7 hours to run on the grid and you should get two output containers, one with plots and one with the run log.(http://bigpanda.cern.ch/task/10527637/)

If the grid job is successful and you get an email with a workable output,and you require the space, then delete the downloaded directory

> rm -r valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.ESD.e4993_s2958_r8727_tid09826097_00


