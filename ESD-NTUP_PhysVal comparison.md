# This document walks the user through the steps required to compare output Ntuples produces through different .# 

## "ESD-NTUP_Physval Object Comparison" ##

Last Edited: 31-01-2017
-------------------------------------------------------------------------------
### Continues on from "Running_ESD's_on_the_grid_on_lxplus.md" ### 


You are trying to compare whether the ESD DCube web output made from the Grid job is the same as the physVal_make_web_display for the same file and for the physVal_make_web_display for the NTUPLES that correspond to the grid job. Hence here the test and reference files in all cases should be the same. 
~~~
cd ~/PATHTO/validation #if you are not there already
setupATLAS 
lsetup rucio
voms-proxy-init -voms atlas
asetup 21.0.14,here
~~~
These steps are required every time you log in. You need to be in the validation folder.

Download the  performance plots from the grid to a directory of your choosing in validation. I choose the date of the grid job finishing and a run tag. The performance plots can be found under the header "Output Containers" from the panda website link you get in the email.
~~~
mkdir 200117test
cd 200117test
rucio download user.dspiteri.200117.test2.e4993_s2958_r8727_tid09826097_00_InDetPerformanceMon/
~~~
Rename the downloaded folder to something more manageable like ESD
~~~
mv user.dspiteri.200117.test2.e4993_s2958_r8727_tid09826097_00_InDetPerformanceMon/ ESD/
cd ESD
~~~
Now merge the output files that you moved into this current folder using hadd. Create a unique output file name
Syntax: hadd selectednameofoutputfile listofinputfiles
~~~
hadd InDetStandardPlots-tid09826097_00.root *.root*
~~~
>   *.root will catch all files with the extension .root. Make sure you want all of the root files to be included.
TProfile histograms cannot be simply added with hadd due to them being summary plots. You will have to download a 1463 line c++ macro that will reprocess these. While not necessary for comparison it should be obtained if you want a TProfile histogram is displayed, though this is not usually the case
~~~
cp /afs/cern.ch/user/m/murray/public/validation-ref/reprocessHistos.cxx .
~~~
Takes all of the files in the directory that have the '.root.' string and reprocessess their TProfile histograms
~~~
root -l -q *.root.* reprocessHistos.cxx+
~~~
Now for dcube. Go to your dcube web display area and setup up a new working area. If you haven't already set it up then you need to follow the instructions here: <https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/InDetPhysValMonitoring>
~~~
cd ~/www/dcube
setupATLAS 
asetup 20.7.5.1,here #DCube only works with cmt which is only available to athena v20
mkdir tid09826097_00 #or anything that identifies the comparison
cd tid09826097_00
~~~
Copy the hadd-ed output file to this area
~~~
cp /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/200117test/ESD/InDetStandardPlots-tid09826097_00.root .
~~~
Set up the DCube configuration and create an XML config file for DCube to use
~~~
svn co svn+ssh://svn.cern.ch/reps/atlasoff/Tools/DCubeServer/tags/DCubeServer-00-00-17 DCubeServer
pkgco.py Tools/DCubeClient 
dcube.py -g -r ./InDetStandardPlots-tid09826097_00.root -c dcube_MyNewConfigFile.xml
~~~
------from the twiki page------
>  Note that upon running this command, the config file dcube_MyNewConfigFile.xml that is produced is the place where you specify (1) which histograms to compare and (2) from which file do you derive the reference histograms. When you create this file, these are specified to be from the file you input, in this case [InDetStandardPlots-tid09826097_00.root]. You can now use this config file as a guide for the production of a DCube page by running the command below. This should be executed in the same dcube directory you are currently situated in.
--------------
~~~
dcube.py -c dcube_MyNewConfigFile.xml -p -s /afs/cern.ch/user/d/dspiteri/www/dcube/tid09826097_00/DCubeServer/server/ ./InDetStandardPlots-tid09826097_00.root 
~~~
Note here in the last command "InDetStandardPlots-tid09826097_00.root" is the MONITORED or TEST file go to your physval make web display area and create a physval web display using the hadd-ed .root file as the testfile, the tid tag as the name of the output, and the itself as the reference file physval_make_web_display.py --reffile ${refname}://${reffile} --outdir=$PWD --title "${testname}" ${testfile} --startpath=IDPerformanceMon. If you have not set up physVal_make_web_display then read the file
"Getting physval_make_web_display to work.md"
~~~
cd ~/www/physVal_make_web_display

setupATLAS 
asetup 21.0.18,here
mkdir tid09826097_00 #same as the one in in the equivalent one you are comparing in the DCube directory
cd tid09826097_00
physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/200117test/ESD/InDetStandardPlots-tid09826097_00.root --outdir=$PWD --title tid09826097 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/200117test/ESD/InDetStandardPlots-tid09826097_00.root --startpath=IDPerformanceMon
~~~
Again this is an internal comparison of the same thing. If you don't want this just change the second root file path to one that points to a different file.
This is the physVal_make_web_display for the ESD data output, but in actual fact there are seperately generated NTUP_PHYSVAL objects generated from these ESD's that need to be separately downloaded and hadd-ed and then put into physval_make_web_display

