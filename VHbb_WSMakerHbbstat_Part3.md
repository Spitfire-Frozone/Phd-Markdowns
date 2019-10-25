# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about editing the framework that produces the fits to VHbb CxAOD's inputs and varying it. # 

## VHbb WSMaker and Hbb Stat ##

## Last Edited: 24-10-2019
-------------------------------------------------------------------------------
# Investingating B-tagging
Last time we played with the fit, some of the b-tagging systematics were being pulled in wierd ways. We will now perform many variations of the 0L fit in an attempt to better understand it, and to see what the issue is with the b-tagging parts of the fit. 
To do this we will perform decorrelations in the b-tagging systematic variables in the number of jets, in ptV and in both at the same time. 


In addition to this we will also do some general fits
- CROnly Fit
- 1-bin-in-all-SR Fit 
- Only 2jet catagroy Fit 
- Merged ptV Fit
- ad vs e

First we will want to get a fresh copy of the fit.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git && lsetup "root 6.14.04-x86_64-slc6-gcc62-opt"
~~~
