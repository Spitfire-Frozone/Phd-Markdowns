# This will be aimed at giving the reader the basic skills to work and manage their work within the git architecture.
=================================================
## Last Edited: 03-03-2020
-------------------------------------------------

### KEYWORDS/JARGON

ORIGIN - The fork you make of a repository
UPSTREAM - The main repository
___________________________
# 0) Creating a git repository
---------------------------

You will only have to do this if you are working on something else separate to a common analysis framework. To create a project in your personal repository, you can go to https://gitlab.cern.ch/<your cern username>.
Then click on "New project" and give it a title and a privacy setting - default is "private"
~~~
setupATLAS && lsetup git
git clone https://:@gitlab.cern.ch:8443/dspiteri/atlas-trackingCP-fakestudy.git
cd atlas-trackingCP-fakestudy
~~~

Make a README.md file for users of this package
~~~
vim README.md
git add README.md
git commit -m "adds the README"
git push -u origin master
~~~

Once this is done you are ready to move changes to git. I do this on the master branch
~~~
git add [files]
git commit -m "[adds files]"
git push -u origin master
~~~
___________________________
# 1) Setup
---------------------------

~~~
cd Code_Development/ATHENA
source dwayneGitSetup.sh
~~~
Only if it's your first time, but you really should get things when you  need them or things may not work. lsetup rucio shouldn't be attempted yet as the athena command makes 'rucio download' commands not work.
~~~
setupATLAS
lsetup git
~~~
_________________________________
# 2) Forking the ATHENA repository
----------------------------------
## based on https://atlassoftwaredocs.web.cern.ch/gittutorial/gitlab-fork/

Go to https://gitlab.cern.ch/atlas/athena and press 'Fork'. Click your ugly mug to fork the project to your user account. Wait for it to sucessfully fork then go to Settings -> Members and then in the 'Add a new member to athena box' type 'atlasbot', and beneath select 'Developer' and then select 'Add to Project'.

Your development will be built and tested by an automated continuous integration system to make sure your code doesn't fuck up anything upon submission. Results of these automatic builds are published on GitLab as small icons next to commit hashes. In order for this to work, the build bot needs to have access to your private fork. 

________________________________
# 3) Local Cloning 
-----------------------------------------
## based on https://atlassoftwaredocs.web.cern.ch/gittutorial/git-clone/ ##

The GitLab fork you just created is used primarily for sharing your changes
with others. To actually make a change you need a local copy that you can edit 
yourself. In git this is done by cloning YOUR FORK. This is done in either of 
two ways.
A__a full checkout: Gives you access to working copies of all files. It’s great if you want to look at many files or substantial parts of the repository (it’s  the standard way that things would normally go into git, but more setup is required).
B__a sparse checkout: Gives you only the parts of the repository you want to update. It’s great if you know you only want to make a limited set of changes and want to limit the space used by your working copy.

____________________
### This will cover option A
--------------------------
~~~
git clone https://:@gitlab.cern.ch:8443/dspiteri/athena.git
~~~
When code is developed it should begin from the current HEAD version of the main repository. So we add the main repository as a remote repository, called upstream
~~~
cd athena
git remote add upstream https://:@gitlab.cern.ch:8443/atlas/athena.git #or any other valid URL
~~~

If you wanted to target some changes already present on a directory then you can clone the branch like so, but if you are beginner, you should really be working with the head because it should the base that all other codes are defined off of and should be guarenteed to build.
~~~
git clone -b <branch> <remote_repository>
~~~

### This will cover option B
--------------------------
This will automatically use your forked version, so before you do this it is important that you have your own personal fork of athena
~~~
git atlas init-workdir https://:@gitlab.cern.ch:8443/atlas/athena.git
cd athena
git atlas addpkg <packagenames> #Don't need full path, just package directory
~~~
_________________________________
# 4) Creating a Topic Branch to store changes
---------------------------------------------

