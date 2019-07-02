# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about generating and creating VHbb CxAOD's and testing them. # 

## "VHbb CxAOD production" ##
===============================================================================
Last Edited: 21-02-2018
-------------------------------------------------------------------------------
### Useful Twiki: https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/HiggsbbCxAODproduction
https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/Release21Migration
###

# Setup
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
Once you are happy with all of these steps there is a script by Adrian Buzatu that runs all of the above, but the commads depend on the release so run
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
cat CxAODFramework/source/FrameworkSub/bootstrap/release.txt
cp /afs/cern.ch/user/a/abuzatu/work/public/BuzatuAll/BuzatuATLAS/CxAODFramework/getMaster.sh .
source getMaster.sh origin/master CxAODFramework_branch_21.2.19_1 1 1
> source getMaster.sh r30-02 CxAODFramework_tag_21.2.19_1 1 1 [for a tag]
~~~
Then once you get this to work, everytime you log in you can do.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb
cd CxAODFramework_branch_21.2.19_1
source source/FrameworkExe/scripts/setupLocal.sh
~~~

## Running
To test that it works out of the box, try the following command from the run directory, with the results in a directory called submitDir
~~~
cd ../run
hsg5framework
~~~

# Code Architecture
Lets dial it back a bit. We've set up the base and we can now run, but what is it really have we set up? How do we actually DO anything? What are we doing? What did we just run? These are all good questions I currently only partially have an answer to. The answer to these is to start digging and to see what we can find within the things we have installed.

Producing CxAOD's is the job of the maker, not many people eventually do this and if the analysis you are running os for a milestone, then only the production team will run the maker. To edit the job options for the maker you want to edit it's config file

~~~
cd ../source/
vim FrameworkExe/data/framework-run-2L-a.cfg
~~~

The file is 360 lines long and most of it is commented out, which is nice.
-Top Level Settings (L1-L12)
> Contains number of maximum events
- Sample Lists (L13-133)
> Lists of different paths to .txt sample files for the three different channels
> Event Selection variables
> Good Run List selection
> MC-Data samples comparison selection
> Output settings
> Debugging Options
- CxAODMaker Settings (L133-L326)
> Flags for applying settings to events (e.g useTausInMET = false)
> Container name processessing
> Trigger handling
> Jet Configuration variables.
- Grid Settings (L327-L359)
> Has some flags to be turned on/off and some variables to be initialised such that you can run on the grid.
> They all seem pretty explanitory

## framework-run.cfg (from VER 23 onwards)
The file is 385 lines long and most of it is commented out, which is nice. It is vastly cleaner and more ordered than the previous version.
-Top Level Settings (L5-L13)
> Contains number of maximum events
-Sample Lists (L13 - L149)
> Example Samples for local testing
> Sample Lists with DxAOD from the grid, to run on the grid - ordered by channel
>   >  [HIGG5D1 = 0 leptons, HIGG5D2 = 1 lepton, HIGG2D4 = 2 leptons]
> Changes to analysisType variable  [0lep, 1lep 2lep, WBF0ph, WBF1ph, vvqq lvqq]
> Selection of GRL and advice on what datasets-MC comparisons are sensible.
> Selection of Pile-Up Reweighting (PRW)
> submition Directory naming
- CxAODReader Settings (L149-L352)
> Booleans for general settings for writing histograms and trees.
> Tagging options, energy corrections and container names
> b-tagging configuration
> Trigger Options
> Systematics
> Turning on/off use of shallow copies of inputs
-Grid Settings (L352-L385)
> version of submission and task options
> (boolean for generating the yields file locally. - only in V19)
> nFilesPerJob (and jobSizeLimit - only in V19)