- 1) Go to Grid job output page: http://bigpanda.cern.ch/task/10527637/
- 2) Click on the dataset hyperlink in 'Input containters'
- 3) Click the input task hyperlink number near the top of the page
- 4) in 'datasets,show/hide by type' click the dropdown menu output()
- 5) If this has NTUP_PHYSVAL in it's name check it has the same e,s,r tags as your dataset and if it does these are the NTUPS you are looking for.

Following a similar set of commands to before
~~~
cd ~/PhysVal/ESDforGrid/Validation/200117test
setupATLAS
lsetup rucio
voms-proxy-init -voms atlas

rucio download valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.merge.NTUP_PHYSVAL.e4993_s2958_r8727_p2870_tid09826102_00 #Dataset you found in (5)
mv valid2.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.merge.NTUP_PHYSVAL.e4993_s2958_r8727_p2870_tid09826102_00 NTUP
cd NTUP/

hadd InDetStandardPlots-tid09826102_00.root *.root.1
cp ../ESD/reprocessHistos.cxx .
root -l -q *_PHYSVAL.* reprocessHistos.cxx+
 
cd ~/www/physVal_make_web_display
setupATLAS 
asetup 21.0.14,here
mkdir tid09826102_00
cd tid09826102_00
~~~
If you're comparing different files, which happens more often than not, it's probably best to use the r-tags/p-tags of the compared NTuples to name the directory like r9049vsr9067
~~~
physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/200117test/NTUP/InDetStandardPlots-tid09826102_00.root --outdir=$PWD --title tid09826102 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/200117test/NTUP/InDetStandardPlots-tid09826102_00.root --startpath=IDPerformanceMon
~~~
Web access can be found at:
- https://dspiteri.web.cern.ch/dspiteri/physVal_make_web_display/tid09826097_00/
- https://dspiteri.web.cern.ch/dspiteri/physVal_make_web_display/tid09826102_00/
- https://dspiteri.web.cern.ch/dspiteri/dcube/tid09826097_00/InDetStandardPlots-tid09826097_00.root.dcube.xml.php

## Final commands for comparisons

----------------------------------------------------------r9049vsr9067 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/160217ref/NTUP/InDetStandardPlots-tid10586625_00.root --outdir=$PWD --title r9049vsr9067 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/160217test/NTUP/InDetStandardPlots-tid10552924_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9049vsr9171 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317refs/NTUP049/InDetStandardPlots-tid10552924_00.root --outdir=$PWD --title r9049vsr9171 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317tests/NTUP171/InDetStandardPlots-tid10818258_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9171vsr9202 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317refs/NTUP171/InDetStandardPlots-tid10818258_00.root --outdir=$PWD --title r9171vsr9202 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317tests/NTUP202/InDetStandardPlots-tid10893190_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9198vsr9195 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317ref/NTUP198/InDetStandardPlots-tid10892925_00.root --outdir=$PWD --title r9198vsr9195 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317tests/NTUP195/InDetStandardPlots-tid10892834_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9198vsr9196 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317ref/NTUP198/InDetStandardPlots-tid10892925_00.root --outdir=$PWD --title r9198vsr9196 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317tests/NTUP196/InDetStandardPlots-tid10892842_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9049vsr9200 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317refs/NTUP049/InDetStandardPlots-tid10552924_00.root --outdir=$PWD --title r9049vsr9200 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/070317tests/NTUP200/InDetStandardPlots-tid10893092_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9273vsr9274-p3086 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317refs/NTUP273_086/InDetStandardPlots-tid11046035_00.root --outdir=$PWD --title r9273vsr9274-p3086 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317tests/NTUP274_086/InDetStandardPlots-tid11046038_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9273vsr9275-p3086 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317refs/NTUP273_086/InDetStandardPlots-tid11046035_00.root --outdir=$PWD --title r9273vsr9275-p3086 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317tests/NTUP275_086/InDetStandardPlots-tid11065641_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9273vsr9274-p3087 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317refs/NTUP273_087/InDetStandardPlots-tid11027513_00.root --outdir=$PWD --title r9273vsr9274-p3087 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317tests/NTUP274_087/InDetStandardPlots-tid11027525_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9273vsr9275-p3087 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317refs/NTUP273_087/InDetStandardPlots-tid11027513_00.root --outdir=$PWD --title r9273vsr9275-p3087 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/290317tests/NTUP275_087/InDetStandardPlots-tid11046032_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9299vsr9300 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/030417ref/NTUP299/InDetStandardPlots-tid11058575_00.root --outdir=$PWD --title r9299vsr9300 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/030417tests/NTUP300/InDetStandardPlots-tid11058584_00.root --startpath=IDPerformanceMon

----------------------------------------------------------r9299vsr9301 data comparison
> physval_make_web_display.py --reffile reference:///afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/030417ref/NTUP299/InDetStandardPlots-tid11058575_00.root --outdir=$PWD --title r9299vsr9301 /afs/cern.ch/user/d/dspiteri/PhysVal/ESDforGrid/validation/030417tests/NTUP301/InDetStandardPlots-tid11058590_00.root --startpath=IDPerformanceMon