~~~
git fetch upstream #Ensures local repository is up to date with the main one
git checkout -b master_PVWD-edit upstream/master --no-track
git checkout -b [branch to merge to]-[name] upstream/[parent_branch] --no-track
~~~
To find out what parent branch to choose go pack to the main project page (https://gitlab.cern.ch/dspiteri/athena in this case) and select one of the options under the 'Branch' heading in the table.

>   git branch : with no arguments lists the branches available and the one you are on with an asterisk.
>   git branch -d branch_name : removes a local branch. Bear in mind you can't delete a branch you are currently on

You can also target an already existing branch to store your changes. This option creates a new branch. 
~~~
git checkout -b [local_name] [origin-OR-upstream/originorupstreamnameofbranch]
~~~
Though sometimes you will want to add changes and NOT create a new branch, but commit to the same branch. Fot this you will want to track the branch upstream. 
~~~
git fetch origin
git checkout --track origin/<remote_branch_name>
~~~

Explicitly set up the Development release for release 21 (21.0 branch).
~~~
asetup 21.0.20,Athena,gcc62
asetup master,r28,Athena,gcc62 for master branch (OR for master branch)
~~~

This asetup command will setup a runtime environment for the gcc62 version  of the master branch build of Athena that was built on the second of this month. Ensure that this is as up to date as possible. The latest one can be obtained from https://atlassoftwaredocs.web.cern.ch/athena/athena-nightly-builds/setting-up-a-nightly-release

To rebuild part of the release (called a sparse build), take a local copy of Projects/WorkDir/package_filters.txt and edit it to list only the packages you want to build.


Develop code in athena/Projects/Workdir but make sure you change packages in
the packages you downloaded.

> cd Projects/WorkDir

If you want to pull a branch that already exists and store your changes on that then you need.
~~~
git checkout master
git checkout --track -b [target branch]
~~~

Some of the time you will want to work not from the master, but the latest stable version or a particular past version. These versions of the code are called tags. If you are required to work from them, you can get the list of tags *once you have cloned the repository in the normal fashion* via the command.
~~~
git fetch --all --tags
~~~
To start work on the tag all you simply do is
~~~
git checkout tags/<tag> 
~~~
But this command will put you on a branch name which is the same as the tag. If you want to develop code this is not ideal and you can create your own topic branch at the same time. 
~~~
git checkout tags/<tag> -b <branch>
~~~
_____________________________________ 
# 5) Setting up to compile and test code
-------------------------------------
Once you have developed the code in athena/Projects/Workdir you need to build it
 
> Full Checkout - option A 
---------------------------
~~~
mkdir ../../../build && cd ../../../build
cp ../athena/Projects/WorkDir/package_filters_example.txt ../package_filters.txt
vim ../package_filters.txt
~~~
>     -ADD- +DataQuality/DataQualityUtils/
> >  Add paths that start from the root of the cloned repository that go to the directory that contains the folder and no further.
~~~
cmake -DATLAS_PACKAGE_FILTER_FILE=../package_filters.txt ../athena/Projects/WorkDir
make -j
~~~

> Sparse checkout - option B
----------------------------
~~~
mkdir ../../../build && cd ../../../build
cmake ../athena/Projects/WorkDir #This folder needs to have CMakeLists.txt
make -j6
~~~
__________________________________ 
# 6) Local testing
----------------------------------
First make the run directory and get some data somehow.
~~~
mkdir ../run && cd ../run
lsetup rucio
voms-proxy-init -voms atlas
rucio download --nrandom=1 mc16_valid:mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00
ln -s mc16_valid.410000.PowhegPythiaEvtGen_P2012_ttbar_hdamp172p5_nonallhad.recon.AOD.e3698_s2995_r9273_tid11027507_00/<*TAB*> AOD.pool.root

source ../build/x86_64-slc6-gcc62-opt/setup.sh
athena ../athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/run/PhysVal_jobOptions.py
~~~
DO ANY ADDITIONAL TESTS AT THIS POINT

After this you need to perform standardised tests on the release
~~~
cd ../build
rm -rf (*)
setupATLAS && asetup 21.0,Athena,r04
cmake ../athena/Projects/WorkDir
make -j6
cd ../run
source ../build/x86_64-slc6-gcc62-opt/setup.sh
RunTier0Tests.py
~~~
_____________________________________________________ 
# 7) Path exporting for code tests (shouldn't be needed)
-----------------------------------------------------

To test the code locally, the environment variables needs to see local directory that you have the package in. If you need these then maybe you have the wrong athena release or something.


> Full Checkout - option A 
---------------------------
~~~
export PATH=/afs/cern.ch/user/d/dspiteri/Code_Development/ATHENA/athena/DataQuality/DataQualityUtils/scripts:$PATH
export PYTHONPATH=/afs/cern.ch/user/d/dspiteri/Code_Development/ATHENA/athena/DataQuality/DataQualityUtils/python:$PYTHONPATH
~~~

> Sparse checkout - option B
----------------------------
~~~
export PATH=/afs/cern.ch/user/d/dspiteri/Code_Development/ATHENA/athena/Projects/WorkDir:$PATH
export PYTHONPATH=/afs/cern.ch/user/d/dspiteri/Code_Development/ATHENA/athena/InnerDetector/InDetValidation/InDetPhysValMonitoring/python:$PYTHONPATH
~~~
__________________________________ 
# 8) Troubleshooting
-------------------------------------------

If you are working over several session/ installations of asetup, then the  environment can start to get cluttered and not be cleaned up properly. You might want to make another fresh sparse checkout of the packages to check  that this is the case or just simply log in and out again or delete and  rebuild the 'build' directory. Also look in Debugging[PROGRAM].md files for  specific errors.

___________________________________________________ 
# 9) Making a local Commit and Pushing Changes to Fork + Stashing
----------------------------------------------------
Go to the root of the athena you want to stage the commit from
~~~
cd ~/Code_Development/ATHENA/athena
setupATLAS
lsetup git
git status (-s   To see what state the project is in shortened version)
git diff (To see what was changed - press 'q' to exit)
~~~
Add all the files that were in the last commit and have been changes
~~~
git commit -a -m "<insert message here>"
~~~
Add all the files that are in the AthenaWorkDirectory
Use fewer than 72 characters per line. 

Alternatively you can do
~~~
git add [filenames]
git commit
> Opens up an editor (nano) to write a commit message
> ['Ctrl X' to exit the 'y' to save, then press 'enter']
~~~

BEFORE you push if you want to edit your commit message. Developers may reject merges that contain bad commit messages
~~~
git commit --amend
git push --set-upstream origin <branchname>
~~~
The --set-upstream (or just -u) associates your topic branch with the copy on your fork. So subsequent pushes can be a plain git push. Otherwise you can do
~~~
git push origin <local branch name>:<remote branch name>
~~~
which will push the changes to a new branch, but your local branch name will remain unchanged

In git you need to store changes before you can move to another branch. Often, when you’ve been working on part of your project, things are in a messy intermediate state and you want to switch branches to work on something else. The problem is, you don’t want to do a commit of half-done work just so you can get back to this point later, especially if the branch you are on is not yours. The answer to this issue is the git stash command.
~~~
git status 
git stash
~~~
This now leaves you free to change your branch to something else. If you want to return to the branch and make more changes, you can reapply the stashed changes.
~~~
git stash apply
~~~
If there are multiple stashed changes, you can see which changes you have stashed. 
~~~
git stash list 
~~~
which comes up in the form
>       stash@{0}: WIP on master: vw29g0n add 250ptv bin to SR
>       stash@{1}: WIP on master: 0v189af removes 'grl' flag
>       stash@{2}: WIP on master: pt20jf5 change plotting parameters

"git stash apply" will then add the last stash back into development unless specified by

~~~
git stash apply stash@{1}
~~~
___________________________________________________ 
# 10) Making a Merge Request
-----------------------------------------------------------------
[ONLY DO THIS IF YOU HAVE RESOLVED ALL ISSUES THAT AROSE DURING TESTING]

Before making any merge requst, please make sure you have run the appropriate tests for the release (git branch) which you are targetting. For instance, for the tier-0 release you need to respect the frozen tier 0 policy and run the RunTier0Tests.py script. For all releases you should explain what tests you have run when making the merge request.
 
Secondly log into your GitLab account for ATLAS and locate the name of the  branch with contains the changes that you have created. Click the "Make Merge Request" button that lies next to this branch or type in the command line
~~~
git merge --ff-only upstream/(master) # <- target branch head
~~~
If you find yourself with the a merge conflict, and the interactive mode of fixing the problem doesn't come up then you will have to go on to the branch manually and fix any conflicts there

NOTE - If you are working on an offshoot particular branch i.e. master. You can't merge your changes to a different branch. You need to go to point 11. 

___________________________________________________ 
# 11) Merging to a different branch
---------------------------------------------------
If the branch you are wanting to submit your changes to is NOT from the base of the branch you are working, you will need to create a new branch to store your changes. Here I am on the master branchline, but my changes are wanted in the 21.0 branch. (Repeat up to step 4).
~~~
git fetch upstream #Ensures local repository is up to date with the main one
git checkout -b 21.0_Loose-Selection upstream/21.0 --no-track
~~~

Now we need to cherry-pick the commits we want to keep. You can do this by Searching git lab for the branches you worked on and looking at the changes to see if you want to include them. Then use the commit hash next to the commit in the lines below. Make sure the order is right. 

~~~
git cherry-pick e3da711e
git status
~~~
After each cherry-pick go into the files marked by "both modified" You work out which lines of code are the ones that you want and keep those. Then add that file into the staging area with "git add ..."

~~~
vim [PATH TO FILE]/InDetPhysValMonitoringTool.py
git add [PATH TO FILE]/InDetPhysValMonitoringTool.py
~~~
Once all conflicts have been fixed.
~~~
git cherry-pick --continue
~~~
Then repeat for all relevant commits you want to harvest.
~~~
git cherry-pick 54f7f944
git cherry-pick 807078a0
~~~
Don't forget to test this new branch. (Continue from Step 6)
__________________________________________________________
# 12) Merging to a modified branch.
--------------------------------------------------------------------------
When you make a branch in git, you are creating a clone of the trunk of the repository and usually the trunk code remains the same if you are editing things on a short timescale. However while you are working on your developement branch sometimes changes occur in the master branch (other people have had braches merged for example) so that you dont know that when you merge your changes from the branch back to the trunk that your code will work.

In most of these cases you will want to import these changes into your local branch. Say for example, our branch is a branch off of the master branch.
~~~
git checkout master
git pull
git checkout [localbranch]
git merge master
~~~

A similar thing is done if the inverse is true. As in you have a new version of the master and want to merge changes you made upstream to this version of the master.

~~~
git checkout master
git pull origin master
git branch -m <nameoflocalbranch> || or git checkout -b <nameoflocalbranch>
git merge origin/<nameofupstreambranch>
~~~
__________________________________________________________ 
# 13) Generating SSH Keys and adding it to the ssh-agent.
----------------------------------------------------------
Sometimes to authenticate certain git repositories, you need to generate an SSH key. 

Before you generate an SSH key, you can check to see if you have any existing SSH keys.
~~~
ls -al ~/.ssh
~~~
By default, the filenames of public keys follow the form id_XXXX.pub.
If you don't have an existing public and private key pair, or don't wish to use any that are available to connect to GitHub, then generate a new SSH key.

If you receive an error that ~/.ssh doesn't exist, fear not. It'll be created it when a new SSH key is generated. 

~~~
ssh-keygen -t rsa -b 4096 -C "dwayne.patrick.spiteri@cern.ch"
~~~
Press enter when prompted for a file to save the key to save the key file in the default locatation. Next a passphrase will be needed. 
~~~
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
> Enter the passphrase you just made (if you made one)
~~~
__________________________________________________________ 
# 14) Submodules.
----------------------------------------------------------
## based off of https://git-scm.com/book/en/v2/Git-Tools-Submodules

Sometimes when working on a project, you will have to use another project from within it. i.e you find some code on the internet you want to use. The two projects are seperate but you will want to still use one within another.

To create a submodule:
~~~
git submodule add https://github.com/dattanchu/bprinter
~~~
New .gitmodules file appears and this is a configuration file that stores the mapping between the project’s URL and the local subdirectory you’ve pulled it into.

Now when you commit and push as normal using
~~~
git commit -am "added bprinter submodule"
git push -u origin <branch_name>
~~~

[master fb9093c] added bprinter module
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 bprinter


Notice the 160000 mode for the bprinter entry. This is a special mode for git showing you that you are recording a commit as a directory rather than a subdirectory or a file. This push will not store any changes to the work in the submodule. 
 
To edit the code in the submodule, first you want to check for new work in a submodule.
~~~
cd bprinter
git fetch
git merge origin/master
    or
git submodule update --remote bprinter # to update the local code.
~~~
To check out the changes in the directory one can run
~~~
git diff --submodule (in the main directory)
git diff (in the submodule)
~~~

To work on the submodule directory. Three things need to be done. 1) Checkout a branch in the submodule; 2) Commit your changes to this branch 3) Tell Git what to do if updating the submodule pulls in new work from the upstream and you have made changes. 
~~~
cd bprinter
git checkout -b master_bprinter-edit --no-track
git add [files, directories]
git commit -am "Adding bprinter library for main project CMakeLists to find"
git submodule update --remote --merge (or --rebase)
cd ..
~~~

