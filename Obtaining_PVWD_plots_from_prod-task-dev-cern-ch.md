## Obtaining PVWD plots from tasks run on prod-task.dev.cern.ch ##
===============================================================================
Last Edited: 29-03-2017
-------------------------------------------------------------------------------

Occasionally a processed task will be sent to you not via pandapage but via a prodtask page such as the one below https://prodtask-dev.cern.ch/prodtask/inputlist_with_request/11523/ as opposed to something like http://bigpanda.cern.ch/task/11027507/. 

- Click the 'done' box under the sample's ptag you want. This is important as you click the one under the rtag you get to a different page with AOD outputs
- In 'datasets,show/hide by type' click the dropdown menu output()
- If this has NTUP_PHYSVAL in it's name check it has the same e,s,r tags as your 
dataset and if it does these are the NTUPS you are looking for.

Log into lxplus and download the files and follow the proceedure set out in 
"ESD-NTUP_PhysVal comparison"
 
