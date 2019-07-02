1009  cd RIVETtutorial/
 1010  setupATLAS
 1011  cp ∼/cgutscho/public/rivetATLASUK/material.tar.gz .
 1012  cp ∼cgutscho/public/rivetATLASUK/material.tar.gz .
 1013  cp ∼afs/cern.ch/user/cgutscho/public/rivetATLASUK/material.tar.gz .
 1014  cp ∼/afs/cern.ch/user/cgutscho/public/rivetATLASUK/material.tar.gz .
 1015  cp ∼/afs/cern.ch/user/c/cgutscho/public/rivetATLASUK/material.tar.gz .
 1016  cp /afs/cern.ch/user/c/cgutscho/public/rivetATLASUK/material.tar.gz .
 1017  ls
 1018  tar -xvzf material.tar.gz .
 1019  tar -xvzf material.tar.gz
 1020  ls
 1021  vim setupRivet2.5.4
 1022  source setupRivet2.5.4
 1023  rivet-mkanalysis MY_TEST_ANALYSIS
 1024  ls
 1025  rivet-buildplugin RivetMY_TEST_ANALYSIS.so MY_TEST_ANALYSIS.cc'

 1026  rivet-buildplugin RivetMY_TEST_ANALYSIS.so MY_TEST_ANALYSIS.cc
 1027  vim MY_TEST_ANALYSIS.cc
 1028  ls
 1029  cd Events/
 1030  ls
 1031  cd ..
 1032  ls
 1033  vim MY_TEST_ANALYSIS.cc
 1034  rivet-buildplugin MY_TEST_ANALYSIS.cc
 1035  vim MY_TEST_ANALYSIS.cc
 1036  rivet-buildplugin MY_TEST_ANALYSIS.cc
 1037  vim MY_TEST_ANALYSIS.cc
 1038  rivet-buildplugin MY_TEST_ANALYSIS.cc
 1039  ls
 1040  rivet -a MY_TEST_ANALYSIS
 1041  rivet -a MY_TEST_ANALYSIS --pwd Events/ZJets_mumu_13TeV_10k.hepmc
 1042  rivet-buildplugin MY_TEST_ANALYSIS.cc
 1043  vim MY_TEST_ANALYSIS.cc
 1044  ls
 1045  rm RivetAnalysis.so RivetMY_TEST_ANALYSIS.so
 1046  rivet-buildplugin MY_TEST_ANALYSIS.cc
 1047  rivet -a MY_TEST_ANALYSIS --pwd Events/ZJets_mumu_13TeV_10k.hepmc
 1048  ls
 1049  rivet-mkhtml Rivet.yoda
 1050  imgcat rivet-plots/MY_TEST_ANALYSIS/Z_mass.png
 1051  history