From the main directory now you can push the subdirectory and the main one to upstream at the same time.  
~~~
git push -u origin master_d0z0Plots --recurse-submodules=on-demand
~~~
Using the above command, Git will go into the every submodule and push it before pushing the main project. If a submodule push fails for some reason, the main project push will also fail.

Failing this a patch will always work so make it just in case.
~~~
git diff > ../bprinter.patch (for unstaged changes)
git diff --cached > ../bprinter.patch (for staged changes)
git log
git diff 9addd30ead6b88078968d0f8feabd15ca9023186 ec8ab3931f6335f78319b5de6ac8f9feb7817397 > ../bprinter.patch
~~~

Now when you want to clone a directory with a submodule in it then you have to run to clone the direcory, initialise the config file and fetch the data about ut any commits relating to the submodule. 
 ~~~
git clone --recurse-submodules https://gitlab.cern.ch/dspiteri/atlas-trackingCP-fakestudy/
~~~
If this doesn't work try
~~~
git clone https://gitlab.cern.ch/dspiteri/atlas-trackingCP-fakestudy/
git submodule init
git submodule update
git apply bprinter.patch
~~~
Since each submodule is a repository in it's own right, the only thing that your main repository tracks from it is the commit it should work on. Now if you have a particularly complicated directory structure in terms of submodules you will want to perform simple commands in your main repository and your submodules at the same time. For example "git status" only works for the region you are currently in.
~~~
git submodule foreach [git command]
~~~
This performs the command in [] in every submodule you have and the main git module. For example you could run git submodule foreach [git checkout master] to get the master version of every submodule at the same time.

WARNING! Usually you will have to actually give permissions to external groups/persons that want to access submodules you control within your repository
________________________________
# 15) .gitignore
-----------------------------------------
The .gitignore is a file you can create in the base of your repository (like the .gitconfig) that is basically a list of the relative path and filenames of files that when generated by the user, are not tracked by git. Simply just create a file called .gitignore in the correct place, put the relative path (and file name) from the .gitignore directory to the one the nuisance junk file(s) are in.

Now these files should not appear (when generated) upon a git status
_____________________________________________________________
# 16) Useful Advanced Commands
------------------------------------------------------------------------------
~~~
rm (-rf) [object]
git rm (-r) [object]
~~~
git rm  adds the fact that you are removing a file to the stage. In this case it acts in a similar way to git add. Hence it is a good way of cleaning up your directory as it stops git caring about the deleted files

~~~
git mv [fileinplace1] [fileinplace2]
~~~
git mv moves all the files/directories to where you specify and also stages this move for you at the same time.

~~~
git commit --amend "blah"
~~~
git commit --amend "blah" command lets a user replace the comment text of the last commit with "blah".

