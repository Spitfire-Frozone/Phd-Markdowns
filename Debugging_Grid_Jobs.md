# This document walks the user through some ideas of steps to be taken if the grid submissions are not working.# 

## "Debugging Grid Jobs" ##

Last Edited: 31-01-2017
-------------------------------------------------------------------------------
# Continues on from "ESD-NTUP_Physval Object Comparison" which itself # 
# Continues on from "Running_ESD's_on_the_grid_on_lxplus.md" # 

On the eventual inevitability that something goes wrong with your grid jobs you may need to do the one thing that no one in their right of mind would want to do willingly. Scroll through the log files fo the grid jobs and see what is the matter.
It is usually very useful for you to have the pandaweb page of your run to hand http://bigpanda.cern.ch/task/10729125/

Find the Output Container section and below it is shows the types of dataset. The 4th hyperlink along should be 'log(number)'. Select it to open it and highlight the name (user.dspiteri.160217.ref.f644_m1453_r9067_tid10586619_00.log.122297711)

Open a new terminal in lxplus and call the same directory you downloaded the files from the grid into, not forgetting to set up the standard environment. 

--for example
~~~
ssh -X dspiteri@lxplus.cern.ch
cd Physval/ESDforGrid/validation
setupATLAS
lsetup rucio
asetup 21.0.15,here
cd 160217ref/ESD
~~~
download the file you highlighted, making sure to use the user.[username]: at the start to indicate the container and rename the folder for ease.
~~~
rucio download user.dspiteri:user.dspiteri.160217.ref.f644_m1453_r9067_tid10586619_00.log.122297711
mv user.dspiteri.160217.ref.f644_m1453_r9067_tid10586619_00.log.122297711 runlogs
~~~
You should end up with a .tgz file for each output root file in the folder you are in. These need decompressing and should be done one at a time as the folders they get unpacked to are not clear as to their origin
~~~
cd runlogs
tar -xvzf user.dspiteri.160217.ref.f644_m1453_r9067_tid10586619_00.log.10729125.000001.log.tgz
mv tarball_PandaJob_3236364957_ANALY_SWT2_CPB/ 000001.log
~~~
Call one of the directory you just made and look at the file pilotlog.txt (or any of the other .txt's) in your favorite editor
~~~
cd 000001.log
vim pilotlog.txt
~~~
In vim you can type?\<error\> to highlight all instances of 'error' in the document. Trace them down and try to resolve them individually from the first instance of the error messages. 


