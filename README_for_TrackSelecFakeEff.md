## Documentation to accompany my code hosted on https://gitlab.cern.ch/dspiteri/atlas-trackingCP-fakestudy/ To be used by people trying to reproduce recommendations for the estimatation of the uncertainty due to fakes on inclusive tracks under the Loose and TightPrimary working points. ## 

# Reproducing Recommendations #

## Last Edited: 09-02-2018

## Everytime you see "/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/" Replace it with the absolute path of the directory that holds the atlas-trackingCP-fakestudy directory

# Setup the environment
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks
setupATLAS && lsetup root
rcSetup Base,2.5.0  
rc find_packages && rc compile 
~~~

# Produce "RAW" recommendations from data
Read Fake_Rate_Study_2.md for details on how to generate the samples you run over in the below command.
~~~
root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/I-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/_G-I_","2017Data2vMC16c",0)'  

vim Plots/_G-I_/Recommendations_RAW.txt
rm Plots/_G-I_/Recommendations_RAW.txt
~~~

# Generate systematic variations
~~~
source runRecommendationVariations.sh

vim Plots/_G-I_/Recommendations_RAW.txt
vim Plots/_G-I_/Recommendations_Inner_d0_RAW.txt

lsetup "ROOT 6.10.04"
cd build
make
cd ..
./build/Recom_beatif Plots/_G-I_/Recommendations_RAW.txt > Plots/_G-I_/Recommendations.txt
./build/Recom_beatif Plots/_G-I_/Recommendations_Inner_d0_RAW.txt > Plots/_G-I_/Recommendations_Inner_d0.txt

vim Plots/_G-I_/Recommendations.txt
vim Plots/_G-I_/Recommendations_Inner_d0.txt
~~~
